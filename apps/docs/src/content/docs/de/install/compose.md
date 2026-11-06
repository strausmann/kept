---
title: Installation mit Docker Compose
description: Kept und sein Postgres mit Docker Compose auf dem eigenen Server oder im Homelab betreiben, zum ersten Mal anmelden und die Schlüssel sicher aufbewahren.
sidebar:
  order: 1
---

Mit Docker Compose betreibst du Kept auf einer Maschine: Kept selbst und Postgres 18 mit pgvector.
Caddy für HTTPS und Ollama für lokale KI kommen mit einer zweiten Datei
([Caddy und Ollama](#caddy-und-ollama)). Du brauchst Docker mit dem Compose-Plugin. Getestet auf
einer Maschine mit 1 GB RAM und einem CPU-Kern; Kept und seine Datenbank brauchen davon etwa
200–400 MB ([was getestet wurde](/de/install/hardware/)).

Vier Schritte, ohne Kopie des Repositorys und ohne Passwörter, die du dir ausdenken musst:

```sh
mkdir kept && cd kept
curl -fsSLO https://raw.githubusercontent.com/keptinventory/kept/v1.14.5/compose.yaml
docker compose run --rm init
docker compose up -d
```

`compose.yaml` funktioniert für sich allein, du kannst es also auch in Portainer, Dockge und
ähnliche Werkzeuge einfügen (siehe [ohne Terminal](#ohne-terminal)).

:::note[Images]
Releases erscheinen unter `ghcr.io/keptinventory/kept` mit den Tags `1`, `1.2` und `1.2.3`, nie
`latest`; `1.0.0` ist das erste signierte. Dieselben Images, mit demselben Schlüssel signiert,
liegen auf Docker Hub als `keptinventory/kept`. `compose.yaml` startet das Release, mit dem es
veröffentlicht wurde, außer `KEPT_IMAGE` in `.env` nennt ein anderes; `init` trägt dort die
Version ein, mit der es lief. Wenn du magst, [prüf es](/de/admin/verify-release/) vorher. Um einen
eigenen Build zu betreiben, in einem Checkout: `docker build -t kept:local .` und
`KEPT_IMAGE=kept:local`.
:::

## Was läuft

| Dienst | Läuft |
|---|---|
| `db` | Postgres, bleibt an |
| `kept` | Kept auf Port 8080, bleibt an |
| `bootstrap` | einmal bei jedem Start, danach beendet: die Datenbank-Logins und -Erweiterungen |
| `migrate` | einmal bei jedem Start, danach beendet: die Datenbank-Migrationen |
| `init` | nur, wenn du es startest (`docker compose run --rm init`): schreibt `.env` |

## Caddy und Ollama

`compose.yaml` startet nur Kept. `compose.full.yaml` ist dieselbe Datei plus Caddy für HTTPS (das
Profil `https`), Ollama für lokale KI (das Profil `ollama`) und eine auskommentierte Aufteilung in
Web und Worker. Sie funktioniert ebenfalls für sich allein. So nutzt du sie stattdessen:

```sh
curl -fsSLO https://raw.githubusercontent.com/keptinventory/kept/v1.14.5/compose.full.yaml
docker compose -f compose.full.yaml run --rm init
echo COMPOSE_FILE=compose.full.yaml >> .env
docker compose up -d
```

Mit `COMPOSE_FILE` in `.env` liest ein einfaches `docker compose` die Datei `compose.full.yaml`,
also funktioniert jeder Befehl in dieser Doku wie angegeben. Die Dienste, das Netz und die Volumes
von Kept sind in beiden Dateien gleich: Eine Installation kann jederzeit wechseln und behält ihre
Daten.

## 1. `.env` schreiben

```sh
docker compose run --rm init
```

`init` fragt nach **der Adresse, unter der du Kept öffnen wirst**, etwa
`http://kept.home.arpa:8080` oder `https://kept.example.org`. Daraus entstehen die Links, die
QR-Codes auf gedruckten Etiketten und die Prüfung auf dieselbe Website. Sie muss also genau das
sein, was der Browser anzeigt, und sie sollte sich nicht ändern: **Nimm einen Namen statt einer
IP-Adresse.** Ein Heimrouter kann dieser Maschine eines Tages eine neue Adresse geben, und die
gedruckten Etiketten zeigen dann ins Leere. Trag den Namen im DNS deines Routers ein oder in der
hosts-Datei jedes Geräts. Eine IP-Adresse geht trotzdem, wenn du sie bestätigst.

Danach schreibt es `.env` neben `compose.yaml`, mit dieser Adresse (`KEPT_PUBLIC_URL`), dem Port
(`KEPT_HTTP_PORT`, aus einer `http://`-Adresse), dem Image und einem Einrichtungscode
(`KEPT_SETUP_CODE`), und zeigt den Code an. `.env` enthält kein Passwort und ist nur für dich
lesbar. Um die Adresse zu ändern, starte `init` noch einmal: Es aktualisiert diese Zeilen und
behält alles, was du ergänzt hast. Ohne Terminal gibst du die Adresse auf der Befehlszeile mit:
`docker compose run --rm init --url http://kept.home.arpa:8080`.

Alles Weitere ist optional; `compose.env.example` und die
[Konfigurationsreferenz](/de/reference/configuration/) beschreiben jede Variable.

## 2. Starten

```sh
docker compose up -d
```

Compose startet die Datenbank und wartet, bis sie gesund ist. Danach:

- **`bootstrap`** (einmalig, `kept bootstrap`) legt die vier Datenbank-Logins von Kept und die
  Erweiterungen `pg_trgm`, `unaccent` und `vector` an. Beim ersten Start erzeugt es für jedes Login
  ein Passwort in einem Volume; bei jedem Start setzt es jedes Login auf diesen Wert, damit
  Postgres und Kept immer übereinstimmen.
- **`migrate`** (einmalig, `kept migrate`) spielt die Datenbank-Migrationen einmal ein, unter einer
  Sperre.
- **Kept** startet auf Port 8080 (`KEPT_HTTP_PORT` ändert den Port auf dem Host).

## 3. Einrichtung abschließen

Öffne die Adresse, gib den Einrichtungscode ein, den `init` angezeigt hat, und leg das erste Konto
an: Es wird Instanz-Admin. Kept schreibt den Code beim ersten Start auch in sein Log:

```sh
docker compose logs kept | grep "KEPT SETUP CODE"
```

Einen verlorenen Code stellst du so neu aus:

```sh
docker compose run --rm migrate admin setup-code
```

Falsche Codes sperren für eine Weile, je Adresse und für die ganze Instanz. Steht Kept hinter einem
Proxy, dem es nicht vertraut, teilen sich alle die Adresse des Proxys; siehe
[Reverse-Proxys](/de/install/reverse-proxy/).

## 4. Die Schlüssel aufbewahren

Beim ersten Start, wenn `KEPT_SECRET_KEY` und `KEPT_AUTH_SECRET` nicht gesetzt sind, erzeugt Kept
beide im Volume `kept-config` und schreibt eine Zeile ins Log, wo sie liegen. Der geheime Schlüssel
versiegelt jeden geheimen Wert (KI-Schlüssel, die Passwörter, die du in Kept speicherst): **Eine
Sicherung stellt alles wieder her, außer diesen Werten, wenn du nicht auch den Schlüssel hast.**

Gib die Schlüssel aus und bewahre sie woanders als auf diesem Server auf, etwa in einem
Passwort-Manager:

```sh
docker compose exec kept kept admin recovery-kit
```

Unter Admin → Status musst du das Wiederherstellungskit bestätigen, und Kept verlangt das vor dem
ersten geheimen Wert, KI-Schlüssel oder Sicherungseintrag. Siehe
[das Wiederherstellungskit](/de/admin/recovery-kit/).

## 5. Wie es weitergeht

- [Kept auf HTTPS bringen](/de/install/https/): Kamera, App-Installation, Push-Benachrichtigungen
  und Standortfreigabe funktionieren nur über HTTPS.
- [Sicherungen einrichten](/de/admin/backups/).
- E-Mail: Setz `KEPT_SMTP_URL` (`smtp://` oder `smtps://`, mit Zugangsdaten) und `KEPT_SMTP_FROM`
  für Passwort-Zurücksetzungen, Einladungen und Erinnerungen per E-Mail. Ohne sie erreichen
  Erinnerungen die Leute nur in der App, und Admin → Status meldet, dass E-Mail nicht eingerichtet
  ist.

## Was Compose anlegt

| Volume | Enthält | Sichern? |
|---|---|---|
| `kept-db` | die Postgres-Datenbank | über die Sicherung von Kept, nie durch Kopieren des Volumes im laufenden Betrieb |
| `kept-data` | hochgeladene Dateien (mit `KEPT_STORAGE=local`, der Vorgabe) | über die Sicherung von Kept |
| `kept-config` | die erzeugten `KEPT_SECRET_KEY` und `KEPT_AUTH_SECRET` sowie die Passwörter der Logins `kept_app`, `kept_auth` und `kept_system` | das Wiederherstellungskit aufbewahren; die Passwörter entstehen bei Verlust neu |
| `kept-owner` | das Passwort des Logins `kept_owner` | nein; es entsteht bei Verlust neu |
| `caddy-data`, `caddy-config` | die Zertifikate von Caddy (Profil `https` aus `compose.full.yaml`) | nein; Caddy holt neue |

Der Superuser der Datenbank hat kein Passwort, das jemand bräuchte: Er ist nur aus dem
Datenbank-Container heraus erreichbar, als `docker compose exec db psql -U postgres`, und für den
Bootstrap, der das Netz dieses Containers mitbenutzt. Über das Netz braucht jedes Login sein
Passwort. Veröffentliche nie den Port des Dienstes `db` (kein `ports:` daran): Kept erreicht ihn
über das Compose-Netz.

Die Container von Kept laufen als Benutzer ohne Root-Rechte (uid 10001), mit nur lesbarem
Root-Dateisystem, ohne jede Linux-Capability und mit `no-new-privileges`. Der Dienst `migrate`
hält das Login des Datenbank-Eigentümers; der ausliefernde Container sieht es nie (das Volume
`kept-owner` ist dort nicht eingehängt), bis du es ihm für die nächtliche Sicherung gibst
([Sicherungen](/de/admin/backups/#turning-it-on)). `init` ist die eine Ausnahme von der Regel
ohne Root: Es läuft als root, ohne Netz und nur mit den Capabilities, die `.env` in deinem Ordner
anlegen und dir übergeben, und beendet sich dann.

## Ohne Terminal

Portainer, Dockge und ähnliche Werkzeuge übernehmen `compose.yaml` (oder `compose.full.yaml`) wie
es ist. Sie starten `init` nicht, setz deshalb stattdessen zwei Variablen in ihrem
Umgebungseditor: `KEPT_PUBLIC_URL` (die Adresse, wie oben) und `KEPT_SETUP_CODE` (sechs Zeichen
aus 0–9 und A–Z ohne I, L, O und U, etwa `KPT-2X9`; oder lass es leer und lies den Code aus dem Log
von Kept). Leg `KEPT_IMAGE` auf ein Release fest, wenn das Werkzeug es nicht tut.

## Eine Installation von vor 1.13.0

Eine Installation, die mit einer Kopie des Repositorys angelegt wurde, hat die fünf Zeilen
`KEPT_DB_*_PASSWORD` in `.env`. Mit diesem `compose.yaml` werden sie genau so verwendet, und nichts
wird erzeugt: Aktualisiere den Checkout (oder lade das neue `compose.yaml` neben die alte `.env`)
und starte `docker compose up -d`. Die Zeilen später zu entfernen oder `.env` mit `init` neu zu
schreiben, ist ebenfalls sicher: Der nächste Start erzeugt neue Passwörter und setzt sie in
Postgres. Ein leeres `KEPT_IMAGE` hat früher den Checkout gebaut; jetzt startet es das Release, das
`compose.yaml` nennt. Setz also `KEPT_IMAGE=kept:local`, wenn du selbst baust.

## Web und Worker getrennt

Ein Container liefert die Web-App aus und erledigt die Hintergrundaufgaben (`KEPT_ROLE=all`, die
Vorgabe). Um beides zu trennen, betreib einen zweiten Kept-Dienst mit `KEPT_ROLE=worker` neben
einem mit `KEPT_ROLE=web`. Ein Worker liefert kein HTTP aus; der Healthcheck des Images richtet
sich nach `KEPT_ROLE` und prüft beim Worker stattdessen dessen Aufgabenschleife, keiner von beiden
braucht also eine Anpassung (`compose.full.yaml` enthält die beiden Dienste auskommentiert). Der
Worker führt die nächtliche Sicherung aus, er ist also derjenige, der das Eigentümer-Login und bei
einem Verzeichnis als Ziel dessen Einhängung braucht.
