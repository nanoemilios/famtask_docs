# Architektur und Datenmodell

## Komponenten

| Komponente | Aufgabe |
|---|---|
| `index.html`, `styles.css`, `i18n.js`, `lang/` | Browseroberflaeche, Styles und Uebersetzungen; `index.html` laedt `app.js` |
| `src/app.jsx` | React-Quelltext (JSX), die Quelle der Wahrheit |
| `app.js` | Vorgebaute App aus `src/app.jsx`; wird ausgeliefert, nicht bearbeitet |
| `tools/build_app.mjs` | Baut `app.js` mit dem mitgelieferten Babel (kein npm noetig) |
| `vendor/react*.production.min.js` | Lokal mitgelieferte React-Laufzeit |
| `vendor/babel.min.js` | Nur fuer den Entwicklungsmodus (`?dev=1`) und den Build |
| `api.php` | Installation, Datenzugriff, Einstellungen, Statistiken, Backup/Restore und iCal-Proxy |
| `jukebox.php` | Backend-Endpunkte der Jukebox: Album-Scan, Streaming, Cover, Upload, Verwaltung |
| `id3.php` | Metadaten aus Audiodateien lesen: ID3v2.3/2.4, ID3v1 (MP3) und M4A/MP4, samt eingebettetem Cover |
| `remote.php` | Externe Musikquellen: WebDAV, FTP/FTPS, HTTPS/S3 und gemountete Laufwerke (seit 1.5.0) |
| `manifest.json`, `sw.js` | PWA-Metadaten und Service-Worker-Verhalten |
| `.famtask_cfg.php` | Lokale Datenbankverbindung; wird nicht versioniert |

Die Anwendung ist in getrennte Repositories aufgeteilt: App, Web-Installer, Docker-Setup, Proxmox-Installer und diese Dokumentation. Das Docker-Setup mountet das App-Repository nach `/var/www/html`; Setup-Skripte erzeugen dort die lokale Datenbankkonfiguration.

## Frontend-Aufbau

`src/app.jsx` enthaelt die gesamte Anwendung. Der Build (`node tools/build_app.mjs`)
erzeugt daraus `app.js`; `index.html` laedt nur noch diese Datei. Ohne Build
springt die Seite auf den Quelltext-Weg: `?dev=1` an der URL (oder eine fehlende
`app.js`) laedt `vendor/babel.min.js` und uebersetzt `src/app.jsx` im Browser.
Der Build schreibt einen Inhalts-Hash als `?v=...` in `index.html`.

`FamTask` ist die einzige Top-Level-Komponente und haelt den gesamten Zustand;
Unterbildschirme werden ueber ein `ui`-Prop versorgt.

**Wichtige Konvention:** Komponenten, die eigene Eingabefelder enthalten, muessen
auf oberster Ebene definiert sein. Definiert eine Komponente ihre Eingabefelder
innerhalb des Render-Bodies von `FamTask` neu, wird sie bei jedem Rendern neu
erzeugt; der Cursor springt dann mitten im Tippen in ein anderes Feld zurueck.
Betroffen waren Modals, Admin-Bereich, Kind-Screen und der Musik-Player. Aus
demselben Grund liegt der Jukebox-Zustand in `FamTask` und nicht in
`AudioPlayerScreen`.

## remote.php

Eine eigene Schnittstelle neben `api.php`, weil sie andere Rechte und Grenzwerte
hat. Aktionen:

| Aktion | Zweck |
|---|---|
| `status` | Meldet, welche Protokolle und PHP-Extensions der Server unterstuetzt |
| `list` | Gespeicherte Quellen, ohne Passwoerter |
| `save`, `delete` | Quellen anlegen, aendern, loeschen |
| `browse`, `test` | Verbindung pruefen, Ordnerinhalt auflisten |
| `import` | Musik nach `my/audio/<Album>/` kopieren oder verlinken |

Zugangsdaten werden in `list` und `save` nie zurueckgegeben, sondern nur als
`hasPassword: true|false` gemeldet. Eingaben kommen sowohl als GET-Parameter als
auch als JSON-Body entgegen; PHP fuellt `$_REQUEST` bei `application/json`
allerdings nicht, deshalb liest die Datei den Body selbst.

Grenzwerte: 5000 Dateien, 2 GB gesamt und 1 GB je Datei pro Import, maximal
4 Unterordner-Ebenen.

## Backend-Aktionen

`api.php` routet unter anderem diese Aktionen ueber den Parameter `action`:

| Aktion | Zweck laut aktuellem Quellcode |
|---|---|
| `install`, `testdb`, `status`, `update` | Erstinstallation, Datenbanktest, Status und Migrationen |
| `register`, `verify` | Familien-/Geraete-Registrierung und Verifikation |
| `families`, `rename_family`, `delete_family` | Familienverwaltung |
| `settings`, `save_settings` | Einstellungen lesen und speichern |
| `track`, `stats` | Ereignisse erfassen und Statistiken abrufen |
| `ical` | Kalenderimport-Proxy |
| `backup`, `restore` | Daten sichern und wiederherstellen |
| `ytplaylist` | Playlist-bezogene Jukebox-Funktion |
| `admin` | Administrationsoberflaeche |

Dies ist eine Orientierung und keine vollstaendige API-Spezifikation. HTTP-Methode, Payload und Berechtigungen sind am jeweiligen Handler in `api.php` zu pruefen.

