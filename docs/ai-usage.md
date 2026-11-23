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

In Phase 2 haben wir Claude Code (Claude Opus 5.5) direkt im Repository verwendet. Es hatte also `requirements.md` und `mvp.md` als Grundlage.

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

## Fazit Phase 2

Claude war gut darin, die Anforderungen in technische Entscheidungen zu übersetzen und Alternativen sauber gegenüberzustellen. Die Wahl der Technologie haben wir selbst getroffen, und Fakten wie Preise mussten wir selbst nachprüfen, weil die Quellen sich widersprochen haben.
