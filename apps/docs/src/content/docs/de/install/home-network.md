---
title: Kept im Heimnetz
description: Kept im Heimnetz über einfaches HTTP betreiben - was funktioniert, was HTTPS braucht und warum, und die zwei Wege zu HTTPS ohne öffentliche Domain - Tailscale, oder Caddy mit eigener Zertifizierungsstelle.
sidebar:
  order: 3
---

Kept funktioniert in deinem Heimnetz unter einer einfachen `http://`-Adresse, etwa
`http://192.0.2.20:8080` oder `http://kept.home.arpa:8080`. Deine eigenen Daten funktionieren dort:
Gegenstände hinzufügen, finden und verschieben, Fotos und Dateien, Etiketten, Berichte, Importe,
und Exporte ohne geheime Werte.

Ein paar Dinge brauchen HTTPS. Manche, weil Kept sich weigert, einen Schlüssel herauszugeben oder
ein Geheimnis anzuzeigen, über eine Verbindung, die jeder im Netz mitlesen kann; andere, weil
Browser sie nur unter einer sicheren Adresse erlauben, und daran kann Kept nichts ändern.

## Was über einfaches HTTP funktioniert

| | Über einfaches HTTP | Warum |
|---|---|---|
| Gegenstände, Orte, Suche, Verschieben, Etiketten, Berichte | Funktioniert | |
| Fotos und Dateien, vom Gerät ausgewählt | Funktioniert | |
| Import (Homebox, CSV, ein Kept-Export) | Funktioniert | |
| Die Geheimnisse eines Exports beim Import öffnen | Braucht HTTPS. Über einfaches HTTP wird der Rest des Exports ohne sie importiert | Kept: die Passphrase und die geheimen Werte |
| Export ohne geheime Werte | Funktioniert. Jeder geheime Wert wird weggelassen und als weggelassen markiert; der Export lässt sich wie gewohnt importieren | Deine eigenen Daten |
| Einen geheimen Wert setzen | Funktioniert | |
| Einen geheimen Wert anzeigen | Braucht HTTPS | Kept: Ein Geheimnis wird nie über einfaches HTTP gesendet |
| Export mit geheimen Werten | Braucht HTTPS | Kept: die Geheimnisse und ihre Passphrase |
| Tokens für KI-Apps (MCP) und Home Assistant | Nicht aus dem Browser. Leg eines auf dem Server an: `kept admin token create` ([CLI](/de/admin/cli/#ein-token-vom-server)) | Kept: Ein Token ist ein Schlüssel |
| Eine App per OAuth verbinden (ein Connector) | Braucht HTTPS | Kept: OAuth braucht eine sichere Adresse |
| Das Wiederherstellungskit herunterladen | Nicht aus dem Browser. Auf dem Server: `kept admin recovery-kit` ([CLI](/de/admin/cli/)) | Kept: Es enthält die Schlüssel des Servers |
| Sicherungseinstellungen und ihr Test | Braucht HTTPS, oder setze sie in der Umgebung des Servers ([Konfiguration](/de/reference/configuration/)) | Kept: das Passwort der Sicherung |
| Passkeys | Braucht HTTPS | Der Browser: keine Passkeys unter einer einfachen HTTP-Adresse |
| Die Kamera, Kept installieren, offline arbeiten, Push-Benachrichtigungen, Standort | Braucht HTTPS | Der Browser |

Über einfaches HTTP hält eine Anmeldung außerdem 12 Stunden und wird nicht verlängert; über HTTPS
hält sie 30 Tage. Wie du die 12 Stunden änderst, steht unter
[Im eigenen Netz bleiben](#im-eigenen-netz-bleiben).

„Erste Schritte“ auf der Startseite bittet einen Instanz-Admin, Kept auf HTTPS umzustellen. Bleibst
du in deinem Heimnetz, wähle **Ich bleibe in meinem Heimnetz**, dann gilt der Schritt als erledigt.
Die HTTPS-Zeile oben auf jeder Seite lässt sich ebenfalls wegklicken.

`localhost` gilt in Browsern als sichere Adresse, aber Kept richtet sich nach `KEPT_PUBLIC_URL`:
Solange dort `http:` steht, gelten die Ablehnungen von Kept oben auch auf `localhost`.

## Im eigenen Netz bleiben

Wer Kept betreibt, kann auf dem Server zwei Dinge ändern: wie lange eine Anmeldung über einfaches
HTTP hält, und unter welchen Adressen Kept antwortet. Beides lässt sich nicht in der App ändern, und
Admin → Status zeigt, was gerade gilt.

### Längere Anmeldungen über einfaches HTTP

Über einfaches HTTP meldest du dich alle 12 Stunden neu an. `KEPT_HTTP_SIGN_IN_DAYS` in `.env`
macht daraus so viele Tage, 1 bis 30, verlängert, solange du Kept nutzt. 7 passt zu einem Heimnetz
oder zu Tailscale:

```sh
KEPT_HTTP_SIGN_IN_DAYS=7
```

Dann `docker compose up -d`. In der Home-Assistant-App ist es die Option **Sign-in length over
http**.

Eine Installation, die von 1.14.4 nur durch Ändern von `KEPT_IMAGE` aktualisiert wurde, hat noch die
alte `compose.yaml`, die diese Einstellung nicht weitergibt:
[lade zuerst die neue herunter](/de/admin/upgrades/#einstellungen-die-die-neue-composeyaml-brauchen).

**Der Preis:** Über einfaches HTTP kann jeder, der den Datenverkehr sieht, ein Anmelde-Cookie
kopieren. Mit der Einstellung funktioniert ein kopiertes so lange weiter, wie es benutzt wird, bis
sich dieses Gerät abmeldet oder deine anderen Geräte abgemeldet werden. Nutze sie also nur in einem
Netz, dem du vertraust, und melde ein verlorenes Gerät ab.

| | Nicht gesetzt | Gesetzt |
|---|---|---|
| Eine Anmeldung über einfaches HTTP hält | 12 Stunden | So viele Tage |
| Verlängert | Nein | Einmal pro Tag der Nutzung (bei 1 Tag nach einem halben Tag) |
| Über HTTPS | 30 Tage, verlängert | 30 Tage, verlängert: Die Einstellung bewirkt nichts |
| Geheimnisse und Schlüssel über einfaches HTTP | Abgelehnt | Abgelehnt |
| Sensible Aktionen | Passwort, oder eine Anmeldung in den letzten zehn Minuten | Genauso; eine Verlängerung zählt nicht als Anmeldung |
| Überall abmelden | Beendet jede Anmeldung | Beendet jede Anmeldung |

Jeder andere Wert (0, 31, `7d`) hält Kept beim Start an, mit einer Meldung, was es erwartet. Leer
lassen ist dasselbe wie weglassen.

Eingeschaltet bekommt eine bestehende Anmeldung die längere Dauer, sobald sie das nächste Mal
benutzt wird. Ausgeschaltet läuft eine von selbst aus, nach höchstens der Dauer, die sie hatte; um
sie früher zu beenden, nutze **Überall abmelden** unter Admin → Benutzer.

### Mehr als eine Adresse

Kept nimmt Anmeldungen und Speichervorgänge nur von `KEPT_PUBLIC_URL` an. Ein zweiter Weg zum
selben Kept, ein Tailscale-Name neben einem Namen im Heimnetz oder eine IP neben einem Namen, sagt
deshalb „Dieses Kept ist für … eingerichtet“. `KEPT_EXTRA_ADDRESSES` listet die anderen, durch
Kommas getrennt, genau so, wie der Browser sie anzeigt:

```sh
KEPT_PUBLIC_URL=http://kept.lan:8080
KEPT_EXTRA_ADDRESSES=http://192.0.2.20:8080,http://<machine>.<tailnet>.ts.net:8080
```

Dann `docker compose up -d`. In der Home-Assistant-App ist es die Option **Extra addresses**. Über
HTTPS funktioniert es genauso.

Eine Installation, die von 1.14.4 nur durch Ändern von `KEPT_IMAGE` aktualisiert wurde, hat noch die
alte `compose.yaml`, die diese Einstellung nicht weitergibt:
[lade zuerst die neue herunter](/de/admin/upgrades/#einstellungen-die-die-neue-composeyaml-brauchen).

| | `KEPT_PUBLIC_URL` | Eine zusätzliche Adresse |
|---|---|---|
| Kept nutzen, mit Passwort oder Einmalcode anmelden | Ja | Ja |
| Ein E-Mail-Link oder Passwort-Reset | Öffnet sich hier | Öffnet sich unter `KEPT_PUBLIC_URL` |
| Passkeys | Ja, über HTTPS | Nein; ein Passkey gehört zu einer Adresse |
| Anmeldung mit Google und OIDC | Ja | Der Knopf öffnet `KEPT_PUBLIC_URL` |
| KI-Apps (MCP, OAuth) | Ja | Verbinde sie unter `KEPT_PUBLIC_URL`: `/mcp` lehnt diese Adresse ab, und die OAuth-Zustimmung wird hier abgelehnt |
| Links, Etiketten, QR-Codes, Mails, Kalender-Feeds | Zeigen hierher | Zeigen weiter auf `KEPT_PUBLIC_URL` |

- **Jede Adresse meldet sich getrennt an und wird als eigene App installiert,** weil Browser Cookies
  und Offline-Daten pro Adresse speichern. Abmelden an der einen lässt die andere angemeldet;
  **Überall abmelden** beendet beide. Push-Benachrichtigungen, wo HTTPS sie erlaubt, öffnen sich
  unter der Adresse, an der das Gerät sie eingeschaltet hat. Zwei Ports unter einem Hostnamen teilen
  sich ihre Cookies: Browser trennen die nicht nach Port.
- **Ein E-Mail-Link**, der an einer zusätzlichen Adresse angefordert wurde, öffnet sich unter
  `KEPT_PUBLIC_URL` und meldet dich dort an, nicht dort, wo du ihn angefordert hast. Ein
  Passwort-Reset setzt das Passwort dort; melde dich danach an der Adresse an, an der du warst. Ist
  `KEPT_PUBLIC_URL` von deinem Standort aus nicht erreichbar, nutze dein Passwort.
- **Passkeys:** Leg sie unter `KEPT_PUBLIC_URL` an und nutze sie dort. Eine zusätzliche Adresse
  bietet dein Passwort oder einen E-Mail-Link an und sagt, warum.
- **Exakte Adressen:** Schema, Host und Port, als Ganzes verglichen. `http://kept.lan` ist nicht
  `http://kept.lan:8080`, und `kept.lan.example.com` ist nicht `kept.lan`. Ein Platzhalter, ein
  Pfad, Zugangsdaten oder eine Mischung aus `http` und `https` halten Kept beim Start an, mit einer
  Meldung, woran es liegt. Leer lassen ist dasselbe wie weglassen.
- **Weitergeleitete Header:** `Host` und `X-Forwarded-*` spielen keine Rolle, also kann dein Proxy
  nicht versehentlich eine Adresse hinzufügen oder wegnehmen. Richte ihn für jede Adresse, die du
  einträgst, auf Kept.

**Der Preis:** Eine Seite, die unter einer eingetragenen Adresse ausgeliefert wird, gilt als Kept
selbst und kann die Anmeldung dessen nutzen, der sie dort öffnet. Trag also nur Adressen ein, die
dieses Kept erreichen, unter Namen und IPs, die du kontrollierst.

## HTTPS hinzufügen: zuerst Tailscale

Am wenigsten Arbeit für einen Haushalt: Installiere Tailscale auf dem Rechner, auf dem Kept läuft,
und auf jedem Handy und Computer, der es nutzt, schalte HTTPS-Zertifikate für dein Tailnet ein und
stelle Kept bereit mit

```sh
tailscale serve --bg 8080
```

Kept liegt dann unter `https://<machine>.<tailnet>.ts.net`, mit einem Zertifikat, dem jedes Gerät
schon vertraut, zu Hause wie unterwegs. Setze `KEPT_PUBLIC_URL` auf diese Adresse und starte Kept
neu. Mehr unter [Kept auf HTTPS umstellen](/de/install/https/#tailscale).

## HTTPS hinzufügen: Caddy mit eigener Zertifizierungsstelle

Ohne Tailscale und ohne öffentliche Domain kann Caddy selbst ein Zertifikat für einen Namen im
Heimnetz signieren (`tls internal`). Das funktioniert, ist aber nicht schnell gemacht: **Jedem Gerät,
das Kept öffnet, muss man sagen, dass es dem Root-Zertifikat von Caddy vertrauen soll**, eines nach
dem anderen, für jede Person im Haus. Bis ein Gerät ihm vertraut, zeigt sein Browser eine
Sicherheitswarnung.

1. Wähle einen Namen, den jedes Gerät zum Rechner auflösen kann, zum Beispiel `kept.home.arpa` (ein
   DNS-Eintrag auf deinem Router oder ein Eintrag in der hosts-Datei jedes Geräts).
2. Trag ihn in `.env` ein:

   ```sh
   KEPT_DOMAIN=kept.home.arpa
   KEPT_PUBLIC_URL=https://kept.home.arpa
   ```

3. Caddy kommt mit `compose.full.yaml`: Lade sie und `docker/caddy/Caddyfile` herunter und setze
   `COMPOSE_FILE=compose.full.yaml` in `.env`, wie unter
   [Kept auf HTTPS umstellen](/de/install/https/#das-caddy-profil-von-compose). Ergänze dann
   `tls internal` im Caddyfile:

   ```caddy
   {$KEPT_DOMAIN} {
   	tls internal
   	encode zstd gzip
   	reverse_proxy kept:8080
   }
   ```

4. Starte Caddy und kopiere sein Root-Zertifikat aus seinem Daten-Volume:

   ```sh
   docker compose --profile https up -d
   docker compose cp caddy:/data/caddy/pki/authorities/local/root.crt ./kept-root.crt
   ```

5. Installiere `kept-root.crt` auf jedem Gerät und vertraue ihm:
   - **iPhone und iPad:** Schick es aufs Gerät (AirDrop oder Mail), installiere das Profil unter
     Einstellungen → Allgemein → VPN und Geräteverwaltung, und schalte dann unter Einstellungen →
     Allgemein → Info → Zertifikatsvertrauenseinstellungen volles Vertrauen dafür ein.
   - **Android:** In den Sicherheitseinstellungen als CA-Zertifikat installieren (der Name des Menüs
     hängt vom Hersteller ab). Manche Apps ignorieren Zertifikate, die der Nutzer installiert hat.
   - **Computer:** Füge es den vertrauenswürdigen Zertifikaten des Systems hinzu (Schlüsselbundverwaltung
     auf dem Mac, die Zertifikatsverwaltung unter Windows, der CA-Speicher der Distribution unter
     Linux). Firefox führt unter Umständen eine eigene Liste.

Das Root-Zertifikat von Caddy hält zehn Jahre; die Zertifikate, die es damit signiert, erneuern
sich von selbst. Der private Schlüssel der Root bleibt im Volume `caddy-data`: Wer ihn kopiert, kann
für jeden Namen signieren, dem deine Geräte jetzt vertrauen, also schütze dieses Volume wie ein
Passwort. [mkcert](/de/install/https/#ein-zertifikat-dem-nur-deine-geräte-vertrauen) ist dieselbe
Idee mit einem anderen Tool.

## Kept unter der richtigen Adresse öffnen

Öffne Kept unter der Adresse in `KEPT_PUBLIC_URL`. Öffnest du es unter einer anderen (etwa der
IP-Adresse des Rechners, während die URL `kept.home.arpa` nennt), sagt die Anmeldung, für welche
Adresse dieses Kept eingerichtet ist, mit einem Link dorthin.
