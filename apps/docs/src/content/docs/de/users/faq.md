---
title: Fragen und Antworten
description: Kurze Antworten auf die Fragen, die vor und nach der Installation von Kept gestellt werden, jeweils mit einem Link zur Seite mit den Einzelheiten.
---

## Schickt Kept irgendetwas über mich oder meinen Server irgendwohin?

Nein. Es gibt keinerlei Telemetrie, nicht einmal eine freiwillige Zählung der Installationen. Die
Update-Prüfung ist aus, bis ein Admin sie einschaltet, und schickt dann nur die Anfrage nach der
neuesten Version. Barcode-Suche und KI sind ebenfalls aus, bis jemand sie verbindet. Siehe
[Beobachtbarkeit](/de/admin/observability/) und [Updates](/de/admin/upgrades/).

## Wo liegen meine Daten?

Auf deinem Server: die Datenbank in Postgres, die Dateien (Fotos, Belege, Dokumente) im
Daten-Volume von Kept oder in einem S3-Bucket deiner Wahl. Dein Handy hält eine Kopie für die
Offline-Nutzung, ohne geheime Werte. Siehe [Dateiablage](/de/admin/storage/) und
[die Handy-App](/de/users/phone-app/).

## Wie lösche ich mein Konto?

Einstellungen → Profil → Mein Konto löschen, mit deinem Passwort. Gib vorher jeden Standort, der dir
gehört (außer Persönlich), an jemand anderen ab oder lösche ihn; ist das Konto der einzige Admin der
Instanz, muss es zuerst einen weiteren Admin ernennen. Danach wartet das Konto 30 Tage: Bei der
Anmeldung siehst du das Datum und einen Weg, das Konto zu behalten, und sonst öffnet sich nichts.
Danach wird es zusammen mit deinem Standort Persönlich gelöscht. Was du in Standorten anderer
hinzugefügt hast, bleibt dort. [Exportiere deine Daten](/de/users/import-export/#exportieren) vorher,
wenn du eine Kopie willst.

## Kann ich Kept ohne KI nutzen?

Ja. Alles funktioniert, außer Gegenstände anhand von Fotos zu benennen, Belege zu lesen, dem
Assistenten und der Suche nach Bedeutung. Ohne Anbieter warten erfasste Gegenstände im Eingang auf
einen Namen, und die Suche arbeitet mit Stichwörtern. Siehe [KI-Anbieter](/de/users/ai-providers/).

## Welche KI-Anbieter funktionieren, und wer zahlt?

Groq (empfohlen), OpenAI, Anthropic, Google, OpenRouter, oder jeder OpenAI-kompatible Server wie
Ollama. Du nutzt deinen eigenen Schlüssel; Arbeit in einem Standort läuft über den Schlüssel des
Eigentümers dieses Standorts, und du kannst die Ausgaben deckeln. Siehe
[KI-Anbieter](/de/users/ai-providers/).

## Funktioniert es offline?

Auf einem Handy mit installiertem Kept ja, für Finden und Erfassen: stöbern, suchen, erfassen,
Gegenstände verschieben, Ablesungen, Kistenkontrollen und Etiketten scannen. Details bearbeiten,
Berichte, Importe und Exporte brauchen eine Verbindung. Was du offline tust, wird abgeglichen,
sobald du wieder online bist. Siehe [Kept auf dem Handy](/de/users/phone-app/#was-offline-funktioniert).

## Gibt es eine App im App Store oder bei Google Play?

Nein. Kept ist eine Web-App, die du aus dem Browser zum Startbildschirm hinzufügst. Dafür muss Kept
über HTTPS laufen. Siehe [Kept auf dem Handy](/de/users/phone-app/#installieren) und
[HTTPS](/de/install/https/).

## Läuft es auf arm64 oder einem Raspberry Pi?

Die Images werden für amd64 und arm64 gebaut, also laufen sie auf ARM-Servern und mit Docker auf
Apple Silicon. Ein Raspberry Pi mit 1 GB oder mehr könnte reichen, langsamer; keine Version wird auf
einem getestet. Getestet ist Kept mit 1 GB RAM und ein oder zwei CPU-Kernen (arm64). Siehe
[was getestet wurde](/de/install/hardware/).

## Wie viele Leute können ein Kept nutzen?

Kept ist nach Gegenständen ausgelegt, nicht nach Personen: Ziel sind 10.000 Gegenstände in einem
Standort und 50.000 in einer Instanz; der Test mit 1 GB nutzte 3.000 in einem Standort. Leute
kommen per Einladung zu einem Standort, und jeder Standort kann außerdem bis zu 20 verwaltete Konten
für Menschen ohne E-Mail haben. Siehe [was es braucht](/de/install/hardware/) und
[Leute einladen](/de/users/first-run/#5-leute-einladen).

## Kann ich mehr als einen Haushalt führen, oder nur einen Teil meiner Sachen teilen?

Ja. Jeder Standort (ein Zuhause, eine Garage, ein Lagerabteil, das Haus der Eltern) wird für sich
geteilt, mit eigenen Mitgliedern und Rollen; wer die Garage teilt, teilt sonst nichts. Eine Person
kann zu vielen Standorten gehören, und jeder hat einen privaten Standort Persönlich. Siehe
[deine ersten Minuten](/de/users/first-run/#5-leute-einladen).

## Was unterscheidet Kept von Homebox?

Beide halten fest, was du besitzt und wo es ist. Kept hat außerdem:

- **Einen 3D-Raumplan** deiner Etagen, Räume, Regale und Kisten.
- **Fahrzeuge**: Ablesungen, Wartungen, Tanken und die Kosten dafür.
- **Erinnerungen** für Garantien, Wartungen, Leihgaben und knappe Vorräte.
- **Offline-Nutzung** auf dem Handy, mit Etiketten, die du druckst und scannst.
- **KI, wenn du willst**: Erfassen aus einem Foto, und einen MCP-Server für Assistenten.

Es importiert eine Homebox-Sicherung, du kannst es also mit deinen eigenen Sachen ausprobieren.

## Kann ich von Homebox umziehen?

Ja. Ab Homebox v0.26.1 (empfohlen v0.27.0) legst du in Homebox eine Sicherung an (**Collection →
Tools → Backup & Restore → Create Backup**, dann **Download**), importierst diese ZIP-Datei in
Kept, prüfst, was passieren wird, und importierst dann. Fotos, Belege, Garantien, Wartungen und
Schlagwörter kommen mit, und alte Homebox-Etiketten lassen sich weiter scannen. Ein Gegenstand kommt
nur dann als verkauft an, wenn Homebox einen Käufer oder einen Preis dafür hat (ein Verkaufsdatum
allein bleibt in seinen Notizen), und archivierte Gegenstände kommen als **Archiviert** an, es sei
denn, du wählst **In Gebrauch** oder **Weglassen**. Verbinde zuerst
[Paperless-ngx](/de/users/paperless/), dann kommen die Links von Homebox auf seine Dokumente als
Paperless-Links an. Siehe [Import und Export](/de/users/import-export/#ein-homebox-export); ein
Import von vor 1.14.1 lässt sich [reparieren](/de/users/import-export/#einen-früheren-import-reparieren).
Ein älteres Homebox hat keine Sicherung, die Kept lesen kann: Aktualisiere zuerst eine Kopie davon
oder importiere seine Tabelle. Siehe
[von einem älteren Homebox kommen](/de/users/import-export/#von-einem-älteren-homebox-kommen).

## Wie sichere ich es?

Der Server legt jede Nacht einen verschlüsselten restic-Snapshot der Datenbank, der Dateien und
einer lesbaren Kopie jedes Standorts an, auf einer anderen Festplatte, in einem S3-Bucket oder auf
einem SFTP-Server. Bewahre das Wiederherstellungskit woanders auf. Um einen Standort mitzunehmen,
exportiere ihn. Siehe [Sicherungen](/de/admin/backups/),
[das Wiederherstellungskit](/de/admin/recovery-kit/) und
[Import und Export](/de/users/import-export/#exportieren).

## Und wenn der Server mit dem Haus verloren geht?

Die lesbare Kopie aus der Sicherung öffnet sich in jedem Browser, mit restic, dem Passwort der
Sicherung und einem Laptop, ganz ohne Kept. Um Kept selbst auf einem neuen Rechner zurückzuholen,
brauchst du zusätzlich das Wiederherstellungskit. Siehe
[das Inventar nur mit restic lesen](/de/admin/read-with-restic/) und
[Wiederherstellen](/de/admin/restore/).

## Welche Sprachen spricht es?

Englisch, Arabisch (von rechts nach links), Französisch, Deutsch, Italienisch, Vietnamesisch,
Ukrainisch und Spanisch, gewählt pro Gerät. Siehe [Sprache und Design](/de/users/first-run/#6-sprache-und-design).

## Unter welcher Lizenz steht es?

Unter der GNU Affero General Public License, Version 3 (AGPL-3.0). Du darfst es betreiben, ändern und
weitergeben; betreibst du eine geänderte Fassung für andere über ein Netzwerk, musst du ihnen den
Quelltext anbieten. Der Text liegt im Repository in
[LICENSE](https://github.com/keptinventory/kept/blob/main/LICENSE).

## Ist Kept kostenlos? Wird es kostenpflichtig?

Selbst gehostetes Kept ist kostenlos und Open Source. Apps für Handy und Desktop sowie eine
Cloud-Version sind in Arbeit.

## Gibt es eine Version?

Ja: `1.0.0` war die erste signierte Version, auf GHCR als `ghcr.io/keptinventory/kept`, und jede
Version seitdem steht auf der [Release-Seite](https://github.com/keptinventory/kept/releases).
Updates führen ihre Migrationen einmal aus, legen vorher einen Snapshot der Datenbank an und können
eine Version zurückgehen. Siehe [Mit Docker Compose installieren](/de/install/compose/) und
[Updates](/de/admin/upgrades/).
