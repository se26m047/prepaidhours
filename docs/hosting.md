# Hosting

## Was wir brauchen

- Eine Node.js-Anwendung (Next.js) und eine PostgreSQL-Datenbank
- Hosting in der EU, weil wir Kundendaten speichern (DSGVO)
- HTTPS mit eigener Domain
- Tägliche Backups der Datenbank
- Günstig: Ein Semesterprojekt mit wenigen Benutzern. Wenn ein echter Dienstleister es später nutzt, soll es ohne Umbau weiterlaufen.
- E-Mail-Versand (Passwort-Links, Paketanfragen)

## Optionen

### Option 1: Hetzner Cloud Server mit Docker Compose

Ein kleiner virtueller Server bei Hetzner in Deutschland (Nürnberg oder Falkenstein). Darauf laufen mit Docker Compose drei Container: die Next.js-App, PostgreSQL und Caddy als Reverse Proxy, der sich automatisch HTTPS-Zertifikate von Let's Encrypt holt. Ein Cronjob macht jede Nacht ein `pg_dump` und lädt es auf eine Hetzner Storage Box.

- **Vorteile:** Am günstigsten. Deutsche Firma, Rechenzentrum in Deutschland. Volle Kontrolle. Dieselbe `docker-compose.yml` läuft auch lokal, also gibt es kaum Unterschiede zwischen Entwicklung und Betrieb.
- **Nachteile:** Wir kümmern uns selbst um Updates des Servers, die Firewall und die Backups. Wenn der Server ausfällt, ist alles weg, bis wir ihn neu aufsetzen (die Backups liegen aber woanders).

### Option 2: Render (Region Frankfurt)

Platform as a Service: Man verbindet das GitHub-Repository, und Render baut und deployt bei jedem Push. Die PostgreSQL-Datenbank gibt es dort als verwalteten Dienst dazu.

- **Vorteile:** Kein Server zu verwalten. HTTPS, Deployments und Datenbank-Backups sind eingebaut.
- **Nachteile:** Teurer, sobald man aus dem Free-Plan raus ist. Im Free-Plan schläft die App nach Inaktivität ein (erster Aufruf dauert dann lange, das verletzt NFA-03). Render ist eine US-Firma, auch wenn die Server in Frankfurt stehen.

### Option 3: Vercel + Neon

Vercel ist der Hersteller von Next.js und hostet es serverless. Neon bietet serverless PostgreSQL, auch in der EU.

- **Vorteile:** Für Next.js am einfachsten. Beide haben großzügige Free-Pläne.
- **Nachteile:** Der Hobby-Plan von Vercel ist nur für nicht-kommerzielle Nutzung erlaubt. Für das Semester ist das egal, aber nicht für einen echten Dienstleister. Unsere Buchung mit `SELECT … FOR UPDATE` funktioniert auch hier, weil die Transaktion kurz ist und in einer Anfrage bleibt. Sie braucht aber eine direkte (nicht gepoolte) Datenbankverbindung, also zusätzliche Konfiguration. Zwei US-Firmen, und kalte Starts der Funktionen machen den ersten Aufruf langsamer.

## Kostenvergleich

Monatliche Kosten für unseren Umfang. Die Preise stammen von den Preisseiten der Anbieter bzw. aus Vergleichsseiten, Stand November 2026. Hetzner hat die Preise im Juni 2026 erhöht. **Vor der Abgabe nochmal auf den Anbieterseiten prüfen.**

| | Option 1: Hetzner | Option 2: Render | Option 3: Vercel + Neon |
|---|---|---|---|
| App | Server CX23 (2 vCPU, 4 GB RAM, 40 GB): ca. 5,49 € | Web Service „Starter": ca. 7 $ | Hobby: 0 $ (nicht kommerziell), Pro: 20 $ pro Person |
| Datenbank | im selben Server enthalten | Postgres „Basic-256mb": ca. 6 $ + Speicher | Neon Free: 0 $ |
| IPv4-Adresse | ca. 0,50 € | enthalten | enthalten |
| Backups | Storage Box BX11: ca. 4 € (oder Hetzner-Backups für +20 % vom Serverpreis, ca. 1,10 €) | enthalten | im Plan enthalten |
| HTTPS | Caddy + Let's Encrypt: 0 € | enthalten | enthalten |
| **Summe ca.** | **7 bis 10 € / Monat** | **ca. 13 bis 15 $ / Monat** | **0 $ im Semester, ab kommerzieller Nutzung ab 20 $** |
| Domain | ca. 10 bis 15 € / Jahr, bei allen gleich | | |

