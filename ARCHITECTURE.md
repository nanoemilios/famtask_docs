# Architektur und Datenmodell

## Komponenten

| Komponente | Aufgabe |
|---|---|
| `index.html`, `styles.css`, `i18n.js`, `lang/` | Browseroberflaeche, Styles und Uebersetzungen |
| `vendor/react*.production.min.js`, `vendor/babel.min.js` | Lokal mitgelieferte React-Laufzeit und JSX-Transpilierung |
| `api.php` | Installation, Datenzugriff, Einstellungen, Statistiken, Backup/Restore und iCal-Proxy |
| `jukebox.php` | Backend-Endpunkte der Jukebox |
| `manifest.json`, `sw.js` | PWA-Metadaten und Service-Worker-Verhalten |
| `.famtask_cfg.php` | Lokale Datenbankverbindung; wird nicht versioniert |

Die Anwendung ist in getrennte Repositories aufgeteilt: App, Web-Installer, Docker-Setup, Proxmox-Installer und diese Dokumentation. Das Docker-Setup mountet das App-Repository nach `/var/www/html`; Setup-Skripte erzeugen dort die lokale Datenbankkonfiguration.

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

Das Schema entwickelt sich ueber versionierte Migrationen in `api.php`. Vor Updates sollte ein Backup erstellt werden; Migrationen nicht manuell auslassen.

## Container-Zusammensetzung

Die aktuelle Compose-Datei definiert drei Dienste:

- PHP/Apache auf Host-Port `8888` (Container-Port `80`)
- MariaDB `10.11` auf Host-Port `3307` (Container-Port `3306`)
- phpMyAdmin auf Host-Port `8081` (Container-Port `80`)

Die Datenbank liegt im benannten Volume `famtask_db_data`. Die Compose-Pfade sind relativ zur Datei im Repository-Stamm. Der bekannte Konflikt mit den Installer-Aufrufen ist in [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md) beschrieben.