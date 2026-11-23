# AI-Einsatz

Wir haben Claude (Anthropic) verwendet. In Phase 1 für die Anforderungen, in Phase 2 für die Architektur.

# Phase 1: Anforderungen

## 1. User Stories erstellen

Wir haben Claude die Aufgabenstellung und unsere Idee gegeben und gefragt, ob es daraus User Stories und Anforderungen machen kann.

Prompt:
> Hier ist die Aufgabe für Phase 1. Unsere Idee: Kunden kaufen Stundenpakete im Voraus, wir buchen die Arbeit davon ab und die Kunden sehen ihr Restguthaben. Erstelle User Stories mit Akzeptanzkriterien, funktionale und nichtfunktionale Anforderungen und einen MVP-Vorschlag.

Was gut war:
- Wir hatten sehr schnell eine Struktur, mit der wir weiterarbeiten konnten.
- Das Format der User Stories hat gepasst.

Was wir ändern mussten:
- Einige Akzeptanzkriterien waren zu ungenau, zum Beispiel „das Guthaben wird richtig angezeigt". Das kann man nicht testen. Wir haben das durch konkrete Zahlen ersetzt.
- „Das System soll schnell sein" war als Anforderung nutzlos. Jetzt steht da unter 2 Sekunden bei 1000 Buchungen.
- Online-Zahlung war als Must drin. Das ist für ein Semester zu viel, deshalb haben wir es rausgenommen.
- Es waren mehr Stories als nötig. Wir haben auf 11 gekürzt.

## 2. Anforderungen prüfen

Danach haben wir Claude die Anforderungen nochmal gegeben und nach Fehlern suchen lassen.

Prompt:
> Prüfe diese Anforderungen auf Widersprüche, Lücken und Sachen, die man nicht testen kann. Sag bei jedem Problem, welche Anforderung es betrifft und wie man es lösen könnte.

Was gefunden wurde und was wir gemacht haben:
- Es war nicht klar, was passiert, wenn eine Buchung größer ist als der Rest im ältesten Paket. Das stimmt, wir haben festgelegt, dass auf zwei Pakete aufgeteilt wird.
- Wenn man das Überbuchen nur im Frontend verhindert, kann es bei zwei gleichzeitigen Buchungen trotzdem passieren. Das war der wichtigste Hinweis, daran hatten wir nicht gedacht.
- Es war unklar, ob angefragte Pakete schon zum Guthaben zählen. Jetzt steht drin, dass sie das nicht tun.
- Claude hat vorgeschlagen, dass man bei zu wenig Guthaben einen Teil buchen kann. Das haben wir abgelehnt, weil dann genau das passiert, was wir verhindern wollen.
- Claude hat vorgeschlagen, dass Pakete nach 12 Monaten verfallen. Das ist eher eine rechtliche Frage, das haben wir als offen markiert.

## Fazit Phase 1

Die AI war gut darin, schnell einen Entwurf zu machen und Lücken zu finden. Aber was das Produkt genau können soll, mussten wir selbst entscheiden. Zwei Vorschläge hätten unser eigentliches Ziel kaputtgemacht, wenn wir sie einfach übernommen hätten.

# Phase 2: Architektur und Design

In Phase 2 haben wir Claude Code (Claude Opus 5.5) direkt im Repository verwendet. Es hatte also `requirements.md` und `mvp.md` als Grundlage. Wir haben AI für drei Sachen eingesetzt: Technologien abfragen, Architektur entwerfen und die Architektur von einer zweiten, unabhängigen AI prüfen lassen.

## 1. Technologie und Hosting festlegen

Claude hat zuerst nicht losgeschrieben, sondern zwei Fragen gestellt: welcher Stack (Next.js, Django, Spring Boot) und welches Hosting (Hetzner, Render, FH-Infrastruktur). Begründung: Laut Kurs muss das Team die Technik verstehen und verteidigen können, also soll das Team entscheiden.

Wir haben **Next.js + PostgreSQL** und **Hetzner** gewählt.

## 2. Architektur, Datenmodell, API und ADRs entwerfen

Prompt (sinngemäß):
> Phase 2 ist jetzt dran. Erstelle docs/architecture.md (zwei Varianten vergleichen, Technologieauswahl begründen, C4-Diagramm, Datenmodell, API-Design), docs/hosting.md (mindestens zwei Hosting-Optionen mit Kosten) und mindestens zwei ADRs, passend zu unseren Anforderungen.

Was Claude gemacht hat:
- Zwei Varianten verglichen (modularer Monolith vs. SPA + eigene API) und Microservices mit Begründung ausgeschlossen.
- Den Überbuchungsschutz (NFA-01) als wichtigste Architekturentscheidung erkannt und drei Lösungen verglichen (Guthaben berechnen + Sperre, Rest pro Paket speichern + Sperre + `CHECK`, `SERIALIZABLE`), siehe ADR-002.
- Für die Hosting-Preise eine Websuche gemacht. Dabei waren die Quellen nicht einig: Manche nennen noch den alten Servertyp CX22 und Preise von vor der Hetzner-Preiserhöhung im Juni 2026, andere schon CX23 mit ca. 5,49 €. Deshalb stehen die Preise in `hosting.md` als „ca." mit dem Hinweis, sie vor der Abgabe zu prüfen.

## 3. Architektur von einer zweiten AI prüfen lassen

Den Entwurf haben wir von einer zweiten Claude-Instanz prüfen lassen. Sie hat den Gedankengang des Entwurfs nicht gekannt, sondern nur die fertigen Dokumente gelesen. So sollte sie nicht einfach bestätigen, was die erste Instanz geschrieben hat.

