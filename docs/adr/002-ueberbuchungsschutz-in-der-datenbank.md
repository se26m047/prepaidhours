# ADR-002: Überbuchungsschutz mit Zeilensperre und gespeichertem Restguthaben

- **Status:** Angenommen
- **Datum:** 2026-11-23

## Kontext

US-08 und NFA-01 verlangen, dass das Guthaben nie negativ wird, auch wenn zwei Admins genau gleichzeitig buchen. Die Absicherung muss in der Datenbank liegen, nicht nur im Frontend. Gleichzeitig wird laut US-07 und FA-07 immer zuerst vom ältesten bezahlten Paket abgezogen, und eine Buchung kann auf zwei Pakete aufgeteilt werden.

Das typische Problem: Zwei Anfragen lesen gleichzeitig „1:00 h Guthaben", beide prüfen „1:00 h reicht", beide speichern. Am Ende sind −1:00 h übrig.

Wir haben drei Möglichkeiten angeschaut:

1. **Guthaben jedes Mal ausrechnen** (Summe bezahlter Pakete minus Summe der Buchungen) und die Firma vor der Prüfung mit `SELECT … FOR UPDATE` sperren.
2. **Restguthaben pro Paket speichern** (`package.remaining_minutes`), die betroffenen Pakete mit `FOR UPDATE` sperren und zusätzlich `CHECK (remaining_minutes >= 0)` setzen.
3. **Isolationsstufe `SERIALIZABLE`** ohne explizite Sperren und bei Konflikt die Transaktion wiederholen.

## Entscheidung

Wir nehmen **Möglichkeit 2**. Eine Buchung läuft in einer Transaktion:

1. Bezahlte Pakete der Firma mit `remaining_minutes > 0` lesen, sortiert nach `paid_at, id`, mit `FOR UPDATE`. `id` entscheidet, wenn zwei Pakete gleichzeitig bezahlt wurden.
2. Ist die Summe kleiner als die Dauer, Abbruch mit `INSUFFICIENT_BALANCE` und dem Restguthaben.
3. Sonst die Buchung speichern und die Minuten vom ältesten Paket an verteilen: `remaining_minutes` verringern und je Paket eine Zeile in `booking_allocation` anlegen.
4. Commit.

Der Constraint `CHECK (remaining_minutes >= 0 AND remaining_minutes <= total_minutes)` ist die zweite Sicherung.

## Begründung

- **Zwei unabhängige Sicherungen.** Die Sperre sorgt dafür, dass gleichzeitige Buchungen nacheinander laufen. Der `CHECK` greift auch dann, wenn unser Code einen Fehler hat. Bei Möglichkeit 1 hängt alles an der Sperre, und einen `CHECK` über eine Summe aus mehreren Tabellen kann PostgreSQL nicht ausdrücken.
- **Die Aufteilung braucht den Rest pro Paket sowieso.** Um vom ältesten Paket abzuziehen, muss man wissen, wie viel in welchem Paket noch übrig ist. Wenn das gespeichert ist, muss man es nicht bei jeder Buchung aus allen alten Buchungen neu ausrechnen.
- **Schnelles Dashboard (NFA-03).** Das Guthaben ist eine Summe über ein paar Pakete statt über bis zu 1000 Buchungen.
- **Gegen `SERIALIZABLE`:** Funktioniert auch, aber dann muss jede Buchung bei Konflikten wiederholt werden. Diese Logik müssen wir schreiben und testen, und das Verhalten ist schwerer zu erklären.

## Konsequenzen

- **Positiv:** Der Test aus NFA-01 (10 gleichzeitige Buchungen zu je 1 h auf 5 h Guthaben, genau 5 erfolgreich) ist direkt umsetzbar und deterministisch.
- **Negativ:** Das Restguthaben ist eine abgeleitete Zahl und könnte von den Buchungen abweichen. Dagegen hilft, dass jede Änderung an `remaining_minutes` in derselben Transaktion wie die passende `booking_allocation` passiert. Zusätzlich prüft ein Test, dass `total_minutes − remaining_minutes` gleich der Summe der nicht stornierten Zuordnungen ist.
- **Negativ:** Gleichzeitige Buchungen für **dieselbe** Firma warten kurz aufeinander. Bei unserer Größe merkt man das nicht. Buchungen für verschiedene Firmen blockieren sich nicht.
- **Folge für Stornieren (US-10):** Eine Stornierung gibt die Minuten über `booking_allocation` an genau die Pakete zurück, aus denen sie kamen, in derselben Transaktion. Sie sperrt die Pakete in derselben Reihenfolge wie das Buchen (`paid_at, id`), damit sich Buchen und Stornieren nicht gegenseitig blockieren (Deadlock).
- **Folge für Statuswechsel (nach dem Review ergänzt):** Die Sperre schützt nur das Buchen. `mark-paid` und `cancel` könnten bei einem Doppelklick zweimal laufen. Ein zweites `mark-paid` würde das Restguthaben wieder auf die volle Stundenzahl setzen, ohne dass der `CHECK` anschlägt. Deshalb ist jeder Statuswechsel ein bedingtes Update (`… WHERE status = 'open'`, `… WHERE cancelled_at IS NULL`). Kommt keine Zeile zurück, gibt es `409 INVALID_STATUS`. Zusätzlich prüft ein `CHECK`, dass nur bezahlte Pakete Restguthaben haben. Getestet wird beides mit zwei parallelen Aufrufen.
