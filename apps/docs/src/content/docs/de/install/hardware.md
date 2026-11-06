---
title: Was es braucht
description: Worauf Kept getestet wurde, wie viel Speicher es dabei brauchte und was den Bedarf erhöht.
sidebar:
  order: 6
---

**Getestet: 1 GB RAM, ein oder zwei CPU-Kerne**, mit einem externen KI-Anbieter oder ganz ohne.
Dort lief alles durch, auch eine Sicherung während Foto-Uploads. Mit 2 GB bleibt Luft.

## Was getestet wurde

- **Version:** die Änderungen von Kept 1.14.3 (ein Build vor dem Release), `compose.yaml` wie
  veröffentlicht (Kept, Postgres 18 mit pgvector). 1.14.2 brauchte für seine Sicherung 2 GB.
- **Maschine:** in Docker auf einem arm64-Mac emuliert. Docker, Kept und Postgres teilen sich ein
  Speicher- und CPU-Budget wie auf einer kleinen VM, 128 MB sind für das Betriebssystem reserviert.
  Kein Swap.
- **Daten:** ein Haushalt, dazu ein Standort mit 3.000 Gegenständen in 206 Orten und 200 Behältern,
  und 300 echte Fotos.
- **Last:** 20 Fotos mit 12 Megapixeln, zwei PDFs, 120-mal Karte und Graph dieses Standorts laden,
  120 Listen- und Suchabrufe, `kept admin backup` während die Fotos hochladen, die geplante
  Sicherung, danach 10 Minuten Leerlauf.

| Maschine | Fotos, Karte, Suche, PDFs | Sicherung während Uploads | Geplante Sicherung | Spitze |
|---|---|---|---|---|
| 1 GB, 2 Kerne (3 Läufe) | bestanden | bestanden | bestanden | 730–745 MB |
| 1 GB, 1 Kern | bestanden | bestanden | bestanden | 740 MB |
| 768 MB, 1 Kern | bestanden, bis der Sicherung der Speicher ausging; 9 Uploads und beide PDFs scheiterten, während Kept neu startete | Speicher voll | allein bestanden (ein Lauf) | — |

Die Spitzen sind Speicher, der sich nicht zurückgeben lässt (ohne Seitencache), für die ganze
Maschine, Docker eingeschlossen. Wurde der Speicher knapp, beendete das System den Kept-Prozess,
und Docker startete ihn neu.

- **Im Ruhezustand:** nach einem Start Kept etwa 160 MB, Postgres etwa 30–70 MB. Sobald Fotos
  verkleinert wurden, bleibt Kept bei etwa 300 MB.
- **Echte Daten:** auf einem amd64-Heimserver Kept 206 MB und Postgres 119 MB (eine Messung).
- **Geschwindigkeit** (Mediane): eine Listenseite 30–40 ms, eine Suche 50–70 ms, ein Foto-Upload
  0,3–0,4 s, die Karte des Standorts mit 3.000 Gegenständen 0,4–0,5 s.
- **Noch nicht getestet:** eine echte VM, amd64 in diesen Größen, 512 MB, lokale KI. Die
  Leistungstests gehen weiter: [#379](https://github.com/keptinventory/kept/issues/379).

## Was das heißt

- **1 GB, 1 oder 2 Kerne:** getestet, jeder Lauf bestanden, mit etwa 150 MB Reserve an der Spitze.
- **2 GB:** Luft, auch für andere Dienste neben Kept.
- **768 MB:** Kept lief, ebenso eine von Hand gestartete Sicherung (derselbe Weg wie die geplante);
  `kept admin backup` während Foto-Uploads lief in den Speichermangel. Nicht empfohlen.

## Raspberry Pi

Auf einem Pi nicht getestet. Das arm64-Image ist dasselbe, mit dem diese Tests liefen. Ein Pi mit
1 GB oder mehr dürfte also wie die 1-GB-Zeilen passen; rechne damit, dass er langsamer ist. Kein
Release wird auf einem Pi getestet.

## Was Last macht

- **Sicherungen.** `kept admin backup` braucht während des Laufs etwa 300–400 MB zusätzlich, die
  geplante Sicherung etwa 150 MB; restic selbst davon etwa 100 MB. In der
  [Home-Assistant-App](/de/install/home-assistant/) entfällt das, dort laufen die Sicherungen von
  Home Assistant.
- **Fotos verkleinern.** Jeder Upload wird beim Eintreffen verkleinert. Unter 3 GB RAM oder bei
  einem Container-Limit unter 3 GB verkleinert Kept eins nach dem anderen, sonst zwei gleichzeitig
  (`KEPT_IMAGE_CONCURRENCY` legt es fest). Der Speicher steigt über die ersten Fotos um etwa 150 MB
  und bleibt dort.
- **Karte und Graph eines sehr großen Standorts.** Der Server liest bei jedem Laden den ganzen
  Standort: etwa 670 KB für 3.000 Gegenstände.
- **Nicht auf dem Server:** Karte, Graph und Raster zeichnet dein Browser, sie kosten im
  Ruhezustand keinen Serverspeicher; Karte und Graph werden erst geladen, wenn du sie öffnest.
- **Lokale KI mit Ollama.** Ein Modell auf derselben Maschine teilt sich den Speicher mit Kept und
  braucht weit mehr als Kept selbst; siehe [Ollama](/de/admin/ollama/).
- **Speicherplatz.** Die Datenbank ist klein, wachsen tun die Dateien. Lass Platz für Sicherungen
  auf einer anderen Platte. Admin → Status warnt, und Admins werden benachrichtigt, sobald eine
  Platte zu mehr als 85 % voll ist.

## Postgres

Kept braucht PostgreSQL 18 mit pgvector. Compose startet das Image `pgvector/pgvector`, per Digest
festgelegt; für die Datenbank eines Anbieters siehe [Managed Postgres](/de/install/managed-postgres/).
