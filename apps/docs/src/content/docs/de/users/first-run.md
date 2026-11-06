---
title: Deine ersten Minuten
description: Eine neue Kept-Instanz mit dem Einrichtungscode übernehmen, das erste Zuhause erstellen, Gegenstände hinzufügen, Leute einladen und Sprache und Design wählen.
---

Diese Seite beginnt, sobald Kept läuft und du seine Adresse öffnen kannst. Die Installation steht
unter [Mit Docker Compose installieren](/de/install/compose/).

## 1. Den Server mit dem Einrichtungscode übernehmen

Startet Kept zum ersten Mal ohne Admin, schreibt es einen einmaligen Einrichtungscode in sein Log,
etwa `KEPT SETUP CODE: K7Q-2M9`. Die Adresse von Kept führt dich zu **Diese Kept-Instanz
einrichten**, drei Schritte:

1. **Einrichtungscode.** Gib die sechs Zeichen ein. Der Bindestrich ist optional, Groß- und
   Kleinschreibung egal. Nach der Verwendung ist der Code ungültig. Log verloren?
   `kept admin setup-code` stellt einen neuen aus
   ([so geht es unter Compose](/de/install/compose/#3-einrichtung-abschließen)).
2. **Erstes Konto.** Dein Name, deine E-Mail-Adresse und ein Passwort mit mindestens 8 Zeichen.
   Dieses Konto ist der **Instanz-Admin**: Er betreibt den Server (Benutzer, Registrierung,
   Sicherungen). In das Zuhause anderer sieht er nicht, solange sie ihn nicht einladen.
3. **Instanzoptionen.** Ein Schalter: **Jedem mit der Adresse erlauben, ein Konto zu erstellen**.
   Für einen Haushalt bleibt er aus: Leute kommen über eine Einladung dazu, oder du legst sie an.
   Du kannst das später unter Einstellungen → Admin ändern.

**Einrichtung abschließen** bringt dich zur Startseite.

:::caution[Sofort abschließen]
Solange die Einrichtung nicht abgeschlossen ist, kann jeder, der die Adresse öffnen kann,
versuchen, den Server zu übernehmen.
:::

KI und Barcode-Suche gehören nicht zur Einrichtung. Kept funktioniert ohne sie, und KI kannst du
später verbinden ([KI-Anbieter](/de/users/ai-providers/)).

## 2. Die Checkliste „Erste Schritte“ auf der Startseite

Die Startseite beginnt mit **Erste Schritte**: eine Zeile mit deinem Fortschritt und dem nächsten
Schritt. Kept ermittelt sie aus dem, was du wirklich getan hast, also haken sich die Schritte von
selbst ab. Der Pfeil klappt die Schritte auf:

| Schritt | Was er verlangt |
|---|---|
| **Kept auf HTTPS umstellen** | Erscheint beim Instanz-Admin, solange Kept über einfaches HTTP läuft. Handys sperren über HTTP die Kamera, das Installieren von Kept, Push-Benachrichtigungen und den Standort. Siehe [HTTPS](/de/install/https/). |
| **Dein erstes Zuhause erstellen** | Ein Standort, siehe unten. |
| **3 Gegenstände hinzufügen** | Ein paar, die du ungern aus den Augen verlieren würdest. |
| **Drucke dein erstes Etikett** | Wenn in einem Standort Etiketten eingeschaltet sind. |
| **Jemanden einladen** | Ein Link oder ein QR-Code; niemand braucht eine E-Mail-Adresse. |
| **KI-Anbieter verbinden** | Erscheint nicht bei eingeladenen Mitgliedern und nicht in einem Standort mit der Vorlage Basis. |
| **Auf dem Handy installieren** | Hakt sich ab, wenn du Kept zum ersten Mal über sein Symbol auf dem Startbildschirm öffnest. |

**×** blendet die Checkliste dauerhaft aus; **Hilfe → Checkliste „Erste Schritte“ → Wieder
anzeigen** holt sie zurück. Sind alle Schritte erledigt, verschwindet sie von selbst. Mehr zur
Startseite unter [Alltag mit Kept](/de/users/everyday-use/#startseite).

## 3. Dein erstes Zuhause erstellen

Jeder hat von Anfang an einen Standort **Persönlich**: Er gehört nur dir (keine Mitglieder,
Einladungen oder Freigabelinks), und dort landet alles, was du ohne Standort erfasst. Umbenennen
kannst du ihn unter **Standorteinstellungen → Allgemein**, dann zeigt Kept überall den neuen
Namen. Sobald du einen weiteren Standort hast, kannst du ihn unter **Einstellungen → Profil → Deine
Standorte** [ausblenden](/de/users/locations/#persönlich-ausblenden). Was ihr gemeinsam nutzt, gehört in
einen eigenen Standort. Wähle **Dein erstes Zuhause erstellen** (oder **Neuer Standort** auf der
Startseite):

1. **Wie heißt dieser Standort?** Ein Name, etwa Zuhause oder Garage, und seine Art: Haus, Wohnung,
   Garage, Lagerabteil, Büro, Ferienhaus oder Sonstiges.
2. **Räume.** Die Vorlage der Art füllt Räume und Plätze vor; umbenennen, entfernen oder **Raum
   oder Platz hinzufügen**. Jeder Standort hat außerdem **Ohne Ort**, für Gegenstände, die noch
   keinen Platz haben. **Überspringen** behält nur Ohne Ort.
3. **Was erfasst wird.** Drei Vorlagen:
   - **Basis**: Dinge finden: Räume, Kisten, Etiketten und Suche.
   - **Haushalt**: dazu Belege und Garantien, Erinnerungen, Verleihen, Unterlagen und Fahrzeuge.
   - **Vollständig**: dazu Tanken, Was zur Neige geht, Passwörter und Codes, Umzug und „ChatGPT
     oder Claude verbinden“.

Zeitzone und Währung kommen aus deinem Browser. Ändern, und einzelne Module ein- oder ausschalten,
kannst du in den **Standorteinstellungen** des Standorts (**Allgemein** und **Was erfasst wird**).
Ein ausgeschaltetes Modul wird nur ausgeblendet; gelöscht wird nichts.

## 4. Den ersten Gegenstand hinzufügen

Am schnellsten geht es mit **Erfassen**, dem mittleren Reiter auf dem Handy: Kamera ausrichten,
auslösen, und der Gegenstand ist dort gespeichert, wo der Ort oben steht. Einen Namen brauchst du
beim Erfassen nicht: Mit einem verbundenen KI-Anbieter benennt Kept ihn anhand des Fotos; ohne
warten unbenannte Gegenstände im **Eingang** auf einen Namen. Am Computer fügt **Hier
hinzufügen** in einem Raum oder einer Kiste einen Gegenstand über ein Formular hinzu.
[Alltag mit Kept](/de/users/everyday-use/) behandelt Erfassen, Etiketten und den Rest.

## 5. Leute einladen

Geteilt wird pro Standort: Wen du zu Zuhause einlädst, der sieht Zuhause und sonst nichts von dir.
Nachdem du einen Standort erstellt hast, zeigt die Startseite eine Karte **Leute zu … einladen**;
dieselbe Seite liegt unter **Standorteinstellungen → Mitglieder → Einladen**.

1. Wähle die **Rolle** und unter **Zugriff** **Kein Enddatum** oder **Bis zu einem Datum**. Ihr
   Zugriff, und alle Tokens, die sie anlegen, enden an diesem Datum.
2. **Einladungslink erstellen.** Du bekommst den Link mit **Link kopieren** und einen QR-Code, um
   ihn **Mit ihrem Telefon scannen** zu lassen. Der Link funktioniert einmal, 7 Tage lang, und
   steht bis zur Verwendung als offene Einladung unter Mitglieder, wo du ihn zurückziehen kannst.
3. **Per E-Mail senden** funktioniert nur, wenn auf dem Server E-Mail eingerichtet ist
   (`KEPT_SMTP_URL`).

Die eingeladene Person öffnet den Link und tritt mit ihrem Konto bei oder legt eines an; solange
die Registrierung geschlossen ist, ist die Einladung der einzige Weg zu einem Konto.

### Die Rollen

| Rolle | Was sie in diesem Standort darf |
|---|---|
| **Eigentümer** | Alles, auch Zwei-Faktor-Regeln, Übertragen und Löschen. Es gibt einen Eigentümer. |
| **Admin** | Mitglieder und Betrachter einladen und entfernen, Einstellungen und Module, Import und Export, Freigabelinks und Schadenspakete, endgültig löschen. Nur der Eigentümer ernennt oder entfernt Admins. |
| **Mitglied** | Gegenstände hinzufügen, bearbeiten, verschieben, verleihen und in den Papierkorb legen; Ablesungen und Wartungen eintragen; Etiketten drucken; mit KI erfassen. Geldbeträge sind sichtbar. |
| **Betrachter** | Nur ansehen und suchen. Geldbeträge sind verborgen, wenn der Standort sie Betrachtern nicht erlaubt; der Assistent antwortet, kann aber nichts ändern. |

Die vollständige Übersicht ist `MATRIX` in
[`packages/shared/src/roles.ts`](https://github.com/keptinventory/kept/blob/main/packages/shared/src/roles.ts).

### Personen ohne E-Mail

Für ein Kind oder ein Elternteil ohne E-Mail nutzt ein Eigentümer oder Admin auf der Seite
Mitglieder **Person ohne E-Mail hinzufügen**. Kept legt ein **verwaltetes Konto** mit einem
Benutzernamen an und zeigt einen Einmalcode (er funktioniert einmal, 30 Minuten lang). Die Person
meldet sich mit **Ich habe einen Einmalcode**, ihrem Benutzernamen und dem Code an und wählt ihr
eigenes Passwort; du siehst es nie. Wer das Konto angelegt hat (solange er dort noch Eigentümer
oder Admin ist), oder der Eigentümer dieses Standorts, kann später einen neuen Code ausstellen; das
meldet das Konto überall ab. Die Person kann eine E-Mail-Adresse hinzufügen und das Konto damit
selbst übernehmen. Ein verwaltetes Konto kann keinen Standort besitzen, und ein Standort hat
höchstens 20 davon.

## 6. Sprache und Design

**Einstellungen → Profil → Anzeige** enthält die Anzeigeeinstellungen des Geräts. Sie gelten,
bevor die Seite gezeichnet wird:

- **Sprache**: English, العربية, Français, Deutsch, Italiano, Tiếng Việt, Українська oder Español.
  Klapp die Liste auf oder tipp, um eine zu finden: ihren eigenen Namen, ihren englischen Namen oder ihren
  Namen in der Sprache, die du gerade nutzt; kleine Tippfehler sind kein Problem (`german`,
  `deutsch` und `francais` funktionieren alle). Arabisch stellt die ganze App von rechts nach links
  dar. Die Anmeldeseiten haben dieselbe Auswahl.
- **Ziffern auf Arabisch**: westlich (0123) oder östlich (٠١٢٣), nur sichtbar, solange die Sprache
  Arabisch ist. Kurz-IDs und Seriennummern bleiben immer in westlichen Ziffern.
- **Region**: **United States** (Oct 29) oder **United Kingdom** (29 Oct, 29/10/2026), nur sichtbar,
  solange die Sprache Englisch ist.
- **Datumsformat**: **Standard** (29. Okt., mit Jahr nur bei anderen Jahren), 29. Okt. 2026,
  29.10.2026 oder 2026-10-29. Jede Auswahl zeigt das heutige Datum in dieser Schreibweise, in der
  Schreibweise deiner Sprache und Region. Kauf- und Garantiedaten zeigen immer das Jahr.
- **Design**: **System** (folgt der hellen oder dunklen Einstellung des Geräts), **Hell** oder
  **Dunkel**.
- **Inhaltsbreite**: **Zentriert** oder **Volle Breite**, auf Bildschirmen ab 768 px Breite.

## 7. Die Tour

**Hilfe → Die Tour** führt dich auf den echten Bildschirmen durch Kept, in kurzen Kapiteln mit
jeweils drei bis sechs Schritten:

| Kapitel | Was es zeigt |
|---|---|
| Grundlagen | Die Startseite, deine Standorte, Scannen, Suche, und wo alles andere liegt |
| Gegenstände & Orte | Etwas zu einem Standort hinzufügen, was darin ist, und die Seite eines Gegenstands mit ihren Reitern |
| Graph & 3D-Karte | Der Umschalter Liste · Raster · Graph · Tabelle · Karte (Tabelle am Computer), der Graph in 2D oder 3D, 3D oder Grundriss, Markierungen, Etagen und Anordnen, wobei ein Raum mit seinem Inhalt umzieht |
| Erfassen & Eingang | Auswählen, was du erfasst, der Auslöser, und prüfen, was Kept erkannt hat |
| Scannen | Eine Scan-Sitzung: zählen, packen oder verschieben, wo, und was Fertig ändert |
| Suche & Etiketten | Das Suchfeld, gespeicherte Ansichten, Auswählen (verschieben, Etiketten drucken, In den Papierkorb), Etiketten drucken und Scannen, das ein leeres Etikett einem vorhandenen Ort zuordnen kann |
| Umzug | Einen Umzug beginnen, seine Kisten, und wo sie gerade sind |
| Verleihen | Was verliehen ist, Als zurückgegeben markieren, und Verleihen an einem Gegenstand |
| Zeitpläne & Erinnerungen | Ein neuer Zeitplan, was fällig ist, Erledigen, und wie Erinnerungen dich erreichen |
| Unterlagen & Garantien | Die Unterlagen-Bibliothek, was abläuft, eine Datei hinzufügen, Schadensfälle |
| Fahrzeuge & Tanken | Die Fahrzeugliste, eine Ablesung oder Wartung eintragen, die Wartungen, die du öffnen kannst, und Tanken |
| Import & Export | Woher ein Import kommt, die Datei, und Exportieren |
| Einstellungen & Teilen | Benachrichtigungen, deine Standorte, Leute einladen, was erfasst wird |

Aufgeführt sind nur die Kapitel, die du hast: Das Kapitel eines Moduls erscheint, solange das Modul
in einem deiner Standorte eingeschaltet ist, und ein Betrachter bekommt keine Schritte, die etwas
ändern.

- Oben: wie viele Kapitel erledigt sind, mit einem dünnen Balken, und **Tour starten**. Sind schon
  welche erledigt, heißt es **Tour fortsetzen** und setzt beim ersten Kapitel an, das du noch nicht
  abgeschlossen hast.
- Darunter eine kurze Zeile pro Kapitel: seine Nummer und seine Schritte. Tipp auf eine Zeile, um
  die Tour bei diesem Kapitel zu beginnen. Erledigte Kapitel klappen unter **Erledigt (N)** weg;
  klapp es auf, um eines noch einmal anzusehen.
- Die Tour startet über die Hilfe. Die obere Leiste einer Seite zeigt nur die Funktionen dieser
  Seite; **Sitzung starten** in einer Scan-Sitzung hat ein kleines **Zeig es mir** für ihr Kapitel.
- Das Popover zeigt, wo du bist: „Kapitel 3 von 12 · Schritt 2 von 5“.
- **Weiter** geht vor, **Zurück** innerhalb des Kapitels zurück, die Pfeiltasten tun beides, und
  **Esc** oder **×** schließt die Tour.
- Der letzte Schritt eines Kapitels nennt das nächste: **Weiter** geht direkt hinein, **Hier
  aufhören** beendet die Tour an dieser Stelle. Das letzte Kapitel endet mit **Fertig**.
- Die Seite funktioniert weiter, während die Tour läuft: Tipp auf alles außerhalb der Hervorhebung.
  Gehst du an eine Stelle, die die Tour nicht geöffnet hat, schließt sie sich.
- Ein abgeschlossenes Kapitel steht in der Hilfe als **Fertig**, auf jedem Gerät. Ein Kapitel, das
  du mittendrin schließt, beginnt beim nächsten Mal mit seinem ersten Schritt.
- Braucht ein Kapitel etwas zum Zeigen und findet nichts, legt es ein deutlich benanntes Beispiel
  an, etwa „Beispiel: Akkuschrauber“. Am Ende der Tour entscheidest du, ob du es behältst oder
  verwirfst; was du schon hast, rührt sie nie an.
- Die Tour startet nie von selbst.

Ein paar einmalige Hinweise erscheinen außerdem, wenn du Erfassen, Scannen, die vorgeschlagenen
Werte im Eingang oder Etiketten zum ersten Mal benutzt.

Nach einem Update von Kept zeigt **Neu in Kept** die Höhepunkte dieser Version, einmal pro Version
und Person, auf jedem Gerät: einer pro Folie, mit einem Bild oder einem kurzen Clip, einem Titel
und einer Zeile. **Weiter** und **Zurück**, die Punkte, die Pfeiltasten oder ein Wischen wechseln
zwischen ihnen; **Verstanden** auf der letzten Folie schließt sie, und **Hilfe → Was ist neu** zeigt
sie wieder. Clips laufen stumm und lassen sich anhalten; mit reduzierter Bewegung bleiben sie
stehen, bis du **Abspielen** drückst. Offline zeigt eine Folie statt ihres Bildes das Symbol ihres
Bereichs. Bei einem neuen Konto erscheint sie nie, und auch nicht über Erfassen oder Scannen. Die
Höhepunkte sind in jeder Sprache englisch, mit einem Link zu den vollständigen Versionshinweisen
auf GitHub.

:::tip[Die Schlüssel aufbewahren]
Vor dem ersten KI-Schlüssel, geheimen Wert oder der ersten Sicherung wartet Kept, bis der
Instanz-Admin das [Wiederherstellungskit](/de/admin/recovery-kit/) gesichert hat (Admin → Status
fragt danach). Ohne das Kit kann eine Sicherung auf einem neuen Server keine KI-Schlüssel und
keine gespeicherten Passwörter zurückbringen.
:::
