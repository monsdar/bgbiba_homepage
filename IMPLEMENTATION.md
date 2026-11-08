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
         getenv = ['^BS_', '^SLAPI_', '^TEAMSL_', '^INSTAGRAM_', '^IG_']
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

### 2.3 Datenschutzfreundliche Webanalyse via GoatCounter (`layouts/_default/baseof.html`)
* **DSGVO-Prinzip:** Komplett Cookie-freie Reichweitenmessung ohne Speicherung personenbezogener Daten (kein Cookie-Banner erforderlich).
* **Funktionsweise:**
  * Das GoatCounter-Skript (`https://bgbiba.goatcounter.com/count`) wird in `baseof.html` über `{{- if and hugo.IsProduction (not hugo.IsServer) }}` ausschließlich im Production-Build eingebunden, sodass lokale Entwicklungsserver (`hugo server`) keine Zugriffe zählen.

---

### 2.4 Asset-Pipeline (Hugo Pipes)
In `layouts/_default/baseof.html` werden die Stylesheets verarbeitet:
```go
{{- $css := resources.Get "css/main.css" | minify | fingerprint -}}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```
* **Minification:** Komprimiert das CSS automatisch für die Produktion.
* **Fingerprinting:** Hängt einen kryptografischen Hash an den Dateinamen für Cache-Busting und Subresource Integrity (`integrity="..."`).

---

### 2.5 Instagram News Feed Integration (`layouts/partials/instagram-news.html`)
* **Ziel:** Die neuesten Beiträge und Reels aus dem Instagram-Kanal [@bgbiba](https://www.instagram.com/bgbiba/) werden als aktuelle Vereins-News auf der Startseite dargestellt.
* **100 % DSGVO-Konformität:**
  * Der Abruf erfolgt zur Build-Zeit via `resources.GetRemote` über die offizielle Meta Graph API:
    `https://graph.instagram.com/me/media?fields=id,caption,media_type,media_url,thumbnail_url,permalink,timestamp&limit=6&access_token=<INSTAGRAM_TOKEN>`
  * Webseiten-Besucher haben keinen direkten Kontakt zu Meta-Servern (keine Cookies, kein Tracking).
* **Interaktives Lightbox-Modal:**
  * Klick auf eine News-Card öffnet ein barrierefreies HTML5 `<dialog>` Element mit Vollbild/Video-Player, vollständigem Text und direktem Link: *„Auf Instagram ansehen & kommentieren ↗“*.
  * Konform mit den `modern-web-guidance`-Best-Practices (Escape-Schließen, Light-Dismiss Backdrop-Click, Video-Pause bei Close).
* **Fallback-Mechanismus:**
  * Fehlt `INSTAGRAM_TOKEN` oder ist Meta offline, lädt Hugo automatisch [assets/data/sample-instagram.json](file:///e:/Projects/Biba_Homepage/assets/data/sample-instagram.json).

---

## 3. CI/CD Deployment Workflow (`.github/workflows/deploy.yml`)

Der Workflow ist für GitHub Pages und dynamische Content-Quellen optimiert:
* **Trigger:**
  * Push auf `main`
  * Cron: `0 */6 * * *` (Alle 6 Stunden: 00:00, 06:00, 12:00, 18:00 UTC)
  * Manueller Trigger: `workflow_dispatch`
* **Vorgelagerter Step: Instagram Token-Refresh:**
  * Vor dem Hugo-Build prüft die Action den Status des Tokens bei Metas offiziellem Refresh-Endpunkt:
    `GET https://graph.instagram.com/refresh_access_token?grant_type=ig_refresh_token&access_token=${IG_TOKEN}`
  * Gibt Meta einen erneuerten Token zurück, wird dieser direkt für den Build bereitgestellt und optional via `gh secret set INSTAGRAM_TOKEN` im Repository aktualisiert (falls ein PAT mit `secrets:write` als `SECRETS_PAT` hinterlegt ist).
* **Umgebungsvariablen im Build-Schritt:**
  ```yaml
  env:
    BS_ID: ${{ secrets.BS_ID }}
    BS_SECRET: ${{ secrets.BS_SECRET }}
    SLAPI_TOKEN: ${{ secrets.SLAPI_TOKEN }}
    INSTAGRAM_TOKEN: ${{ env.ACTIVE_IG_TOKEN || secrets.INSTAGRAM_TOKEN }}
  run: |
    hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
  ```
* **Deployment:** Nutzung der offiziellen Aktionen `actions/upload-pages-artifact@v3` und `actions/deploy-pages@v4`.