**Berechtigungen (Sicherheitsfix vom 2026-10-17):** Eine Admin-Anmeldung
verlangen `install`/`testdb` (nur bei bestehender Installation), `update`,
`settings`, `save_settings`, `families`, `rename_family`, `delete_family`,
`stats`, `backup`, `restore` und `ytplaylist`. Ohne Anmeldung erreichbar bleiben
`status`, `track`, `ical`, `verify`, `register` und der Datenzugriff ohne
`action`. `remote.php` verlangt ausser `status` ebenfalls eine Admin-Anmeldung.
Details siehe [DEPLOYMENT.md](DEPLOYMENT.md).

## Persistenz

Die App legt Tabellen bei der Installation beziehungsweise per Migration an. Aktuelle Tabellenfamilien im Quellcode:

- `famtask_data`: anwendungsspezifische Familiendaten als Schluessel/Wert-Daten
- `famtask_meta`: Metadaten zur App-Installation und Version
- `famtask_families`: Familiencodes und Familiennamen
- `famtask_stats`: Aufgaben-, Medienzeit- und Statistikereignisse
- `famtask_ical_events`: importierte Kalendertermine
- `famtask_remotes`: externe Musikquellen mit Zugangsdaten (seit 1.5.0)
- `famtask_admin_sessions`: Admin-Sitzungen als Zufalls-Token (seit Sicherheitsfix 2026-10-17)
- `famtask_auth_guard`: Fehlversuche pro IP fuer die Brute-Force-Bremse (seit Sicherheitsfix 2026-10-17)

`famtask_remotes` ist bewusst eine eigene Tabelle und nicht Teil von
`famtask_data`: die darin liegenden Zugangsdaten duerfen nicht ueber den
Synchronisations-Endpunkt in alle Geraete wandern.

Das Schema entwickelt sich ueber versionierte Migrationen in `api.php`. Vor Updates sollte ein Backup erstellt werden; Migrationen nicht manuell auslassen.

## Jukebox-Datenfluss

`my/audio/` ist die einzige Quelle der Wahrheit fuer Musik: Unterordner sind
Alben, lose Dateien werden als „Singles" gruppiert. `jukebox.php?action=tracks`
scannt dieses Verzeichnis. Deshalb importiert `remote.php` auch dorthin und
streamt nicht direkt vom Fernspeicher – sonst waeren bei jeder Wiedergabe
Zugangsdaten noetig und die Oberflaeche waere offline blind.

Der pro-Kind-Zustand liegt dagegen in den synchronisierten Familiendaten unter
`ft-jb`:
```
{"lastChild":"c1","kids":{"c1":{"lastSrc":"...","lastTitle":"...","order":["..."],"favs":["..."]}}}
```
Titel werden ueber ihre `src` identifiziert, weil die Laufzeit-ID beim Neuladen
neu erzeugt wird.

### Metadaten und Cover

`id3.php` liest Titel, Interpret, Album und ein eingebettetes Bild direkt aus
den Dateien (ID3v2.3/2.4 und ID3v1 fuer MP3, `ilst`-Atome fuer M4A/MP4) – ohne
fremde Bibliothek. Die Ergebnisse liegen in `audio/.tags-cache.json`, damit
nicht bei jedem Aufruf alle Dateien geoeffnet werden. Ein eingebettetes Bild
wird nach `audio/.covers/<Ordnername>.jpg` extrahiert, wenn dort noch keins
liegt; ein von Hand abgelegtes Cover bleibt unangetastet.

`action=tracks` liefert pro Titel `artist`, `albumTag` und `hasCover` mit.
`album` bleibt bewusst der Ordnername – er ist der Schluessel fuer Cover und
Album-Zuordnung.

Die Suche nach einem fehlenden Cover laeuft serverseitig in `jukebox.php`
(`action=cover`) und arbeitet nach festen Regeln:

1. Der Ordnername zaehlt nur dann als Album-Suchbegriff, wenn er wie ein Album
   aussieht (Ziffer oder mindestens zwei Woerter) oder ein ID3-Album ihn
   bestaetigt. "Mami" oder "Papi" sind Sammlungen, keine Alben.
2. Album-Treffer werden gegen Interpret **und** Albumname geprueft; der
   Interpret des Funds muss im gesuchten Namen aufgehen. Das schliesst fremde
   Alben gleichen Namens aus.
3. Fuer den Titel wird der Dateiname zerlegt (`Interpret - Titel`), Zusaetze wie
   `(Lyrics)` fallen weg. Der Interpret darf teilweise passen (Duette), der
   Titel muss vollstaendig im Fund stecken.
4. MusicBrainz wird nur mit einem aussagekraeftigen Suchbegriff befragt.
5. Liefert der Ordnername kein Album, sucht der Client den einzelnen Titel.

Passt nichts sicher, bleibt der Titel ohne Cover (`{"coverUrl":null}`) – lieber
ein farbiger Platzhalter als ein falsches Bild. Der Client speichert Treffer in
`localStorage` unter `ft-cover-cache`: Treffer dauerhaft, Fehlschlaege einen Tag.

## Container-Zusammensetzung

Die aktuelle Compose-Datei definiert drei Dienste:

- PHP/Apache auf Host-Port `8888` (Container-Port `80`)
- MariaDB `10.11` auf Host-Port `3307` (Container-Port `3306`)
- phpMyAdmin auf Host-Port `8081` (Container-Port `80`)

Das PHP-Image enthaelt seit 1.5.0 zusaetzlich die Extensions `curl` und `ftp`
fuer die externen Musikquellen. Fuer gemountete Freigaben ist in der Compose-Datei
ein auskommentierter Bind-Mount vorbereitet; die erlaubten Stammpfade stehen in
`FAMTASK_REMOTE_ROOTS`.

Die Datenbank liegt im benannten Volume `famtask_db_data`. Die Compose-Pfade sind relativ zur Datei im Repository-Stamm. Der bekannte Konflikt mit den Installer-Aufrufen ist in [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md) beschrieben.