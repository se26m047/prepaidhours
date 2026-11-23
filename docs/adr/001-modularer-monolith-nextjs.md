# ADR-001: Modularer Monolith mit Next.js und PostgreSQL

- **Status:** Angenommen
- **Datum:** 2026-11-23

## Kontext

PrepaidHours hat zwei Rollen (Kunde, Admin), ungefähr zehn Seiten und eine zentrale Regel: Es darf nie mehr gebucht werden, als bezahlt ist (US-08, NFA-01). Das Projekt muss innerhalb eines Semesters von einem kleinen Team gebaut, getestet und betrieben werden. Eine eigene App ist laut `mvp.md` bewusst ausgeschlossen.

Wir haben zwei Varianten verglichen (Details in `docs/architecture.md`):

- **A:** Ein modularer Monolith mit Next.js. Eine Anwendung rendert die Seiten und stellt die API bereit, innen ist sie in Module aufgeteilt.
- **B:** Eine React-Single-Page-App und ein getrenntes REST-Backend, die getrennt deployt werden.

## Entscheidung

Wir bauen **Variante A**: eine Next.js-Anwendung (TypeScript, App Router) mit den Modulen `auth`, `firmen`, `pakete`, `buchungen` und `benachrichtigungen` sowie einer PostgreSQL-Datenbank. Nur die Service-Schicht der Module greift auf die Datenbank zu.

## Begründung

- Variante B erfüllt keine unserer Anforderungen besser. Ihr Vorteil (mehrere Clients nutzen dieselbe API) betrifft nur die eigene App, und die haben wir ausgeschlossen.
- Mit einem Projekt, einer Sprache und einem Deployment haben wir weniger Arbeit. Es gibt kein CORS und keine Token zwischen Frontend und Backend.
- Seiten werden auf dem Server mit der Session geprüft und gerendert. Daten anderer Firmen kommen gar nicht erst in den Browser (NFA-02).
- Server-Rendering schickt wenig JavaScript ans Handy (NFA-03, NFA-04).
- Microservices haben wir ausgeschlossen. Die Buchung braucht Pakete und Buchungen in einer Transaktion (ADR-002).

## Konsequenzen

- **Positiv:** Schneller Start und einfacher Betrieb. Die Geschäftslogik liegt an genau einer Stelle (Services), egal ob ein Formular oder die API sie aufruft.
- **Negativ:** Wir sind an Next.js gebunden. Größere Änderungen im Framework (z. B. beim Caching) können uns treffen.
- **Negativ:** Ohne Disziplin rutscht Logik in Seiten und Routen. Die Regel „nur Services sprechen mit der Datenbank" prüfen wir im Code-Review.
- **Festgelegt nach dem Review:** Seiten lesen, Änderungen laufen nur über die API-Routen. Die Berechtigung prüft jede Service-Funktion selbst (`requireAdmin`, `requireCustomer`). Die Next.js-Middleware leitet nur auf den Login um und ist keine Sicherheitsgrenze. Damit gibt es für jede Änderung genau eine Tür, und kein Einstiegspunkt kann die Prüfung umgehen.
- **Neutral:** Braucht man später doch eine eigene App, gibt es die REST-API unter `/api` schon.
