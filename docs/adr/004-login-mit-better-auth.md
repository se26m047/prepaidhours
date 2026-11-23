# ADR-004: Login mit Better Auth statt eigener Sessions

- **Status:** Angenommen (ersetzt den ersten Entwurf „eigene Sessions" in `architecture.md`)
- **Datum:** 2026-11-23

## Kontext

Im ersten Architekturentwurf wollten wir den Login selbst bauen: Passwort-Hash mit Argon2id, eine Session-Tabelle und ein Cookie. Das Review der Architektur (siehe `docs/ai-usage.md`, Phase 2) hat danach gezeigt, was dafür alles fehlt:

- Rate-Limiting gegen das Erraten von Passwörtern
- CSRF-Schutz über `SameSite` hinaus
- neue Sessions beim Login und Löschen aller Sessions nach einem Passwortwechsel
- ein sicherer Ablauf für „Passwort vergessen" (US-12, inzwischen Must) und für Einladungen (US-05)

Das alles selbst zu schreiben und zu testen, ist sicherheitskritischer Code. Dabei passieren leicht Fehler, und er hat nichts mit dem eigentlichen Thema des Projekts zu tun (Stundenpakete und Überbuchungsschutz).

Verglichen haben wir:

1. **Eigene Sessions** (erster Entwurf)
2. **Auth.js:** stark auf OAuth ausgerichtet. Login mit E-Mail und Passwort ist dort nur eingeschränkt unterstützt, und Datenbank-Sessions gehen damit nicht ohne Weiteres.
3. **Better Auth:** TypeScript-Bibliothek mit E-Mail + Passwort, Datenbank-Sessions, Passwort-Reset, Rate-Limiting und `Origin`-Prüfung. Speichert alles in unserer PostgreSQL über einen Drizzle-Adapter.

## Entscheidung

Wir verwenden **Better Auth** mit Login über E-Mail und Passwort und speichern alles in unserer PostgreSQL:

- **Registrierung:** abgeschaltet (`disableSignUp`), Kunden legt nur der Admin an.
- **Hashing:** Argon2id über `@node-rs/argon2`. Der Standard von Better Auth ist scrypt, das NFA-02 nicht erfüllt.
- **Benutzerfelder:** `user` wird um `role` und `company_id` erweitert.
- **Einladung (US-05):** ist ein Reset-Link mit 48 h Gültigkeit. „Passwort vergessen" (US-12) ist derselbe Mechanismus mit 1 h.
- **Rate-Limit:** in der Datenbank gespeichert.

Die Berechtigung (welche Rolle, welche Firma) bleibt in unseren Services (`requireAdmin`, `requireCustomer`). Better Auth sagt nur, **wer** eingeloggt ist.

## Begründung

- Die Sicherheitsfunktionen, die das Review vermisst hat, sind eingebaut und von vielen Projekten getestet.
- Einladung und Passwort-Reset brauchen nur einen Mechanismus statt zwei.
- Better Auth ist in TypeScript geschrieben und hat einen Drizzle-Adapter. Es passt also direkt zu unserem Stack (ADR-001), ohne zusätzliche Dienste.
- Die Daten bleiben in unserer Datenbank. Es gibt keinen externen Login-Dienst und keine weitere Firma, die Kundendaten verarbeitet.

## Konsequenzen

- **Positiv:** Deutlich weniger eigener Sicherheitscode. US-12 ist mit wenig Aufwand machbar.
- **Negativ:** Eine weitere Abhängigkeit, deren Updates wir verfolgen müssen. Deshalb pinnen wir die Version fest.
- **Negativ:** Better Auth legt eigene Tabellen an (`user`, `session`, `account`, `verification`), deren Aufbau wir nicht frei bestimmen. Das Datenmodell in `architecture.md` passt sich daran an.
- **Zu testen:** Dass wirklich Argon2id verwendet wird (Hash beginnt mit `$argon2id$`), dass nach einem Passwortwechsel alle Sessions ungültig sind und dass das Rate-Limit greift.
