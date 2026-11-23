# AI-Einsatz

Wir haben Claude (Anthropic) verwendet, und zwar für zwei Sachen.

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

## Fazit

Die AI war gut darin, schnell einen Entwurf zu machen und Lücken zu finden. Aber was das Produkt genau können soll, mussten wir selbst entscheiden. Zwei Vorschläge hätten unser eigentliches Ziel kaputtgemacht, wenn wir sie einfach übernommen hätten.
