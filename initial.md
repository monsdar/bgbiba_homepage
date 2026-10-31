Erstelle das Grundgerüst für eine Sportvereins-Homepage auf Basis von Hugo, optimiert für das Deployment via GitHub Pages. Ziel ist eine statische Webseite zu bauen die als Gerüst für weiteren Content aus anderen Quellen dient: Bookstack für Texte, offene CSVs für Spielpläne und eine REST-API für Ergebnisse. Später sollen News über einen Instagram Feed aufgenommen werden. Es soll möglich sein Statistiken über Google Analytics aufzunehmen. Es wird definitiv ein Cookie-Banner, Impressum, Datenschutzerklärung und Co geben. Später soll das Design nach meinen Wünschen weiter detailliert werden. 

### Kernanforderungen & Architektur
1. **Keine Pre-Build-Skripte:** Alle externen Inhalte werden zur Build-Zeit nativ von Hugo über `resources.GetRemote` geladen.
2. **Sicherheit:** Externe API-Tokens dürfen nicht im Repository liegen, sondern werden über Umgebungsvariablen (`os.Getenv`) aus den GitHub Secrets bezogen.
3. **Automatisierung:** Standard-Build über GitHub Actions ohne zusätzliche Skripte oder Runtimes.

### Zu erstellende Komponenten

1. **Konfiguration (`hugo.toml`):**
   - Grundlegende Konfiguration (BaseURL, Sprache Deutsch, Titel).
   - Freigabe der Umgebungsvariablen unter `[security.funcs]` für BookStack (`getenv = ['^BS_']`).

2. **BookStack-Integration (`layouts/shortcodes/bookstack.html`):**
   - Shortcode, der über den Parameter `id` den Markdown-Endpunkt abruft: `https://<wiki-url>/api/pages/<id>/export/markdown`.
   - Header-Authentifizierung mit `BS_ID` und `BS_SECRET` via `os.Getenv`.
   - Fehlerbehandlung mit `try` und Rendern des Markdowns via `markdownify`.

3. **Spielplan-Komponente (`layouts/partials/matches.html`):**
   - Lädt eine Remote-CSV mit Spielansetzungen über `resources.GetRemote`.
    - CSV wird später durch REST-API Aufrufe ersetzt. Das ist erstmal nur für den Prototypen
   - Parsen via `transform.Unmarshal`.
   - Strukturierte Ausgabe (Datum, Heim, Gast, Ergebnis/Uhrzeit, Spielstätte) in einer übersichtlichen Tabelle/Kartenansicht.

4. **Basis-Layout & Design:**
   - Minimalistisches, responsives Layout (Card-Grid für Teams, News und nächste Spiele).
   - Beispiel-Seitenstruktur:
     - `content/teams/u12.md` (bindet den BookStack-Shortcode ein).
     - `content/_index.md` (Startseite mit Match-Widget und News-Platzhaltern).

5. **CI/CD (`.github/workflows/deploy.yml`):**
   - Offizielle GitHub Action für Hugo Pages (`actions/configure-pages`, `peaceiris/actions-hugo` oder standardmäßiges `hugo --minify`).
   - Einbindung der Secrets `BS_ID` und `BS_SECRET` als Environment Variables.
   - Trigger bei Push auf `main` sowie zeitgesteuert per `cron` (z. B. montags und donnerstags).