# BG BiBA – Vereinswebsite (Hugo)

Offizielle Webpräsenz der **BG BiBA** (*Basketballgemeinschaft Bierden-Bassen-Achim*) – einer Spielgemeinschaft der drei Stammvereine **TSV Bierden von 1928 e.V.**, **TSV Bassen e.V.** und **TSV Achim 1860 e.V.** im Landkreis Verden.

Die Website basiert auf **Hugo (Extended)** und ist für ein schnelles, sicheres und vollständig automatisiertes Deployment via **GitHub Pages** konzipiert.

---

## 🏀 Features & Funktionen

* **Zero Pre-Build Skripte:** Alle externen Inhalte (Wiki-Texte, Spielpläne, Tabellen) werden zur Build-Zeit direkt und nativ über Hugos `resources.GetRemote` geladen – ohne separate Python-, Node- oder Shell-Skripte.
* **BookStack Wiki Integration (`{{< bookstack id="..." >}}`):**
  * Synchronisiert Trainingszeiten, Trainer-Infos und Ansprechpartner direkt aus dem Vereins-Wiki (`https://wiki.bgbiba.de/`).
  * Sichere Header-Authentifizierung via Token.
  * Automatisches Markdown-Rendering per `markdownify`.
* **DBB TeamSL REST-API (`{{< teamsl-matches >}}` & `{{< teamsl-standings >}}`):**
  * Direkte Anbindung an den TeamSL API-Server (`https://slapi.up.railway.app/`).
  * Holt Ligaspiele, Spielstände, Hallenorte und Tabellenstände der Mannschaften live zur Build-Zeit.
  * Spezielle Hervorhebung von BG BiBA Teams.
* **Universeller CSV-Spielplan (`{{< matches >}}`):**
  * Ermöglicht die Einbindung von Spielplänen für Turniere, Freundschaftsspiele oder Sonder-Events per CSV-Datei (Remote oder lokal) via `transform.Unmarshal`.
* **DSGVO & Datenschutzkonformität:**
  * Modernes Cookie-Banner mit Opt-In: Google Analytics 4 (GA4) wird erst nach ausdrücklicher Einwilligung des Nutzers geladen.
  * Einwilligungen werden lokal gespeichert und können jederzeit über die Datenschutzerklärung bzw. den Footer geändert oder widerrufen werden.
  * Vollständiges, deutsches Impressum und Datenschutzerklärung.
* **Modernes Athletic Design System (Vanilla CSS):**
  * Maßgeschneidertes, responsives Design mit BiBA-Vereinsfarben (Marineblau, Basketball-Orange).
  * Gestochen scharfe Vektor- und Bildlogos, mobilfreundliche Navigation und Card-Grids für Teams, News und Spiele.
* **Vollautomatisiertes GitHub Actions CI/CD:**
  * Automatisches Deployment bei Push auf `main`.
  * **Cron-Job (2x wöchentlich):** Baut die Seite automatisch jeden Montag und Donnerstag um 06:00 UTC neu, um frische Wochenend-Spielergebnisse und Vorschauen zu veröffentlichen.
  * Manueller Build per GitHub-Button (`workflow_dispatch`).

---

## 📁 Projektstruktur

```text
├── .github/workflows/
│   └── deploy.yml          # GitHub Actions Pages Deployment & Cron
├── assets/
│   ├── css/
│   │   └── main.css        # Zentrales Design-System & Tokens (Hugo Pipes)
│   ├── data/               # Lokale Mock-/Fallback-Daten (JSON & CSV)
│   └── images/             # Logos für Asset-Pipeline
├── content/
│   ├── teams/              # Team-Seiten (U10 bis Senioren & Daddeltruppe)
│   ├── _index.md           # Startseiten-Metadaten
│   ├── datenschutz.md      # DSGVO-Datenschutzerklärung
│   ├── hallen.md           # Trainings- und Spielstätten
│   ├── impressum.md        # Impressum gem. § 5 TMG
│   ├── spielbetrieb.md     # Spielpläne & Tabellen
│   └── verein.md           # Über die BG BiBA & Stammvereine
├── layouts/
│   ├── _default/           # Baseof, Single und List Templates
│   ├── partials/           # Header, Footer, Cookie-Banner, Matches, Standings
│   ├── shortcodes/         # BookStack, TeamSL und CSV Shortcodes
│   └── index.html          # Startseiten-Layout
├── static/                 # Statische Dateien (Logos, Icons, etc.)
├── .env.example            # Vorlage für lokale Umgebungsvariablen
├── hugo.toml               # Hugo-Konfiguration & Security-Whitelists
└── README.md
```

---

## 🚀 Lokale Entwicklung

### Voraussetzungen
* **Hugo Extended** (Version `v0.134.1` oder neuer). Prüfen mit:
  ```bash
  hugo version
  ```

### 1. Repository klonen & einrichten
```bash
git clone <repo-url>
cd Biba_Homepage
```

### 2. Lokale Umgebungsvariablen (`.env`)
Erstelle eine `.env` Datei im Projektstamm (siehe `.env.example`):
```ini
# BookStack API Credentials
BS_ID=deine_bookstack_token_id
BS_SECRET=dein_bookstack_token_secret

# TeamSL API Token
SLAPI_TOKEN=dein_teamsl_api_token

# Optional: Google Analytics ID
GA_MEASUREMENT_ID=G-XXXXXXXXXX
```
> **Hinweis:** Die `.env` Datei wird durch `.gitignore` geschützt und niemals in Git übertragen. Fehlen die Variablen lokal, nutzt Hugo automatisch sichere Fallbacks (Mock-Daten und Vorschau-Texte), sodass der lokale Server jederzeit lauffähig bleibt.

### 3. Entwicklungsserver starten
```bash
hugo server
```
Die Website ist nun unter **`http://localhost:1313/`** mit automatischem Live-Reload erreichbar.

---

## 🔐 GitHub Secrets für das Deployment

Damit GitHub Actions die externen Inhalte im Live-Betrieb laden kann, müssen im GitHub-Repository unter **Settings → Secrets and variables → Actions** folgende Repository Secrets hinterlegt werden:

| Secret Name | Beschreibung | Beispiel / Format |
| :--- | :--- | :--- |
| `BS_ID` | BookStack API Token ID | `ANUwocdx...` |
| `BS_SECRET` | BookStack API Token Secret | `ENKb2cI2...` |
| `SLAPI_TOKEN` | TeamSL DBB API Bearer Token | `89917e76-3e85-...` |
| `GA_MEASUREMENT_ID` | Google Analytics 4 ID *(optional)* | `G-XXXXXXXXXX` |

Unter **Settings → Pages** muss als Source **GitHub Actions** gewählt werden.

---

## 🏀 Wichtige Shortcodes

### 1. BookStack Inhalt einbinden
```markdown
{{< bookstack id="12" >}}
```
Oder mit lokalem Fallback-Text:
```markdown
{{< bookstack id="12" >}}
Hier steht der Text, falls BookStack offline ist.
{{< /bookstack >}}
```

### 2. TeamSL Spiele anzeigen
```markdown
{{< teamsl-matches league_id="12001" limit="5" only_biba="true" >}}
```

### 3. TeamSL Tabelle anzeigen
```markdown
{{< teamsl-standings league_id="12001" >}}
```

### 4. CSV-Spielplan anzeigen
```markdown
{{< matches url="https://example.com/plan.csv" limit="10" >}}
```
*(Wird keine URL angegeben, wird der lokale Beispieldaten-Spielplan gerendert).*
