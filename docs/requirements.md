# Anforderungen

Wir priorisieren mit Must, Should und Could. Must heißt: ohne das funktioniert der Grundablauf nicht.

## User Stories

### US-01 Login (Must)
Als Kunde möchte ich mich mit E-Mail und Passwort einloggen, damit nur ich meine Daten sehe.

Akzeptanzkriterien:
- Mit richtigen Daten komme ich auf mein Dashboard.
- Mit falschen Daten kommt eine Fehlermeldung. Die Meldung sagt nicht, ob es die E-Mail gibt oder nicht.

### US-02 Guthaben sehen (Must)
Als Kunde möchte ich sehen, wie viele Stunden ich noch habe, damit ich weiß, wann ich nachbestellen muss.

Akzeptanzkriterien:
- Ich habe zwei bezahlte Pakete mit 10 h und 5 h, und es wurden 12 h gebucht. Dann steht auf dem Dashboard 3:00 h.
- Ein Paket, das noch nicht bezahlt ist, zählt nicht zum Guthaben. Es wird aber als „wartet auf Zahlung" angezeigt.

### US-03 Verlauf sehen (Must)
Als Kunde möchte ich alle gebuchten Arbeiten sehen, damit ich weiß, wofür meine Stunden draufgegangen sind.

Akzeptanzkriterien:
- Jeder Eintrag zeigt Datum, Beschreibung und Dauer.
- Die neuesten Einträge stehen oben.
- Ich sehe nur die Einträge von meiner eigenen Firma.

### US-04 Paket anfragen (Must)
Als Kunde möchte ich direkt im Portal ein neues Paket anfragen, damit ich nicht extra eine E-Mail schreiben muss.

Akzeptanzkriterien:
- Nach dem Absenden gibt es ein neues Paket mit Status „angefragt".
- Der Dienstleister bekommt eine E-Mail, damit er die Rechnung schicken kann.
- Das Guthaben ändert sich dadurch noch nicht.
- Der Admin kann eine Anfrage mit Grund ablehnen. Sie bekommt dann den Status „abgelehnt" und wird dem Kunden nicht mehr als „wartet auf Zahlung" angezeigt.

### US-05 Kunden anlegen (Must)
Als Admin möchte ich einen neuen Kunden anlegen, damit er das Portal nutzen kann.

Akzeptanzkriterien:
- Ich muss Firmenname, Name und E-Mail angeben.
- Der Kunde bekommt eine E-Mail mit einem Link, über den er sein Passwort setzt. Der Link gilt 48 Stunden.
- Eine E-Mail-Adresse kann nicht zweimal verwendet werden.
- Ist der Link abgelaufen oder die E-Mail nicht angekommen, kann der Admin einen neuen Link schicken. Der alte Link gilt dann nicht mehr.

### US-06 Paket anlegen und als bezahlt markieren (Must)
Als Admin möchte ich ein Paket mit Stunden und Stundensatz anlegen und es als bezahlt markieren, wenn das Geld da ist.

Akzeptanzkriterien:
- Stunden und Stundensatz müssen größer als 0 sein.
- Den Stundensatz kann man danach nicht mehr ändern, sonst passt er nicht mehr zur Rechnung.
- Sobald ich auf „bezahlt" klicke, steigt das Guthaben des Kunden um die Stunden des Pakets.

### US-07 Arbeit buchen (Must)
Als Admin möchte ich geleistete Arbeit eintragen, damit sie vom Guthaben abgezogen wird.

Akzeptanzkriterien:
- Ich gebe Datum, Beschreibung und Dauer ein. Die Dauer geht in 15-Minuten-Schritten.
- Es wird immer zuerst vom ältesten bezahlten Paket abgezogen.
- Wenn das älteste Paket nur noch 0:30 h hat und ich 1:00 h buche, werden 0:30 h davon und 0:30 h vom nächsten Paket abgezogen.

### US-08 Nicht überbuchen (Must)
Als Dienstleister möchte ich, dass man nicht mehr buchen kann als bezahlt ist, damit wir nie umsonst arbeiten.

Akzeptanzkriterien:
- Bei 1:00 h Guthaben wird eine Buchung über 1:15 h komplett abgelehnt, mit einer Meldung, wie viel noch übrig ist.
- Das muss auch funktionieren, wenn zwei Admins genau gleichzeitig buchen.

### US-12 Passwort vergessen (Must)
Als Kunde möchte ich mein Passwort selbst zurücksetzen können, damit ich nicht den Dienstleister anrufen muss, wenn ich es vergessen habe.

