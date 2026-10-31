# AGENTS.md – Richtlinien für zukünftige AI-Agenten

Willkommen in der Codebase der **BG BiBA Homepage**! Dieses Dokument enthält verbindliche Leitlinien, Architektur-Regeln und Konventionen für alle AI-Coding-Assistenten und Agenten, die an diesem Projekt arbeiten.

---

## 1. Projektkontext & Identität

* **Organisation:** BG BiBA (*Basketballgemeinschaft Bierden-Bassen-Achim*).
* **Stammvereine:** TSV Bierden von 1928 e.V., TSV Bassen e.V., TSV Achim 1860 e.V.
* **Sportart:** Basketball (DBB / Basketball-Verband Niedersachsen / Bremen).
* **Tech-Stack:** Hugo Extended (`>= v0.134.1`), Vanilla CSS (Hugo Pipes), GitHub Pages CI/CD.
* **Externe Schnittstellen:**
  * BookStack Vereins-Wiki: `https://wiki.bgbiba.de/`
  * TeamSL DBB REST-API: `https://slapi.up.railway.app/`

---

## 2. Nicht verhandelbare Grundregeln (MANDATORY)

### 🔴 1. Keine Secrets im Code oder Git
* **Niemals** API-Tokens, Secrets, Passwörter oder private IDs in Dateien schreiben, die in Git versioniert werden.
* Alle sensiblen Werte müssen über `os.Getenv` bezogen werden.
* Neue Umgebungsvariablen müssen in `hugo.toml` unter `[security.funcs]` explizit per Regex freigegeben werden:
  ```toml
  [security.funcs]
    getenv = ['^BS_', '^SLAPI_', '^TEAMSL_', '^GA_', '^DEIN_PREFIX_']
  ```
* Dokumentiere neue Variablen in `.env.example`.

### 🔴 2. Keine Pre-Build-Skripte
* Externe Datenquellen (Wiki, APIs, CSVs) **müssen** zur Build-Zeit nativ über Hugos `resources.GetRemote` abgefragt werden.
* Führe **keine** separaten Python-, Node- oder Shell-Skripte ein, die vor `hugo` ausgeführt werden müssen.

### 🔴 3. Ausfallsicherheit & Lokale Fallbacks
* **Der lokale Entwicklungs-Build (`hugo server`) darf NIEMALS mit einem Fatal Error abbrechen**, wenn Secrets fehlen oder externe Server offline sind.
* Jeder `resources.GetRemote`-Aufruf muss geprüft werden (`if and $res (not $res.Err)`).
* Für alle externen Schnittstellen existieren Fallback-Dateien in `assets/data/`:
  * `sample-matches.json` (TeamSL Spiele)
  * `sample-standings.json` (TeamSL Tabelle)
  * `sample-matches.csv` (CSV Spielplan)
* Bei BookStack: Render im Dev-Modus eine Vorschau-Box oder den inneren Fallback-Text.

### 🔴 4. Keine ungefragten CSS-Frameworks
* Verwende **kein** TailwindCSS, Bootstrap o.ä., es sei denn, der User fordert es ausdrücklich an.
* Pflege bestehende Design-Tokens und Stylesheets in [assets/css/main.css](file:///e:/Projects/Biba_Homepage/assets/css/main.css).
* CSS wird über Hugo Pipes gebündelt (`resources.Get "css/main.css" | minify | fingerprint`).

### 🔴 5. Sprache & Lokalisierung
* Alle Menüeinträge, Buttons, Systemtexte, Metadaten und Inhalte sind auf **Deutsch (`de-DE`)** zu halten.
* Datums- und Zeitformate nach deutscher Konvention (`TT.MM.JJJJ, HH:MM Uhr`).

---

## 3. Dateistruktur & Ablageorte

| Aufgabentyp | Zielordner / Zieldatei | Hinweise |
| :--- | :--- | :--- |
| **Neue Teamseite anlegen** | `content/teams/<slug>.md` | Parameter `category`, `icon`, `title` im Frontmatter setzen. |
| **Neuer Shortcode** | `layouts/shortcodes/<name>.html` | Ermöglicht Aufruf im Markdown via `{{< name ... >}}`. |
| **Neues Partial / Widget** | `layouts/partials/<name>.html` | Einbindung via `{{ partial "<name>.html" . }}`. |
| **Styles & Design-Tokens** | `assets/css/main.css` | Variablen unter `:root` erweitern. |
| **Mock- / Fallback-Daten** | `assets/data/` | JSON- oder CSV-Dateien für Offline-Betrieb. |
| **Statische Bilder & Logos** | `static/images/` und `assets/images/` | Für direkten URL-Zugriff (`/images/...`) und Hugo Pipes. |
| **Globale Einstellungen** | `hugo.toml` | Menüs, Security-Whitelists und Vereins-Params. |
| **Deployment-Anpassungen** | `.github/workflows/deploy.yml` | Cron-Schedules und Environment-Secrets. |

---

## 4. Test- & Validierungsbefehle

Bevor du deine Änderungen abschließt, führe **immer** folgende Prüfungen im Terminal aus:

1. **Syntax- & Build-Check:**
   ```powershell
   hugo --buildDrafts
   ```
   *Erwartetes Ergebnis:* Exit-Code `0`, keine Warnungen, schnelle Buildzeit (< 200 ms).

2. **Git-Status prüfen:**
   ```powershell
   git status
   ```
   *Erwartetes Ergebnis:* Keine `.env`-Dateien oder Secrets in untracked Files.
