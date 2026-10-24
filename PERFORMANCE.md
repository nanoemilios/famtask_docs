# Performance: Lade- und Antwortzeiten

Stand **2026-10-24**, App-Version **1.5.0** (Arbeitsstand `[Unveröffentlicht]`).

Ziel: Die App soll auf einem Mittelklasse-Handy in unter 2,5 s bedienbar sein
(LCP) und der laufende Abgleich im Mobilfunknetz kaum Daten und Akku kosten.
Die Landingpage ist heute schon schnell; sie dient als Referenz, wie die App
aussehen kann, wenn kein Transpiler im Weg steht.

> **Stand 2026-10-24:** Stufe 2 (Babel raus) ist umgesetzt. Erster Aufruf
> rund 185 KB gzip statt 628 KB; DOMContentLoaded im Test 0,45 s statt 15-32 s.
> Stufe 1 (Cache-Header, Service Worker) und Stufe 3 (Sync) sind offen.

## 1. Messwerte (vor Stufe 2)

Lokal gemessen (gzip, HTTP/1.0-Testserver ohne Keep-Alive, Desktop-Chrome).
Die Zahlen sind ein Vergleichsmassstab, kein Laborergebnis fuer Mobilgeraete.

| Kennzahl | `famtask` (App) | `famtask_landingpage` |
|---|---|---|
| Uebertragene Bytes | 628 KB | 68 KB |
| Requests | 10 | 8 |
| DOMContentLoaded | 15-32 s\* | 0,4 s |
| `load` | 15-32 s\* | 0,6 s |

\* Der Wert schwankt stark, weil der Browser bei jedem Start rund 300 KB JSX
mit Babel uebersetzt. Der Anteil der JavaScript-Verarbeitung bleibt der
dominierende Faktor, auch wenn die absolute Zeit auf echter Hardware kleiner ist.

### Groesste Einzelposten der App (gzip)

| Datei | roh | gzip | Anteil |
|---|---|---|---|
| `vendor/babel.min.js` | 2.458 KB | 554 KB | 88 % |
| `index.html` (inkl. JSX) | 313 KB | 70 KB | 11 % |
| `vendor/react-dom.production.min.js` | 132 KB | 42 KB | 7 % |
| `lang/de.js` | 42 KB | 14 KB | 2 % |
| `styles.css` | 63 KB | 11 KB | 2 % |
| `vendor/react.production.min.js` | 11 KB | 4 KB | < 1 % |

### Landingpage

- 59 KB Ressourcen: Nunito Latin 39 KB, Fredoka One 16 KB, Rest unter 5 KB.
- Nur das Latin-Subset wird geladen (`unicode-range` in `vendor/fonts/fonts.css`),
  weitere Subsets erst, wenn Zeichen daraus vorkommen.
- Schriften liegen seit Paket 2 lokal: keine Anfragen mehr an
  `fonts.googleapis.com` / `fonts.gstatic.com` (weniger DNS/TLS-Handshakes,
  kein externer Single Point of Failure).

## 2. Befunde (Ursachen)

1. **Babel im Browser.** `index.html` laedt `vendor/babel.min.js` (2,4 MB) und
   uebersetzt das komplette JSX bei jedem Seitenaufruf neu. Das blockiert den
   Main Thread, bevor React ueberhaupt rendern kann, und laesst sich nicht
   cachen, weil das JSX inline im HTML steht.
2. **Monolithisches HTML.** 313 KB `index.html` mit JSX, Styles und Logik.
   Jede Aenderung invalidiert das gesamte Dokument; Teile koennen nicht
   einzeln gecacht werden.
3. **Voller Zustand bei jedem Poll.** `_loadData()` holt alle 8 s den
   kompletten Key-/Value-Bestand (`api.php` -> `handleGet`, eine Zeile pro
   Datenbereich). Mit wachsender Historie wird jeder Poll teurer, auch wenn
   sich nichts geaendert hat.
