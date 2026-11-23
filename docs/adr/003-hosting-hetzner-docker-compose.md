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

- **Später echt nutzbar:** Ein Dienstleister kann das Portal ohne Umbau weiterbetreiben. Vercel Hobby ist nur für nicht-kommerzielle Nutzung erlaubt, Render Free schläft ein.
- **Kosten:** ca. 7 bis 10 € im Monat für alles.
- **Datenschutz:** Hetzner und Brevo sind EU-Firmen mit Rechenzentren in der EU.
- **Keine kalten Starts:** Ein dauerhaft laufender Node-Prozess mit direkter Datenbankverbindung. Mit Vercel + Neon ginge `FOR UPDATE` auch, bräuchte aber eine direkte Verbindung neben dem Pooler und mehr Konfiguration. Das ist kein Ausschlussgrund, aber ein Punkt mehr, der schiefgehen kann.
- **Gleiche Umgebung lokal und live:** Dieselbe `docker-compose.yml` läuft auf dem Laptop und auf dem Server.

## Konsequenzen

- **Negativ:** Wir betreiben den Server selbst: Sicherheitsupdates (`unattended-upgrades`), Firewall, SSH-Schlüssel, Backups und Überwachung. Das ist in `hosting.md` unter „Betrieb" beschrieben, damit es jeder im Team kann. Migrationen laufen als eigener Schritt vor dem Start, damit eine fehlerhafte Migration nicht die laufende Version kaputt macht.
- **Negativ:** Ein einziger Server ist ein Single Point of Failure. Für unser Projekt ist das in Ordnung. Die Backups liegen auf einem anderen System, und ein Wiederherstellungstest ist eingeplant.
- **Positiv:** Wenn ein echter Dienstleister das Portal später nutzt, kann es so bleiben. Bei Bedarf wechselt man auf einen größeren Servertyp oder auf eine verwaltete Datenbank.
- **Offen:** Die genauen Preise ändern sich (Hetzner hat sie im Juni 2026 erhöht). Vor dem ersten Bestellen prüfen wir sie nochmal.
