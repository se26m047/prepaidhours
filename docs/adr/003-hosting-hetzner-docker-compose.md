# ADR-003: Hosting auf einem Hetzner Cloud Server mit Docker Compose

- **Status:** Angenommen
- **Datum:** 2026-11-23

## Kontext

PrepaidHours braucht Hosting für eine Next.js-App und eine PostgreSQL-Datenbank in der EU, mit HTTPS und Backups, zu geringen Kosten. Der Überbuchungsschutz (ADR-002) braucht Transaktionen mit Zeilensperren über eine normale Datenbankverbindung. Verglichen haben wir (Details und Preise in `docs/hosting.md`):

1. Hetzner Cloud Server mit Docker Compose (App, PostgreSQL, Caddy)
2. Render, Region Frankfurt (PaaS + verwaltetes PostgreSQL)
3. Vercel + Neon (serverless)

## Entscheidung

Wir hosten auf **einem Hetzner Cloud Server (CX23) in Deutschland**. Mit Docker Compose laufen dort die App, PostgreSQL und Caddy (HTTPS über Let's Encrypt). Jede Nacht kommt ein `pg_dump` auf eine Hetzner Storage Box. E-Mails verschicken wir über **Brevo**.

## Begründung

- **Kosten:** ca. 7 bis 10 € im Monat für alles, ohne „nicht kommerziell" (Vercel Hobby) und ohne Einschlafen der App (Render Free).
- **Datenschutz:** Hetzner und Brevo sind EU-Firmen mit Rechenzentren in der EU.
- **Passt zu ADR-002:** Ein dauerhaft laufender Node-Prozess mit direkter Datenbankverbindung. Serverless-Funktionen mit Connection-Pooler hätten Sperren und lange Transaktionen schwieriger gemacht.
- **Gleiche Umgebung lokal und live:** Dieselbe `docker-compose.yml` läuft auf dem Laptop und auf dem Server.

## Konsequenzen

- **Negativ:** Wir betreiben den Server selbst: Sicherheitsupdates (`unattended-upgrades`), Firewall, SSH-Schlüssel und Backups. Das beschreiben wir einmal in einer Betriebsanleitung, damit es jeder im Team kann.
- **Negativ:** Ein einziger Server ist ein Single Point of Failure. Für unser Projekt ist das in Ordnung. Die Backups liegen auf einem anderen System, und ein Wiederherstellungstest ist eingeplant.
- **Positiv:** Wenn ein echter Dienstleister das Portal später nutzt, kann es so bleiben. Bei Bedarf wechselt man auf einen größeren Servertyp oder auf eine verwaltete Datenbank.
- **Offen:** Die genauen Preise ändern sich (Hetzner hat sie im Juni 2026 erhöht). Vor dem ersten Bestellen prüfen wir sie nochmal.