4. **Voller Zustand beim Speichern.** `_pushData()` schickt nach 800 ms
   Debounce den kompletten Zustand; `handlePost` schreibt jeden Key erneut
   (`INSERT ... ON DUPLICATE KEY UPDATE`) statt nur der geaenderten Zeile.
5. **Kein Service-Worker-Cache.** `sw.js` hat nur einen leeren `fetch`-Handler.
   Die App-Shell wird bei jedem Start neu geladen; offline startet sie nicht.
6. **Keine Kompression/Cache-Header in `.htaccess`.** Ob gzip und
   Cache-Dauern greifen, haengt allein am Hoster. Ohne gzip sind es 2,6 MB
   statt 628 KB.
7. **Sprachdateien.** `lang/de.js` wird immer geladen (Rueckfallbasis fuer alle
   Sprachen), weitere Sprachen kommen per `loadLang()` dazu. Das ist bereits
   bedarfsgerecht; optimierbar ist nur die Groesse der Dateien selbst.

## 3. Massnahmen

Reihenfolge nach Wirkung pro Aufwand. Jede Stufe ist einzeln auslieferbar.

### Stufe 1 - Sofort, ohne Umbau (ca. 0,5 Tag)

1. **`.htaccess`: Kompression und Cache-Dauern**
   - `mod_deflate` fuer `text/html`, `text/css`, `application/javascript`,
     `application/json`, `image/svg+xml`, `font/woff2` (bereits komprimiert:
     ausnehmen).
   - `mod_expires`: `icons/*` und `vendor/fonts/*` ein Jahr (`immutable`),
     CSS/JS eine Woche, HTML `no-cache` (damit Updates ankommen).
   - Wirkung: erster Aufruf 2,6 MB -> 628 KB; Wiederbesuch ohne Netzverkehr
     fuer unveraenderte Dateien. Fuer nginx-Hoster denselben Block in
     `DEPLOYMENT.md` dokumentieren.
2. **Service Worker mit App-Shell-Cache**
   - `index.html`: network-first mit Cache-Fallback (Updates kommen an).
   - `vendor/*`, `icons/*`, `styles.css`, `lang/*`: cache-first.
   - `api.php`, `jukebox.php`, `remote.php`: nie cachen.
   - Wirkung: Wiederbesuch startet in wenigen hundert Millisekunden, App-Shell
     ist offline verfuegbar; Schreibvorgaenge laufen ueber die bestehende
     Offline-Logik.
   - Risiko: Cache-Versionierung. Bei jedem Release `CACHE_VERSION` erhoehen.

### Stufe 2 - Babel entfernen (groesster Hebel, ca. 1 Tag) - umgesetzt

- JSX aus `index.html` nach `src/app.jsx` auslagern und mit **esbuild**
  vorbauen (`tools/build_app.mjs`, Node ist im Projektumfeld vorhanden).
- `dist/app.js` wird versioniert, `index.html` laedt nur noch
  `<script defer src="dist/app.js"></script>`.
- Babel entfaellt im Produktivpfad; `?dev=1` kann weiterhin den
  In-Browser-Transpiler laden, wenn schnell experimentiert werden soll.
- Wirkung: 554 KB gzip JS und 70 KB gzip HTML weniger, kein Transpile beim
  Start. Erwartung: DOMContentLoaded/LCP unter 1 s statt Sekunden, TBT nahe 0.
- Aufwand im Alltag: nach Aenderungen `node tools/build_app.mjs` (oder
  `--watch`). Das gebaute `app.js` liegt im Repository; ohne Build faellt die
  Seite auf `src/app.jsx` plus `vendor/babel.min.js` zurueck (`?dev=1`).
- Umgesetzt: erster Aufruf 185 KB gzip (app.js 54 KB, react-dom 43 KB,
  Schriften 55 KB, Rest klein), `index.html` 2,9 KB gzip, kein Babel mehr im
  Normalfall. Der Build schreibt einen Inhalts-Hash als `?v=...` in `index.html`.

