---
title: Erste Schritte
description: "Der kurze Weg von null bis zum ersten Gegenstand in Kept, mit einem Link zur ausführlichen Seite für jeden Schritt."
---

Der kurze Weg, ein Schritt pro Zeile. Jeder Schritt verlinkt die Seite mit den Einzelheiten.

1. **Installieren.** Docker mit dem Compose-Plugin; getestet mit 1 GB RAM und einem CPU-Kern.
   `compose.yaml` herunterladen, dann `docker compose run --rm init` (fragt nach der Adresse, unter
   der du Kept öffnen wirst, und schreibt `.env`) und `docker compose up -d`. Keine Kopie des
   Repositorys und keine Passwörter, die du dir ausdenken musst.
   [Mit Docker Compose installieren](/de/install/compose/)
2. **Den Server übernehmen.** Die Adresse öffnen, den Einrichtungscode eingeben, den `init`
   angezeigt hat, und das erste Konto anlegen: Es wird Instanz-Admin.
   [Den Server übernehmen](/de/users/first-run/#1-den-server-mit-dem-einrichtungscode-übernehmen)
3. **Die Schlüssel aufbewahren.** Das Wiederherstellungskit ausdrucken und nicht auf dem Server
   aufbewahren. [Das Wiederherstellungskit](/de/admin/recovery-kit/)
4. **Dein erstes Zuhause erstellen.** Ein Name, die Art des Standorts, seine Räume und eine Vorlage
   dafür, was erfasst wird. [Dein erstes Zuhause erstellen](/de/users/first-run/#3-dein-erstes-zuhause-erstellen)
5. **Einen Gegenstand hinzufügen.** **Erfassen** auf dem Handy, oder **Hier hinzufügen** in einem
   Raum oder einer Kiste am Computer.
   [Den ersten Gegenstand hinzufügen](/de/users/first-run/#4-den-ersten-gegenstand-hinzufügen)
6. **Jemanden einladen.** Ein Link oder ein QR-Code mit einer Rolle (Admin, Mitglied oder
   Betrachter); niemand braucht eine E-Mail-Adresse. [Leute einladen](/de/users/first-run/#5-leute-einladen)
7. **Auf HTTPS umstellen**, damit Handys die Kamera, das Installieren von Kept, die Offline-Nutzung
   und Push-Benachrichtigungen erlauben. [HTTPS](/de/install/https/), oder
   [Kept im Heimnetz](/de/install/home-network/) für das, was ohne HTTPS geht
8. **KI verbinden**, wenn du möchtest: Kept funktioniert auch ohne.
   [KI-Anbieter](/de/users/ai-providers/)

Danach: [Alltag mit Kept](/de/users/everyday-use/) für den Rundgang, die
[Fragen und Antworten](/de/users/faq/) für häufige Fragen, und
[Häufige Probleme](/de/users/troubleshooting/), wenn etwas nicht funktioniert.
