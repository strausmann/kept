---
title: Kept auf HTTPS umstellen
description: Warum Kept HTTPS will, und wie du es bekommst - Caddy aus compose.full.yaml, Tailscale, NetBird, Twingate, Cloudflare Tunnel oder dein eigener Proxy.
sidebar:
  order: 2
---

Kept funktioniert über einfaches HTTP, aber ein Handy behandelt eine HTTP-Adresse als unsicher, und
mehreres funktioniert dann nicht:

- **die Kamera** (Kept weicht auf eine Dateiauswahl aus),
- **Kept installieren** auf den Startbildschirm, und offline damit arbeiten,
- **Push-Benachrichtigungen**,
- **der Standort**.

`localhost` gilt als sicher; eine Adresse im Heimnetz über HTTP nicht. Mit einer `https:`-Adresse
in `KEPT_PUBLIC_URL` markiert Kept seine Cookies außerdem als `Secure` und sendet HSTS;
`KEPT_HSTS=off` hebt das auf, wenn Dienste mit einfachem http denselben Hostnamen nutzen
([HSTS](/de/install/reverse-proxy/#hsts)).

Manches lehnt Kept ganz ab, solange `KEPT_PUBLIC_URL` mit `http:` beginnt, selbst auf `localhost`,
weil es ein Geheimnis herausgibt oder ändert: einen geheimen Wert anzeigen, ein Export mit geheimen
Werten, die Geheimnisse eines Exports beim Import öffnen, im Browser ein Zugriffstoken anlegen,
eine App per OAuth verbinden, das Wiederherstellungskit herunterladen, und die
Sicherungseinstellungen ändern oder testen. Ein Export ohne geheime Werte funktioniert, ebenso ein
Import ohne sie. Über HTTP sind auch die Sitzungen kürzer. Solange Kept über HTTP läuft, stellt die
Checkliste „Erste Schritte“ auf der Startseite „Kept auf HTTPS umstellen“ an den Anfang, bis ein
Admin „Ich bleibe in meinem Heimnetz“ wählt. Alles, was über einfaches HTTP funktioniert, in einer
Tabelle: [Kept im Heimnetz](/de/install/home-network/).

Welchen Weg du auch wählst: Setze `KEPT_PUBLIC_URL` auf die `https://`-Adresse, die die Leute
öffnen werden, und starte Kept neu.

## Das Caddy-Profil von Compose

Für einen Domainnamen, der auf diesen Rechner zeigt, mit den Ports 80 und 443 aus dem Internet
erreichbar (über sie weist Caddy die Domain bei Let's Encrypt oder ZeroSSL nach). Caddy kommt mit
`compose.full.yaml`, nicht mit `compose.yaml` ([Caddy und Ollama](/de/install/compose/#caddy-und-ollama)):

```sh
# in .env
COMPOSE_FILE=compose.full.yaml
KEPT_DOMAIN=kept.example.org
KEPT_PUBLIC_URL=https://kept.example.org
```

Caddy liest `docker/caddy/Caddyfile` neben der Compose-Datei. Eine Kopie des Repositorys hat beide
Dateien; eine Installation ohne eine lädt sie einmal herunter:

```sh
curl -fsSLO https://raw.githubusercontent.com/keptinventory/kept/v1.14.5/compose.full.yaml
mkdir -p docker/caddy
curl -fsSLo docker/caddy/Caddyfile https://raw.githubusercontent.com/keptinventory/kept/v1.14.5/docker/caddy/Caddyfile
docker compose --profile https up -d
```

Ohne die Datei bricht `docker compose --profile https up -d` mit „bind source path does not exist“
ab. Jeder andere Proxy funktioniert stattdessen; er leitet an den Port von Kept weiter, wie in den
Abschnitten unten.

Caddy holt und erneuert das Zertifikat und leitet alles an Kept weiter. Compose gibt Caddy eine
feste Adresse in einem eigenen Netz und übergibt diese Adresse an Kept als `KEPT_TRUSTED_PROXIES`,
sodass Kept dem `X-Forwarded-For` von Caddy glaubt und sonst niemandem. Einfaches HTTP kannst du
dann nur an Loopback binden, mit `KEPT_HTTP_PORT=127.0.0.1:8080`.

Kollidiert `172.29.72.0/24` mit deinem Heimnetz, ändere in `compose.full.yaml` gemeinsam das
Subnetz des Netzes, seinen `ip_range`, die `ipv4_address` von Caddy und `KEPT_TRUSTED_PROXIES`.

## Tailscale

Für ein Kept, das nur deine eigenen Geräte erreichen, ohne offenen Port ins Internet: auf dem
Rechner, auf dem Kept läuft, mit eingeschalteten HTTPS-Zertifikaten für dein Tailnet,

```sh
tailscale serve --bg 8080
```

stellt Kept unter `https://<machine>.<tailnet>.ts.net` mit einem echten Zertifikat bereit, für jedes
Gerät im Tailnet. Setze `KEPT_PUBLIC_URL` auf diese Adresse.

Der Proxy von Tailscale läuft auf demselben Rechner und verbindet sich von `127.0.0.1` mit Kept;
Kept sieht jede Anfrage als von dort kommend, solange du ihm nicht vertraust. Bei einer Instanz für
einen einzigen Haushalt spielt das nur für
[die Sperre bei der Einrichtung](/de/install/reverse-proxy/#die-sperre-bei-der-einrichtung) eine
Rolle.

## NetBird

Der Reverse Proxy von NetBird gibt dem Port eines Peers eine öffentliche `https://`-Adresse mit
Zertifikat:

```sh
netbird expose 8080
```

Er gibt die Adresse aus. Setze `KEPT_PUBLIC_URL` darauf (oder auf deine eigene Domain, mit
`--with-custom-domain kept.example.org`, vorher in NetBird eingerichtet). Die Adresse ist
öffentlich, es sei denn, du wählst im Dashboard für den Dienst **NetBird-Only Access**, was sie auf
deine Peers beschränkt. Ein Account-Admin schaltet vorher unter Settings → Clients **Peer Expose**
ein.

## Twingate

Twingate macht Kept für deine Geräte erreichbar, ohne einen Port zu öffnen, aber das Zertifikat ist
deine Sache: Gib Kept einen Hostnamen in einer Domain, die dir gehört (`kept.example.org`), hol dir
dafür ein Zertifikat per DNS-Challenge (Caddy, Traefik oder cert-manager können das) und lege
diesen Hostnamen als Twingate-Ressource an. Setze `KEPT_PUBLIC_URL` auf `https://kept.example.org`.

## Cloudflare Tunnel

Für eine öffentliche Adresse ohne offenen Port. Cloudflare liefert den Hostnamen über HTTPS aus und
leitet über `cloudflared` an Kept weiter:

```sh
cloudflared tunnel login
cloudflared tunnel create kept
cloudflared tunnel route dns kept kept.example.org
cloudflared tunnel run kept
```

mit `~/.cloudflared/config.yml`:

```yaml
tunnel: kept
credentials-file: <home>/.cloudflared/<tunnel id>.json
ingress:
  - hostname: kept.example.org
    service: http://localhost:8080
  - service: http_status:404
```

Setze `KEPT_PUBLIC_URL` auf `https://kept.example.org`. Cloudflare Access kann eine Anmeldung
davorschalten. Eine öffentliche Adresse braucht auch eine Web-App wie claude.ai, um sich
[mit dem MCP-Server von Kept zu verbinden](/de/users/mcp-clients/#connect-as-a-connector-oauth).

`cloudflared` erreicht Kept, wie die anderen Tunnel hier, vom selben Rechner aus; vertraue ihm wie
unter [Reverse-Proxys](/de/install/reverse-proxy/#dem-proxy-vertrauen-und-nur-dem-proxy) beschrieben.

## Dein eigener Reverse Proxy

Jeder Proxy, der TLS terminiert und an Port 8080 weiterleitet, funktioniert. Lies
[Reverse-Proxys](/de/install/reverse-proxy/) für die drei Einstellungen, auf die es ankommt.

## Ein Zertifikat, dem nur deine Geräte vertrauen

In einem Heimnetz ohne Domain kann mkcert ein Zertifikat für die Adresse des Rechners erzeugen,
ausgeliefert über einen beliebigen TLS-Proxy. Jedes Handy muss dann dem Root-Zertifikat von mkcert
vertrauen; auf einem iPhone heißt das, es als Profil zu installieren und unter Einstellungen →
Allgemein → Info → Zertifikatsvertrauenseinstellungen volles Vertrauen einzuschalten. Tailscale ist
meist weniger Arbeit.