### Stufe 3 - Sync und API (ca. 1 Tag)

1. **Leichtgewichtiger Poll**: neue Aktion `action=version` liefert nur
   `{ts}`. Der Client pollt diese alle 8 s (rund 200 Byte) und laedt den
   Vollzustand nur, wenn sich `ts` geaendert hat.
2. **`ETag`/`If-None-Match`** in `handleGet`: unveraenderter Zustand -> `304`
   ohne Body, auch fuer den manuellen Sync.
3. **Nur geaenderte Keys schreiben**: `handlePost` bekommt
   `{_changed:[...], ...}` und schreibt nur diese Zeilen.
4. **Poll nur bei sichtbarem Tab**: `visibilitychange` loest beim Zurueckkehren
   einen Sofort-Sync aus, im Hintergrund wird seltener gepollt.
- Wirkung: Mobilfunk-Traffic pro Stunde von geschaetzt 1-2 MB auf unter
  100 KB; Speichern einer Aenderung schreibt eine Zeile statt aller.
- Risiko: Mehrgeraete-Abgleich. Test mit zwei Tabs bzw. zwei Geraeten und
  bewusst erzeugten Konflikten (beide offline, dann beide online).

### Stufe 4 - Feinschliff (optional)

- Jukebox-Cover als WebP/AVIF mit `loading="lazy"` und `immutable`-Cache.
- `content-visibility: auto` fuer Landingpage-Sektionen unterhalb des Folds.
- CSS/HTML beim Build minifizieren.
- Lighthouse-CI mit Budgets (JS < 200 KB gzip, LCP < 2,5 s, CLS < 0,1).

## 4. Zielwerte

| Kennzahl | heute | Ziel |
|---|---|---|
| JavaScript gzip (Start) | 596 KB | < 150 KB |
| HTML gzip | 70 KB | < 15 KB |
| Requests erster Aufruf | 10 | 8 |
| LCP (4G, Mittelklasse) | geschaetzt 3-6 s | < 2,5 s |
| TBT | hoch (Transpile) | < 200 ms |
| Sync-Traffic pro Stunde | geschaetzt 1-2 MB | < 100 KB |
| Speichern einer Aenderung | alle Keys | 1 Key |

## 5. Messplan

Vor und nach jeder Stufe dieselbe Messung, damit die Wirkung belegbar bleibt:

1. **Netz**: Chrome DevTools, "Disable cache", Mobil-Profil (4G, 4x CPU).
   Werte aus `performance.getEntriesByType('navigation')` und
   `performance.getEntriesByType('resource')` (Bytes, Requests, DCL, `load`).
2. **Lighthouse** (mobil) fuer LCP, TBT, CLS und Accessibility. Accessibility
   bleibt bei 100 - die Aenderungen duerfen das nicht verschlechtern.
3. **Antwortzeiten**: `curl -w '%{time_total} %{size_download}'` gegen
   `api.php` (GET/POST) mit realistischer Datenmenge, dazu der neue
   `action=version`-Poll.
4. **Sync**: zwei Geraete/Tabs, Aenderung auf A, Pruefung auf B (unter 10 s),
   danach beide offline/online.

## 6. Empfehlung

1. **Stufe 2 (umgesetzt)** - groesster Einzelgewinn: die App startet ohne
   Transpiler, auch auf aelteren Handys.
2. **Stufe 1 als Naechstes** - unabhaengig von der App-Logik, spuerbar bei
   jedem Wiederbesuch und Voraussetzung dafuer, dass `app.js` lange im
   Browser-Cache bleiben darf.
3. **Stufe 3 danach** - wichtig fuer Mobilfunk und Akku, braucht aber einen
   sauberen Mehrgeraete-Test.
4. Stufe 4 nur, wenn die ersten drei stehen.
