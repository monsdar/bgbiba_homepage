# 🏀 BG BiBA – Handbuch für die Vereins-Homepage

Willkommen bei der Verwaltung der offiziellen Website der **BG BiBA** ([bgbiba.de](https://bgbiba.de/)).

Diese Anleitung ist speziell für **Vereinsmitglieder, Trainerinnen, Trainer und Verantwortliche** geschrieben. Du brauchst **keine Programmierkenntnisse** und musst nichts auf deinem Computer installieren – alle alltäglichen Anpassungen kannst du direkt hier im Internetbrowser auf GitHub erledigen.

---

## 📖 Inhaltsverzeichnis

1. [Grundlagen: Wie bearbeite ich etwas auf GitHub?](#-grundlagen-wie-bearbeite-ich-etwas-auf-github)
2. [Usecase 1: Team umkonfigurieren (Warteliste, Jahrgänge, E-Mail)](#-1-team-umkonfigurieren-warteliste-an-abschalten)
3. [Usecase 2: Neues Team hinzufügen oder Team pausieren](#-2-neues-team-hinzufügen-oder-pausieren)
4. [Usecase 3: Wiki-Inhalte (Trainingszeiten & Infos) verlinken](#-3-wiki-inhalte-verlinken-trainingszeiten--infos)
5. [Usecase 4: Neue Fotos für die Startseiten-Slideshow hinzufügen](#-4-neue-fotos-für-die-slideshow-hinzufügen)
6. [Usecase 5: Neue Vereinslogos für Gegner (TeamSL) hinzufügen](#-5-neue-logos-für-teamsl-gegner-hinzufügen)
7. [Usecase 6: Saisonwechsel – TeamSL Liga- und Team-IDs anpassen](#-6-saisonwechsel-teamsl-ids-anpassen)
8. [Gut zu wissen: Automatische Updates, Sofort-Update & weitere Seiten](#-gut-zu-wissen-weitere-hilfreiche-tipps)

---

## 💡 Grundlagen: Wie bearbeite ich etwas auf GitHub?

GitHub funktioniert im Grunde wie eine Datei-Ablage (ähnlich wie Google Drive oder OneDrive). Sobald du hier eine Datei änderst und speicherst, wird die Webseite im Hintergrund vollautomatisch neu gebaut. **Nach etwa 1 bis 2 Minuten** ist deine Änderung live auf `bgbiba.de` sichtbar.

### Eine bestehende Textdatei ändern
1. Klicke dich durch die Ordner zur gewünschten Datei (z. B. `content` → `teams` → `herren.md`).
2. Klicke oben rechts über dem Dateitext auf das **Stift-Symbol (✏️ „Edit this file")**.
3. Nimm deine Textänderung vor.
4. Klicke oben rechts auf den grünen Button **„Commit changes..."** (das bedeutet bei GitHub schlicht „Speichern").
5. Es öffnet sich ein kleines Fenster: Klicke dort einfach noch einmal auf den grünen Button **„Commit changes"**. Fertig!

### Eine neue Datei erstellen oder Bilder hochladen
1. Öffne den Ordner, in den die Datei soll (z. B. `slideshow` oder `content/teams`).
2. Klicke oben rechts auf den Button **„Add file"**:
   * Wähle **„Create new file"**, um eine neue Textseite (z. B. ein neues Team) anzulegen.
   * Wähle **„Upload files"**, um Bilder oder Logos von deinem Computer/Handy hochzuladen.
3. Klicke anschließend auf den grünen Button **„Commit changes"**, um das Hochladen abzuschließen.

### Eine Datei löschen (z. B. ein altes Slideshow-Foto)
1. Klicke auf die Datei, die du entfernen möchtest.
2. Klicke oben rechts auf das **Drei-Punkte-Menü (`...`)** und wähle **„Delete file"**.
3. Bestätige wieder mit dem grünen Button **„Commit changes..."**.

---

## 🚦 1. Team umkonfigurieren (Warteliste an-/abschalten)

Alle Mannschaften liegen als eigene Datei im Ordner **`content/teams/`** (zum Beispiel `content/teams/mu12.md` oder `content/teams/u10-mixed.md`).

Wenn du eine solche Datei öffnest, siehst du ganz oben einen Block zwischen zwei gestrichelten Linien (`---`). Dort werden die Grundeinstellungen des Teams gesteuert:

```yaml
---
title: "Männliche U12 (mU12)"
description: "Jahrgang 2015/2016 – Zwei Teams im Ligaspielbetrieb..."
category: "Minis"
icon: "♂️"
email: "mu12@bgbiba.net"
weight: 60
is_open_for_new_players: false
---
```

### Warteliste einschalten (Aufnahmestopp / Halle voll)
Wenn ein Team aktuell keine neuen Spielerinnen oder Spieler zum Probetraining aufnehmen kann, füge zwischen den oberen `---` Linien diese Zeile ein (oder setze sie auf `false`):
```yaml
is_open_for_new_players: false
```
**Was passiert dadurch automatisch?**
* Auf der Teamseite ändert sich der Hinweisbanner und das Kontaktformular automatisch von *„Probetraining vereinbaren"* auf **„Aufnahme auf die Warteliste"**.
* Anfragen über das Formular erhalten automatisch den E-Mail-Betreff `Anfrage / Warteliste – <Teamname>`.

### Warteliste wieder ausschalten (Plätze frei)
Sobald wieder Platz für Probetrainings ist, ändere die Zeile einfach auf:
```yaml
is_open_for_new_players: true
```
*(Alternativ kannst du die Zeile `is_open_for_new_players: false` auch komplett löschen – standardmäßig ist jedes Team offen für neue Spieler.)*

### Jahrgänge, Beschreibung oder E-Mail-Adresse ändern
In derselben Datei kannst du jederzeit:
* bei `email: "..."` die Empfänger-Adresse für das Kontaktformular ändern,
* bei `description: "..."` den kurzen Vorschautext für die Team-Übersicht anpassen,
* weiter unten im Text die **Jahrgänge** (z. B. `* **Jahrgänge:** 2013 / 2014`) zum Saisonwechsel aktualisieren.

---

## ➕ 2. Neues Team hinzufügen (oder pausieren)

### Neues Team anlegen
1. Gehe in den Ordner **`content/teams/`**.
2. Klicke oben rechts auf **„Add file" → „Create new file"**.
3. Gib oben als Dateinamen einen kurzen Namen in **Kleinbuchstaben ohne Leerzeichen und ohne Umlaute** ein, der auf **`.md`** endet (z. B. `wu10.md` oder `herren-2.md`).
4. Kopiere dir folgende Vorlage in das große Textfeld und passe sie an:

```markdown
---
title: "Weibliche U10 (wU10)"
description: "Jahrgang 2017/2018 – Spielerischer Basketball-Einstieg für Mädchen bis 10 Jahre."
category: "Minis"
icon: "♀️"
email: "wu10@bgbiba.net"
weight: 15
is_open_for_new_players: true
---

## Über das Team

* **Jahrgänge:** 2017 / 2018
* **Spielklasse:** Regionsliga U10

Hier steht ein kurzer Beschreibungstext über die Mannschaft, das Training und an wen sich das Team richtet.

---

## Trainingszeiten & Infos

{{< bookstack id="7" >}}
* **Kontakt:** [wu10@bgbiba.net](mailto:wu10@bgbiba.net)
{{< /bookstack >}}

---

## Spiele & Ergebnisse

{{< teamsl-matches league_id="54188" team_id="448537" limit="6" >}}

---

## Tabelle

{{< teamsl-standings league_id="54188" >}}
```

> **Tipp:** Nimmt ein Team (noch) nicht am Ligaspielbetrieb teil (wie z. B. die Daddeltruppe oder ein Team im Neuaufbau), lässt du die unteren Abschnitte `Spiele & Ergebnisse` sowie `Tabelle` einfach weg!

### Bedeutung der Felder im oberen Block (`---`)
* **`title`**: Der angezeigte Name der Mannschaft.
* **`description`**: Kurzer Satz, der auf der Übersichtsseite „Unsere Teams" auf der Karte steht.
* **`category`**: Das kleine Schildchen oben auf der Teamkarte (üblich sind `"Minis"`, `"Jugend"`, `"Senioren"` oder `"Freizeit"`).
* **`icon`**: Das Symbol vor dem Teamnamen (`"♀️"` für weiblich, `"♂️"` für männlich, `"👫"` für Mixed/Freizeit, `"🏀"` allgemein).
* **`email`**: An diese Adresse werden alle Nachrichten aus dem Kontaktformular der Teamseite geschickt.
* **`weight`**: Bestimmt die **Reihenfolge** auf der Team-Übersichtsseite. Kleinere Zahlen stehen weiter oben/links (z. B. `10` = U10, `60` = mU12, `95` = Herren, `130` = Daddeltruppe).

### Ein Team vorübergehend ausblenden (pausieren)
Wenn eine Mannschaft in einer Saison mal nicht zustande kommt, musst du die Datei nicht löschen. Füge im oberen Block zwischen den `---` Linien einfach diese Zeile hinzu:
```yaml
draft: true
```
Damit wird das Team auf der Webseite komplett ausgeblendet, bleibt aber als Vorlage für die nächste Saison erhalten.

---

## 📖 3. Wiki-Inhalte verlinken (Trainingszeiten & Infos)

Damit Trainingszeiten, Hallen und Trainer-Kontakte nicht doppelt gepflegt werden müssen, zieht sich die Homepage diese Infos direkt aus unserem Vereins-Wiki ([wiki.bgbiba.de](https://wiki.bgbiba.de)).

Auf der Homepage wird dafür dieser kurze Baustein verwendet:
```markdown
{{< bookstack id="83" >}}
* **Kontakt:** [herren@bgbiba.net](mailto:herren@bgbiba.net)
{{< /bookstack >}}
```

### Wie finde ich heraus, welche `id` eine Wiki-Seite hat?
1. Öffne die gewünschte Seite im Vereins-Wiki (`https://wiki.bgbiba.de`).
2. **Markiere mit der Maus irgendein Wort** oder einen kurzen Textabschnitt auf der Seite.
3. Es ploppt ein kleines Menü über dem Text auf: Klicke dort auf **„Auswahl teilen"** (bzw. *„Selection teilen"* / Link-Symbol).
4. In dem kleinen Dialogfenster wird dir angezeigt, welche **Seiten-ID** (Zahl) die aktuelle Wiki-Seite hat (z. B. `83` oder `page_id`).
5. Trage diese Zahl bei `{{< bookstack id="DEINE_ZAHL" >}}` auf der Teamseite ein.

> **Wofür ist der Text zwischen den beiden `bookstack`-Zeilen?**  
> Das ist ein reiner Notfall-Text (Fallback), falls der Wiki-Server einmal wegen Wartungsarbeiten nicht erreichbar sein sollte. Im Normalbetrieb wird immer der Live-Inhalt aus dem Wiki angezeigt.

---

## 📸 4. Neue Fotos für die Slideshow hinzufügen

Die Bildergalerie (Diashow) ganz oben auf der Startseite bezieht ihre Fotos automatisch aus dem Ordner **`slideshow/`**.

### So fügst du neue Fotos hinzu:
1. Klicke auf der GitHub-Startseite dieses Projekts auf den Ordner **`slideshow`**.
2. Klicke oben rechts auf **„Add file" → „Upload files"**.
3. Ziehe dein neues Foto (`.jpg`, `.png` oder `.webp`, am besten im **Querformat**) in das Browserfenster.
4. Klicke unten auf den grünen Button **„Commit changes"**.

**Das war's schon!** Du musst nirgendwo im Code etwas eintragen. Die Webseite erkennt jedes Bild im Ordner `slideshow/` automatisch, schneidet und komprimiert es für schnelle Ladezeiten (auch für Handys) und fügt es der Slideshow hinzu.

*Möchtest du ein altes Bild aus der Slideshow entfernen? Klicke das Bild im Ordner `slideshow/` einfach an und lösche es (siehe Grundlagen oben).*

---

## 🛡️ 5. Neue Logos für TeamSL-Gegner hinzufügen

In den Spielplänen zeigt die Homepage neben den Vereinsnamen automatisch die passenden Vereinswappen an. Alle Logos liegen im Ordner **`dbb_logos/`**.

### Wie funktioniert die Zuordnung?
**Vollautomatisch über den Dateinamen!** Du musst die Logos nirgendwo manuell zuweisen.

1. Besorge dir das Vereinslogo des neuen Gegners (als `.svg`, `.png` oder `.jpg`, möglichst **quadratisch**).
2. **Benenne die Datei auf deinem Computer passend um:**
   * Nutze einen eindeutigen Teil des gegnerischen Vereinsnamens in **Kleinbuchstaben**.
   * Beispiele:
     * `verden.png` → wird automatisch bei *„TV Verden"* oder *„BG Verden II"* angezeigt.
     * `delmenhorst.png` → passt auf *„TV Delmenhorst"*.
     * `bc bremerhaven.png` oder `eisbären.png` → passt auf den jeweiligen Verein aus Bremerhaven.
   * Umlaute (`ä`, `ö`, `ü`, `ß`) und Bindestriche erkennt das System von selbst.
3. Öffne auf GitHub den Ordner **`dbb_logos`**, klicke auf **„Add file" → „Upload files"**, lade die Datei hoch und speichere mit **„Commit changes"**.

> **Kein Logo vorhanden?** Kein Problem! Findet die Webseite kein passendes Logo im Ordner `dbb_logos/`, zeigt sie automatisch ein schickes Wappen mit den ersten beiden Buchstaben des Vereinsnamens an.

---

## 🔄 6. Saisonwechsel: TeamSL IDs anpassen

Wenn im Herbst eine neue Basketball-Saison startet, legt der Verband im DBB-System (TeamSL) neue Ligen an. Dadurch ändern sich jedes Jahr zwei Nummern:
* **`league_id` (Liga-ID, 5-stellig):** Die Nummer der Staffel/Liga (z. B. `54166` für die Regionsliga Süd Herren).
* **`team_id` (Team-ID, meist 6-stellig):** Die Nummer unserer eigenen BiBA-Mannschaft innerhalb dieser Liga (z. B. `448402`). Sie sorgt dafür, dass genau die Spiele dieses Teams gefiltert werden – besonders wichtig, wenn wie in der U12 zwei BiBA-Teams (`mU12 I` und `mU12 II`) in derselben Liga spielen.

### Schritt 1: Wo finde ich die neuen IDs zur neuen Saison?
1. **Liga-ID (`league_id`) finden:**
   * Gehe auf [basketball-bund.net](https://www.basketball-bund.net) und klicke dich zur neuen Liga des Teams durch (z. B. *Wettbewerbe → Niedersachsen*).
   * Schau oben in die Internetadresse (URL) deines Browsers oder halte die Maus über den Link zur Tabelle/Ergebnissen: Dort steht `liga_id=54166`. Diese 5-stellige Zahl ist die **`league_id`**.
2. **Team-ID (`team_id`) finden:**
   * Öffne im Browser `https://slapi.up.railway.app/docs` (unsere TeamSL-Schnittstelle) oder frage kurz beim Website-Admin nach den aktuellen `team_id`-Nummern der neuen Saison.
   * *(Notfall-Tipp: Spielt nur ein einziges BiBA-Team in der Liga und du kennst die neue `team_id` gerade noch nicht, kannst du auf der Teamseite bei `teamsl-matches` statt `team_id="..."` vorübergehend auch `only_biba="true"` schreiben. Dann werden trotzdem alle BiBA-Spiele dieser Liga angezeigt!)*

### Schritt 2: Wo müssen die neuen IDs eingetragen werden?

Es gibt **zwei Orte**, die zum Saisonstart aktualisiert werden müssen:

#### Ort A: In der Datei `hugo.toml` (für die Startseite & schöne Teamnamen)
Öffne auf der obersten Ebene die Datei **`hugo.toml`** und scrolle etwa zu Zeile 55. Dort findest du drei Listen, in denen du die alten Zahlen durch die neuen ersetzt:

1. **`active_leagues`** (bestimmt, aus welchen Ligen Ergebnisse & kommende Spiele auf der **Startseite** gesammelt werden):
   ```toml
   active_leagues = [
     '54166', # Regionsliga Süd Herren
     '54167', # Regionsliga Mixed
     # ... weitere Liga-IDs
   ]
   ```
2. **`[params.league_labels]`** (Kurzbezeichnung für jede Liga-ID):
   ```toml
   [params.league_labels]
     54166 = 'Herren'
     54167 = 'Mixed'
   ```
3. **`[params.biba_team_names]`** (sorgt dafür, dass in den Spielplänen kurz und knackig `BiBA Herren` oder `BiBA mU12.1` steht statt des langen offiziellen Verbandsnamens):
   ```toml
   [params.biba_team_names]
     448402 = 'BiBA Herren'
     448410 = 'BiBA Mixed'
   ```

#### Ort B: Auf den einzelnen Teamseiten in `content/teams/`
Öffne nacheinander die Team-Dateien im Ordner **`content/teams/`** (z. B. `herren.md`, `mu14.md`, ...) und trage ganz unten die neue `league_id` und `team_id` ein:

```markdown
## Spiele & Ergebnisse

{{< teamsl-matches league_id="54166" team_id="448402" limit="6" >}}

---

## Tabelle

{{< teamsl-standings league_id="54166" >}}
```

---

## 🌟 Gut zu wissen: Weitere hilfreiche Tipps

### 1. Wann aktualisieren sich Wiki-Texte, Ergebnisse und Instagram-Posts?
Wenn du einen Text im **Vereins-Wiki** änderst, auf **Instagram** etwas postest oder am Wochenende neue **Spielergebnisse** in TeamSL eingetragen werden, holt sich die Homepage diese Daten **alle 6 Stunden vollautomatisch** ab (jeweils nachts, morgens, mittags und abends). Du musst dafür auf GitHub nichts tun.

### 2. Ich habe im Wiki etwas geändert und möchte es SOFORT auf der Homepage sehen!
Du möchtest nicht bis zur nächsten automatischen Aktualisierung warten? So startest du das Update mit zwei Klicks selbst:
1. Klicke ganz oben auf der GitHub-Seite auf den Reiter **„Actions"** (neben *Code*, *Issues*, *Pull requests*).
2. Klicke in der linken Spalte auf **„Deploy BG BiBA Hugo Site to GitHub Pages"**.
3. Klicke rechts auf den grauen Button **„Run workflow"** und im kleinen Menü auf den grünen Button **„Run workflow"**.
4. Nach ca. 1 Minute sind alle frischen Daten aus dem Wiki, von Instagram und aus TeamSL live auf der Homepage!

### 3. Wo ändere ich Seiten wie „Mitglied werden", „Förderverein", „Kontakt" oder „Impressum"?
Alle allgemeinen Unterseiten liegen direkt im Ordner **`content/`**:
* `content/mitglied-werden.md` → Seite „Mitglied werden"
* `content/foerderverein.md` → Seite „Förderverein" (bindet die Wiki-Seite mit der ID `204` ein)
* `content/kontakt.md` → Allgemeine Kontaktseite
* `content/impressum.md` → Impressum (z. B. bei Vorstands- oder Adressänderungen)
* `content/datenschutz.md` → Datenschutzerklärung

### 4. Wo ändere ich globale Links (Fanshop, WhatsApp-Kanal, Social Media)?
Falls sich einmal der Link zum Fanshop, zum WhatsApp-Kanal oder zu Social Media ändert, findest du diese gesammelt in der Datei **`hugo.toml`** (ab ca. Zeile 68 unter `# Social Media & Externe Links` sowie darunter unter `# Navigation`).
