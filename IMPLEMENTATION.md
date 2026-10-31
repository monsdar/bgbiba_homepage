# Technische Implementierungsdokumentation (BG BiBA Hugo)

Dieses Dokument beschreibt die Architektur, Designentscheidungen und technischen Details der Implementierung der BG BiBA Homepage.

---

## 1. Architektur-Prinzipien

1. **Native Build-Time Execution (Zero Runtime / No Pre-Build):**
   * Es existieren keine separaten Pre-Build-Skripte (z. B. Python, Node.js oder Shell).
   * Alle externen Schnittstellen werden nativ durch Hugo mittels `resources.GetRemote` angesprochen.
   * Das reduziert Abhängigkeiten, vermeidet Versionskonflikte in CI/CD-Umgebungen und beschleunigt den Build drastisch (< 150 ms Buildzeit).

2. **Theme-less Custom Layouts:**
   * Anstelle eines externen Themes (Git-Submodule oder Hugo Modules) sind alle Layouts direkt in `layouts/` und alle Styles in `assets/css/` abgelegt.
   * Vorteil: 100%ige Code-Kontrolle, keine Breaking Changes durch Third-Party-Updates, schlankes HTML5 ohne Ballast.

3. **Sicherheitsarchitektur & Whitelist:**
   * Externe API-Tokens dürfen niemals im Repository gespeichert werden.
   * Hugos Standard-Sicherheitsmodell blockiert standardmäßig Environment-Zugriffe und Remote-Requests.
   * In [hugo.toml](file:///e:/Projects/Biba_Homepage/hugo.toml) ist die Whitelist explizit definiert:
     ```toml
     [security]
       enableInlineShortcodes = false
       [security.funcs]
         getenv = ['^BS_', '^SLAPI_', '^TEAMSL_', '^GA_']
       [security.http]
         methods = ['(?i)GET']
         urls = ['.*']
     ```

4. **Graceful Degradation (Offline-Resilienz):**
   * Lokale Builds (`hugo server`) dürfen niemals durch fehlende Secrets oder nicht erreichbare Remote-Server mit einem Fatal Error abbrechen.
   * Sämtliche Komponenten besitzen automatische Fallbacks auf lokale Beispieldaten (`assets/data/`) oder rendern informative Vorschau-Boxen.

---

## 2. Komponenten-Details

### 2.1 BookStack Integration (`layouts/shortcodes/bookstack.html`)
* **Ziel:** Redaktionelle Inhalte (Trainingszeiten, Trainer, Ansprechpartner) werden im Vereins-Wiki gepflegt und synchronisiert.
* **API-Endpunkt:** `https://wiki.bgbiba.de/api/pages/<id>/export/markdown`
* **Authentifizierung:** BookStack verwendet den Token-Header `Authorization: Token <token_id>:<token_secret>`.
* **Code-Ablauf:**
  ```go
  {{- $id := .Get "id" -}}
  {{- $bsId := os.Getenv "BS_ID" -}}
  {{- $bsSecret := os.Getenv "BS_SECRET" -}}
  
  {{- if and $bsId $bsSecret -}}
    {{- $headers := dict "Authorization" (printf "Token %s:%s" $bsId $bsSecret) -}}
    {{- $res := resources.GetRemote $endpoint (dict "headers" $headers) -}}
    {{- if and $res (not $res.Err) -}}
      <div class="bookstack-content">{{ $res.Content | markdownify }}</div>
    {{- end -}}
  {{- else -}}
    {{/* Entwicklungsmodus / Fallback */}}
    <div class="bookstack-box bookstack-dev-preview">...</div>
  {{- end -}}
  ```
* **Fallback:** Fehlen `BS_ID` oder `BS_SECRET`, wird der innere Content des Shortcodes (falls vorhanden) bzw. eine Vorschau-Box gerendert.

---

### 2.2 TeamSL DBB REST-API (`layouts/partials/teamsl-matches.html` & `teamsl-standings.html`)
* **Ziel:** Direkter Bezug von Ligen, Spielplänen und Tabellen aus dem DBB TeamSL Wrapper (`https://slapi.up.railway.app/`).
* **Authentifizierung:** `Authorization: Bearer <SLAPI_TOKEN>`.
* **Endpunkte:**
  * Matches: `GET /leagues/{league_id}/matches`
  * Standings: `GET /leagues/{league_id}/standings`
* **Implementierte Logik:**
  * Datumsformatierung ins deutsche Format (`02.01.2006, 15:04 Uhr`).
  * Hervorhebung von BG BiBA Mannschaften via CSS-Klasse `.match-biba` und `.team-biba`.
  * Dynamische Status-Badges: `Endstand` (inkl. Punktestand), `Geplant` (mit VS) oder `Abgesagt`.
  * Filter-Parameter `only_biba`: Schränkt Spiele wahlweise auf BiBA-Beteiligung ein.
* **Fallback-Mechanismus:**
  * Wenn `$token` fehlt oder `$res.Err` vorliegt, lädt Hugo automatisch:
    * `resources.Get "data/sample-matches.json"`
    * `resources.Get "data/sample-standings.json"`
  * Der Entwickler erkennt dies am Badge `Vorschau` anstelle von `● Live Sync`.

---

### 2.3 CSV-Spielplan (`layouts/partials/matches.html`)
* **Ziel:** Einfache Einbindung von Turnier- oder Offline-Spielplänen.
* **Format:** Komma- oder Semikolon-getrennte CSV mit den Spalten `Datum, Uhrzeit, Heim, Gast, Spielstaette, Ergebnis`.
* **Parsing:**
  ```go
  {{ $rows := $csvData | transform.Unmarshal (dict "delimiter" ",") }}
  {{ $header := index $rows 0 }}
  {{ $dataRows := after 1 $rows }}
  ```
* **Fallback:** Rückgriff auf `assets/data/sample-matches.csv`.

---

### 2.4 Cookie-Banner & Google Analytics (`layouts/partials/cookie-banner.html`)
* **DSGVO-Prinzip:** Kein Tracking-Skript vor ausdrücklicher Einwilligung (*Opt-In*).
* **Funktionsweise:**
  1. Beim Seitenaufruf wird `localStorage.getItem('bgbiba_cookie_consent')` geprüft.
  2. Ist kein Status vorhanden, blendet sich das modale Banner verzögert ein.
  3. Bei Klick auf *„Alle akzeptieren“* wird das Consent-Token gespeichert und das GA4-Tag dynamisch in den `<head>` injiziert:
     ```javascript
     function loadGoogleAnalytics(id) {
       var script = document.createElement('script');
       script.async = true;
       script.src = 'https://www.googletagmanager.com/gtag/js?id=' + id;
       document.head.appendChild(script);
       // gtag Initialisierung mit anonymize_ip: true
     }
     ```
  4. Über den globalen Aufruf `openCookieSettings()` im Footer kann der Nutzer die Auswahl jederzeit revidieren.

---

### 2.5 Asset-Pipeline (Hugo Pipes)
In `layouts/_default/baseof.html` werden die Stylesheets verarbeitet:
```go
{{- $css := resources.Get "css/main.css" | minify | fingerprint -}}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```
* **Minification:** Komprimiert das CSS automatisch für die Produktion.
* **Fingerprinting:** Hängt einen kryptografischen Hash an den Dateinamen für Cache-Busting und Subresource Integrity (`integrity="..."`).

---

## 3. CI/CD Deployment Workflow (`.github/workflows/deploy.yml`)

Der Workflow ist für GitHub Pages optimiert:
* **Trigger:**
  * Push auf `main`
  * Cron: `0 6 * * 1,4` (Mo & Do 06:00 UTC)
  * Manueller Trigger: `workflow_dispatch`
* **Umgebungsvariablen im Build-Schritt:**
  ```yaml
  env:
    BS_ID: ${{ secrets.BS_ID }}
    BS_SECRET: ${{ secrets.BS_SECRET }}
    SLAPI_TOKEN: ${{ secrets.SLAPI_TOKEN }}
    GA_MEASUREMENT_ID: ${{ secrets.GA_MEASUREMENT_ID }}
  run: |
    hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
  ```
* **Deployment:** Nutzung der offiziellen Aktionen `actions/upload-pages-artifact@v3` und `actions/deploy-pages@v4`.
