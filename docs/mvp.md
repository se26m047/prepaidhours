# MVP

## Was wir zeigen wollen

Im MVP soll man einen kompletten Ablauf durchspielen können:

1. Der Admin legt die Firma „Muster GmbH" an.
2. Der Kunde setzt über den Link sein Passwort und loggt sich ein.
3. Er fragt ein Paket mit 5 Stunden an.
4. Der Admin markiert das Paket als bezahlt. Der Kunde sieht jetzt 5:00 h.
5. Der Admin bucht 3 h und dann 2 h. Der Kunde sieht beide Einträge und hat 0:00 h.
6. Der Admin versucht noch 15 Minuten zu buchen. Das wird abgelehnt.

## Was drin ist

Alle Must-Stories, also US-01 bis US-08 und US-12: Login, Guthaben, Verlauf, Paket anfragen, Kunden und Pakete anlegen, Arbeit buchen, der Schutz gegen Überbuchen und Passwort vergessen.

## Was später kommt

Wenn im Semester noch Zeit bleibt:
- Warnung bei wenig Guthaben (US-09)
- Buchungen stornieren (US-10)
- CSV-Export (US-11)

## Was wir bewusst weglassen

- **Online bezahlen (z. B. Stripe):** Das würde viel Zeit kosten und hat Gebühren. Rechnung schicken und dann „bezahlt" klicken reicht.
- **Rechnungen als PDF erstellen:** Rechnungen haben rechtliche Vorgaben. Die schreibt der Dienstleister weiter in seinem Buchhaltungsprogramm.
- **Stoppuhr zum Zeit messen:** Die Admins tragen die Zeit nachher ein, das reicht.
- **Eigene App:** Die Webseite funktioniert auch am Handy.
- **Mehrere Sprachen:** Die Kunden sprechen Deutsch.

## Externe Dienste

Wir brauchen einen Dienst zum E-Mails verschicken und ein Hosting in der EU. Für unseren Umfang reicht bei beidem ein kostenloser oder sehr günstiger Plan. Falls das E-Mail-Versenden mal nicht klappt, geht die Paketanfrage trotzdem nicht verloren, weil sie im System gespeichert ist und der Admin sie sieht.
