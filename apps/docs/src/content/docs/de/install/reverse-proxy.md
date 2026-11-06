---
title: Hinter einem Reverse Proxy
description: Die Einstellungen, auf die es ankommt, wenn Kept hinter nginx, Traefik, Caddy oder einem anderen Proxy läuft - vertrauenswürdige Adressen, Upload-Größen, welcher Hostname, der Adressfehler und HSTS.
sidebar:
  order: 3
---

Kept lauscht auf Port 8080 und spricht einfaches HTTP; der Proxy davor terminiert TLS. Richte den
Proxy auf Port 8080, setze `KEPT_PUBLIC_URL` auf die Adresse, die die Leute öffnen, und prüfe die
Punkte unten. Das [Caddy-Profil](/de/install/https/#das-caddy-profil-von-compose) von Compose
erledigt die ersten drei bereits.

## Dem Proxy vertrauen, und nur dem Proxy

Kept begrenzt Anmeldungen, Einrichtungscodes und andere sensible Anfragen **pro Client-Adresse**.
Hinter einem Proxy kommt die Adresse des Clients in `X-Forwarded-For` an, und das kann jeder
schreiben. Kept glaubt ihr nur von den Adressen in `KEPT_TRUSTED_PROXIES`:

```sh
KEPT_TRUSTED_PROXIES=10.0.0.5            # ein Proxy
KEPT_TRUSTED_PROXIES=10.0.0.5,10.0.1.0/24  # mehrere, oder ein Bereich
```

Kept liest `X-Forwarded-For` von rechts und hält bei der ersten Adresse an, der es nicht vertraut:
Das ist der Client. Leer (die Vorgabe) gilt die Adresse des Sockets als Client. Trag die Adresse des
Proxys so ein, wie Kept sie sieht (im Docker-Netz die Adresse des Proxy-Containers, nicht die des
Hosts), und nichts, von dem aus ein Fremder Anfragen schicken könnte.

Mit Compose gehört sie in `.env` neben `compose.yaml`. Vor 1.14.5 setzte `compose.yaml` einen
eigenen Wert, und eine Zeile in `.env` kam nie bei Kept an; setze sie bei diesen Versionen für den
Dienst `kept` in einer `compose.override.yaml`. Mit dem Caddy aus `compose.full.yaml` lass sie leer
(die Adresse von Caddy ist die Vorgabe) oder behalte `172.29.72.200` in der Liste.

### Die Sperre bei der Einrichtung

Falsche Einrichtungscodes sperren eine Zeit lang die Adresse, von der sie kamen, und viele falsche
Codes sperren die ganze Instanz. Steht der Proxy nicht in `KEPT_TRUSTED_PROXIES`, teilen sich alle
Besucher die Adresse des Proxys, und die Fehlversuche einer Person sperren alle aus.
`kept admin setup-code` stellt einen neuen Code aus und hebt die Sperre der ganzen Instanz auf,
aber nicht die für eine Adresse: Die endet von selbst.

## Uploads durchlassen

Der Proxy muss einen Request-Body annehmen, der mindestens so groß ist wie der, den Kept annimmt,
und sollte Uploads nicht puffern, wenn er nicht muss:

| Upload | Grenze von Kept |
|---|---|
| Ein Foto, Beleg oder Dokument | `KEPT_MAX_FILE_MB`, standardmäßig 25 MB; gestreamt, und mit 413 abgelehnt, bevor es gelesen wird, wenn es zu groß ist |
| Ein CSV-Import | 8 MB |

Die Vorgabe der meisten Proxys ist kleiner (bei nginx etwa `client_max_body_size`); setz sie auf
mindestens `KEPT_MAX_FILE_MB`. Die Zeitlimits sollten einen großen Upload über eine langsame
Handyverbindung zulassen.

## Kept einen eigenen Hostnamen geben

Stell Kept unter einem Hostnamen bereit, auf dessen übergeordneter Domain keine Seiten laufen, die
du nicht kontrollierst, oder unter einer eigenen Domain. Über HTTPS trägt das Sitzungs-Cookie von
Kept das Präfix `__Secure-`. Das lässt trotzdem zu, dass eine Seite auf einer
**benachbarten** Subdomain (`other.example.org` neben `kept.example.org`) ein Cookie für
`example.org` setzt, das Browser auch an Kept schicken. Eine Nachbarseite, die jemand anderes
betreibt, könnte damit eine Sitzung unterschieben (Session Fixation). Deine eigenen anderen Dienste
auf benachbarten Subdomains sind kein Problem.

## „Dieses Kept ist für … eingerichtet“

Die App sagt „Dieses Kept ist für *Adresse* eingerichtet. Öffne es dort.“, wenn sie unter einer
anderen Adresse als `KEPT_PUBLIC_URL` geöffnet wird.

- **Warum:** Kept nimmt Schreibzugriffe nur von der Adresse in `KEPT_PUBLIC_URL` an, gemessen am
  `Origin` des Browsers: Schema, Host und Port, exakt. Der Proxy spielt keine Rolle; seine eigene
  Adresse und `X-Forwarded-*`-Header ändern daran nichts.
- **Lösung:** Setze `KEPT_PUBLIC_URL` auf genau die Adresse, die die Leute öffnen
  (`https://kept.example.org`, nicht das Upstream-Ziel des Proxys `http://kept:8080`), und starte
  Kept neu.
- **Mehr als eine Adresse:** `KEPT_EXTRA_ADDRESSES` listet die anderen, und jede muss Kept ebenfalls
  über deinen Proxy erreichen ([Mehr als eine Adresse](/de/install/home-network/#mehr-als-eine-adresse)).

Mehr unter [Fehlerbehebung](/de/admin/troubleshooting/#dieses-kept-ist-für--eingerichtet-oder-die-anfrage-kam-nicht-von-dieser-website).

## HSTS

Mit einer `https:`-Adresse in `KEPT_PUBLIC_URL` sendet Kept
`Strict-Transport-Security: max-age=31536000`: Browser öffnen diesen Hostnamen dann ein Jahr lang
nur noch über https. Über `http:` sendet es nichts.

- **Ganzer Hostname, jeder Port:** Ein Browser wendet die Regel auf den Hostnamen an, nicht auf den
  Port von Kept. Ein Dienst mit einfachem http unter demselben Namen, etwa `http://server.lan:8081`
  neben Kept unter `https://server.lan`, wird dann auf https gezwungen und lädt nicht mehr. Von
  einer IP-Adresse ignorieren Browser HSTS.
- **Abschalten:** Setze `KEPT_HSTS=off`, wenn sich andere Dienste mit einfachem http den Hostnamen
  von Kept teilen, und starte neu. Kept sendet dann `max-age=0`, und ein Browser, der die Regel
  gespeichert hat, vergisst sie beim nächsten https-Besuch bei Kept. Den Header einfach wegzulassen,
  würde sie nicht löschen.
- **Dein Proxy:** Setzt er einen eigenen `Strict-Transport-Security`-Header, schalte den dort
  ebenfalls ab.
- **Vorab geladene Domains:** Unter einer Top-Level-Domain auf der HSTS-Preload-Liste der Browser,
  etwa `.dev` oder `.app`, ist jeder Name nur über https erreichbar, egal was Kept sendet.

### Eine gespeicherte Regel von Hand löschen

Ist Kept nicht mehr über https erreichbar, kommt `max-age=0` nicht beim Browser an. Lösche den
Hostnamen in jedem Browser, der ihn besucht hat:

- **Chrome, Edge und andere Chromium-Browser:** Öffne `chrome://net-internals/#hsts` (in Edge
  `edge://net-internals/#hsts`), gib den Hostnamen unter **Delete domain security policies** ein und
  wähle **Delete**.
- **Firefox:** Chronik → Gesamte Chronik anzeigen, die Seite suchen, mit der rechten Maustaste
  anklicken und **Gesamte Website vergessen** wählen. Das entfernt auch Chronik, Cookies und
  gespeicherte Zugangsdaten der Seite.
- **Safari auf dem Mac:** hat keine Seite für HSTS. Entferne die Daten der Website (Einstellungen →
  Datenschutz → Websitedaten verwalten, auswählen, **Entfernen**); wird einfaches http trotzdem zu
  https, lösche auch den Verlauf (Verlauf → Verlauf löschen).
- **Safari auf iPhone oder iPad:** Einstellungen → Apps → Safari (vor iOS 18 Einstellungen →
  Safari) → Erweitert → Websitedaten, die Seite suchen und löschen; reicht das nicht, auf der
  Safari-Seite **Verlauf und Websitedaten löschen**.