Prompt (gekürzt):
> Du bist ein kritischer Software-Architekt. Lies requirements.md und mvp.md (die Quelle der Wahrheit), dann architecture.md, hosting.md und die ADRs. Finde echte Schwächen, Risiken, Widersprüche zu den Anforderungen, Over-Engineering und unbegründete Entscheidungen. Prüfe besonders, ob Datenmodell und Buchungsablauf US-07, US-08 und NFA-01 wirklich erfüllen, ob die API zum Datenmodell passt und wo es Sicherheitslücken gibt. Höchstens 8 Befunde, je mit Begründung, Vorschlag und Schweregrad.

Ergebnis: 8 Befunde. Das Review fand die Grundarchitektur passend, vor allem die Rückverfolgbarkeit auf die Anforderungs-IDs und den zweifachen Überbuchungsschutz. Die Lücken lagen bei Abläufen, an die wir nicht gedacht hatten.

## 4. Unsere kritische Prüfung der Befunde

Zu jedem Befund hat die erste Claude-Instanz eine Empfehlung gemacht. Wir haben jeden Befund geprüft und entschieden. Zweimal haben wir uns bewusst gegen die Empfehlung entschieden.

| # | Befund (Schwere) | Empfehlung der AI | Unsere Entscheidung | Begründung |
|---|---|---|---|---|
| 1 | Kunde kann sich aussperren: kein „Einladung erneut senden", kein „Passwort vergessen" (hoch) | Erneut senden ins MVP, „Passwort vergessen" später als Could | **Beides ins MVP**, US-12 als Must (**gegen** die Empfehlung) | Unsere Kunden sind technisch nicht fit (siehe Zielgruppe im README). Wenn der Zugang nur über einen Anruf beim Dienstleister wiederherstellbar ist, scheitert der Grundablauf an einer verlorenen E-Mail. |
| 2 | Statuswechsel nicht atomar: zweimal „bezahlt" setzt das Restguthaben zurück, zweimal stornieren gibt doppelt zurück (hoch) | Bedingte Updates, eine Sperrreihenfolge, Tests mit Doppelklick | **Angenommen** | Das ist dieselbe Art Fehler wie das Überbuchen aus Phase 1, nur an einer anderen Stelle. |
| 3 | Zwei Wege in jede Funktion (Seiten und REST-API), Berechtigung in der Middleware (mittel bis hoch) | Nur einen Weg, Berechtigung in die Services | **Teilweise angenommen:** Berechtigung in die Services, Seiten nur lesend, Änderungen nur über die REST-API. **Abgelehnt:** der Vorschlag, auf Server Actions umzustellen | Eine ausdrückliche, testbare API ist ein Deliverable von Phase 2 und lässt sich mit Playwright direkt testen. Die Middleware bleibt nur eine Umleitung zum Login. |
| 4 | Sicherheit hört bei Hashing und Cookies auf (mittel) | Rate-Limit, `Origin`-Prüfung, Passwortregeln, Sessions nach Passwortwechsel löschen | **Angenommen** | – |
| 4a | Folge aus 1 und 4: so viel Login-Sicherheit selbst bauen? | – | **Login mit Better Auth statt eigener Sessions** (ADR-004) | Die fehlenden Funktionen sind dort eingebaut und getestet. Sicherheitskritischen Code selbst zu schreiben, ist für unser Projekt das größere Risiko. |
| 5 | Falsche Paketanfragen können nie entfernt werden, Status und Stundensatz nur im Code abgesichert (mittel) | Angefragte Pakete löschen dürfen, FA-04 unverändert lassen, mehr `CHECK`s | **Neuer Status „abgelehnt"** mit Grund, FA-04 geändert, `CHECK`s übernommen (**gegen** die Empfehlung) | Eine Ablehnung bleibt nachvollziehbar, statt spurlos zu verschwinden. Das passt dazu, dass wir auch Buchungen nicht löschen, sondern stornieren. |
| 6 | Warnung bei wenig Guthaben (US-09) ungenau, würde nur einmal im Leben kommen (mittel bis niedrig) | Regel zum Setzen und Zurücksetzen festlegen | **Angenommen** | – |
| 7 | Betrieb: Migrationen beim Start, kein Monitoring, Spam-Gefahr (mittel bis niedrig) | Eigener Migrationsschritt, Uptime-Check, SPF/DKIM, Restore-Test vor der Demo | **Angenommen** | – |
| 8 | Hosting-Vergleich leicht schief: `FOR UPDATE` ginge auch mit Vercel + Neon (niedrig) | Begründung korrigieren | **Angenommen**, `hosting.md` und ADR-003 korrigiert | Die Entscheidung bleibt gleich, aber sie soll auf den richtigen Gründen stehen. |

Was die AI übersehen hätte, wenn wir nur den ersten Entwurf genommen hätten: Befund 2. Wir hatten den Überbuchungsschutz für das Buchen sauber gelöst und dachten, damit ist NFA-01 erledigt. Ein doppelter Klick auf „bezahlt" hätte das Guthaben trotzdem falsch gemacht, und zwar so, dass kein `CHECK` anschlägt.

## Fazit Phase 2

Claude war gut darin, die Anforderungen in technische Entscheidungen zu übersetzen und Alternativen gegenüberzustellen. Den größten Wert hatte aber das zweite, unabhängige Review: Der Entwurf allein hat überzeugend gewirkt und trotzdem Lücken gehabt. Bei zwei Empfehlungen haben wir uns bewusst anders entschieden, weil sie nicht zu unserer Zielgruppe bzw. zu unserem Grundsatz „nichts löschen" gepasst haben. Die Arbeitsteilung hat sich bewährt: Die AI schlägt Optionen vor, eine unabhängige AI sucht Schwächen, und wir entscheiden. Fakten wie Preise mussten wir selbst nachprüfen, weil die Quellen sich widersprochen haben.