## E-Mail-Versand

| | Brevo | Resend |
|---|---|---|
| Firma / Sitz | Frankreich (EU) | USA |
| Free-Plan | 300 Mails pro Tag, mit „Sent with Brevo"-Hinweis | 3000 Mails pro Monat |
| Danach | Starter ab ca. 9 $ / Monat | ab ca. 20 $ / Monat |

Wir brauchen nur ein paar Mails pro Woche (neue Kunden, Paketanfragen, später Warnungen bei wenig Guthaben). Der Free-Plan von Brevo reicht also weit, und die Daten bleiben in der EU. In der lokalen Entwicklung werden Mails nur in die Konsole geschrieben.

## Entscheidung

Wir nehmen **Option 1: Hetzner Cloud mit Docker Compose** und Brevo für E-Mails. Die Begründung steht in [ADR-003](adr/003-hosting-hetzner-docker-compose.md). Kurz:

- Ein echter Dienstleister kann das Portal ohne Umbau weiter nutzen. Es gibt keine Einschränkung wie „nicht kommerziell" (Vercel Hobby) und kein Einschlafen (Render Free).
- Alle Auftragsverarbeiter sind EU-Firmen (Hetzner, Brevo).
- Ein dauerhaft laufender Prozess: keine kalten Starts (NFA-03), und die Transaktionen aus ADR-002 laufen ohne Sonderkonfiguration.
- Am günstigsten auf Dauer.
- Die Arbeit für den Server (Updates, Firewall, Backups) ist überschaubar und unten beschrieben.

## Betrieb (Überblick)

- **Server:** Ubuntu LTS, automatische Sicherheitsupdates (`unattended-upgrades`), SSH nur mit Schlüssel, Hetzner-Firewall lässt nur 22, 80 und 443 durch. Port 5432 (PostgreSQL) ist von außen nicht erreichbar.
- **Deploy:** GitHub Actions baut bei jedem Merge auf `main` ein Docker-Image und schiebt es in die GitHub Container Registry. Dann verbindet sich die Action per SSH mit dem Server. Dafür gibt es einen eigenen Deploy-Benutzer, dessen Schlüssel nur als GitHub-Secret gespeichert ist. Auf dem Server laufen dann:
  1. `docker compose pull`
  2. `docker compose run --rm migrate`: Migrationen als eigener Schritt. Schlägt eine fehl, bricht das Deployment ab, und die alte Version läuft weiter.
  3. `docker compose up -d`
- **Geheimnisse:** Datenbank-Passwort, Better-Auth-Secret und Brevo-API-Key liegen in einer `.env`-Datei nur auf dem Server, nie im Repository.
- **Backups:** Jede Nacht `pg_dump` auf die Storage Box, 14 Tage aufbewahrt. Die Wiederherstellung üben wir einmal **vor der MVP-Demo**, damit wir wissen, dass sie funktioniert, bevor es darauf ankommt.
- **Überwachung:** Ein kostenloser Uptime-Dienst ruft alle 5 Minuten `/api/health` auf und schickt eine E-Mail, wenn die Seite nicht antwortet. Die Docker-Logs werden auf 14 Tage begrenzt (`max-size`, `max-file`), damit die Festplatte nicht voll läuft. Fehlgeschlagene E-Mails stehen dort mit Empfänger und Grund.
- **E-Mail-Zustellung:** SPF und DKIM für die Absender-Domain bei Brevo einrichten. Sonst landen Einladungen und Reset-Links im Spam.
- **Branches:** Gearbeitet wird auf `dev`. Was auf `main` gemergt wird, geht live.
