# Architektur und Datenmodell

## Komponenten

| Komponente | Aufgabe |
|---|---|
| `index.html`, `styles.css`, `i18n.js`, `lang/` | Browseroberflaeche, Styles und Uebersetzungen |
| `vendor/react*.production.min.js`, `vendor/babel.min.js` | Lokal mitgelieferte React-Laufzeit und JSX-Transpilierung |
| `api.php` | Installation, Datenzugriff, Einstellungen, Statistiken, Backup/Restore und iCal-Proxy |
| `jukebox.php` | Backend-Endpunkte der Jukebox: Album-Scan, Streaming, Cover, Upload, Verwaltung |
| `remote.php` | Externe Musikquellen: WebDAV, FTP/FTPS, HTTPS/S3 und gemountete Laufwerke (seit 1.5.0) |
| `manifest.json`, `sw.js` | PWA-Metadaten und Service-Worker-Verhalten |
| `.famtask_cfg.php` | Lokale Datenbankverbindung; wird nicht versioniert |

Die Anwendung ist in getrennte Repositories aufgeteilt: App, Web-Installer, Docker-Setup, Proxmox-Installer und diese Dokumentation. Das Docker-Setup mountet das App-Repository nach `/var/www/html`; Setup-Skripte erzeugen dort die lokale Datenbankkonfiguration.

## Frontend-Aufbau

`index.html` enthaelt die gesamte Anwendung als einzelne Babel-Transpilation ohne
Build-Schritt. `FamTask` ist die einzige Top-Level-Komponente und haelt den
gesamten Zustand; Unterbildschirme werden ueber ein `ui`-Prop versorgt.

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

## Persistenz

Die App legt Tabellen bei der Installation beziehungsweise per Migration an. Aktuelle Tabellenfamilien im Quellcode:

- `famtask_data`: anwendungsspezifische Familiendaten als Schluessel/Wert-Daten
- `famtask_meta`: Metadaten zur App-Installation und Version
- `famtask_families`: Familiencodes und Familiennamen
- `famtask_stats`: Aufgaben-, Medienzeit- und Statistikereignisse
- `famtask_ical_events`: importierte Kalendertermine
- `famtask_remotes`: externe Musikquellen mit Zugangsdaten (seit 1.5.0)

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