Akzeptanzkriterien:
- Auf der Login-Seite gebe ich meine E-Mail ein und bekomme einen Link zum Zurücksetzen. Der Link gilt 1 Stunde.
- Die Meldung nach dem Absenden ist immer gleich, auch wenn es die E-Mail nicht gibt.
- Nach dem Zurücksetzen werde ich auf allen Geräten abgemeldet.

### US-09 Warnung bei wenig Guthaben (Should)
Als Kunde möchte ich eine E-Mail bekommen, wenn ich unter 2 Stunden falle, damit ich rechtzeitig nachbestelle. Die E-Mail kommt nur einmal und nicht bei jeder weiteren Buchung.

### US-10 Buchung stornieren (Should)
Als Admin möchte ich eine falsche Buchung stornieren können, damit der Kunde die Stunden zurückbekommt. Die Buchung wird dabei nicht gelöscht, sondern als storniert markiert, mit Grund.

### US-11 Export (Could)
Als Kunde möchte ich meinen Verlauf als CSV herunterladen, damit ich ihn intern weitergeben kann.

## Funktionale Anforderungen

| ID | Anforderung | Prio |
|---|---|---|
| FA-01 | Kunden loggen sich mit E-Mail und Passwort ein. | Must |
| FA-02 | Ein Kunde sieht nur die Daten seiner eigenen Firma. | Must |
| FA-03 | Guthaben = bezahlte Pakete minus gebuchte Stunden. Angefragte oder offene Pakete zählen nicht. | Must |
| FA-04 | Ein Paket ist entweder angefragt, offen, bezahlt oder abgelehnt. Abgelehnt werden kann nur ein angefragtes Paket. | Must |
| FA-05 | Bei einer Paketanfrage geht eine E-Mail an den Dienstleister. | Must |
| FA-06 | Arbeitszeit wird in 15-Minuten-Schritten gebucht. | Must |
| FA-07 | Gebucht wird immer zuerst vom ältesten bezahlten Paket. Eine Buchung kann auf zwei Pakete aufgeteilt werden. | Must |
| FA-08 | Reicht das Guthaben nicht, wird die ganze Buchung abgelehnt. | Must |
| FA-09 | Unter 2 h Guthaben bekommt der Kunde einmal eine Warnung. | Should |
| FA-10 | Stornierte Buchungen bleiben gespeichert. | Should |
| FA-11 | Kunden können ihr Passwort selbst zurücksetzen, Admins können einen neuen Einladungslink schicken. | Must |

## Nichtfunktionale Anforderungen

| ID | Anforderung | Wie wir es prüfen |
|---|---|---|
| NFA-01 | Das Guthaben darf nie negativ werden, auch nicht bei gleichzeitigen Buchungen. Das wird in der Datenbank abgesichert und nicht nur im Frontend. | Test: 10 Buchungen zu je 1 h gleichzeitig auf 5 h Guthaben. Genau 5 dürfen durchgehen. |
| NFA-02 | Passwörter werden gehasht gespeichert (bcrypt oder Argon2). Ein Kunde kommt auch über eine geänderte URL nicht an fremde Daten. | Test: Kunde A ruft eine Seite von Kunde B auf und bekommt einen Fehler. |
| NFA-03 | Das Dashboard lädt in unter 2 Sekunden, auch wenn ein Kunde 1000 Buchungen hat. | Mit Testdaten messen. |
| NFA-04 | Die Seiten funktionieren auf dem Handy (ab 360 px Breite). | Am eigenen Handy testen. |

## Was wir geklärt haben

Beim Durchgehen der Anforderungen sind ein paar Fragen aufgekommen, die wir so entschieden haben:

- **Buchung größer als der Rest im Paket?** Wird auf zwei Pakete aufgeteilt.
- **Teilbuchung, wenn das Guthaben nicht ganz reicht?** Nein. Entweder ganz oder gar nicht, sonst arbeiten wir wieder auf Rechnung.
- **Zählen angefragte Pakete schon zum Guthaben?** Nein, erst wenn sie bezahlt sind.
- **Stundensatz nachträglich ändern?** Nein, weil die Rechnung schon draußen ist.

Noch offen ist, ob Pakete irgendwann verfallen sollen. Das müssen wir noch klären.

## Änderungen nach Phase 2

Beim Review der Architektur (siehe `docs/ai-usage.md`, Phase 2) sind zwei Lücken in den Anforderungen aufgefallen:

- **Fehlerhafte Paketanfragen** konnten nie entfernt werden. Neu: Status „abgelehnt" (US-04, FA-04).
- **Ein Kunde konnte sich aussperren**, wenn die Einladung nicht ankommt oder er das Passwort vergisst. Neu: Einladung erneut senden (US-05) und Passwort vergessen (US-12, FA-11). US-12 ist Must, weil sonst der Grundablauf an einer verlorenen E-Mail scheitern kann.
