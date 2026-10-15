# Workspace- und Online-Abgleich

Geprueft am **2026-10-15**. Die sechs lokalen Projektordner in `famtask_split` wurden mit `git fetch origin` aktualisiert und mit `origin/main` verglichen. Vor Beginn der Arbeit ergab `HEAD...origin/main` fuer alle Repositories `0 0`; die Arbeitsbaeume waren sauber. Es bestand daher kein Risiko, lokale Commits zu ueberschreiben.

## Commit-Stand vor dieser Aktualisierung

| Projekt | Lokaler Commit = GitHub `main` | Betreff |
|---|---|---|
| [famtask](https://github.com/nanoemilios/famtask) | `26c34b9f` | Fix Docker install instructions (reference famtask_proxmox) |
| [famtask_docker](https://github.com/nanoemilios/famtask_docker) | `3c72e8a3` | Remove Proxmox installer (moved to famtask_proxmox), update README |
| [famtask_docs](https://github.com/nanoemilios/famtask_docs) | `07f0c9ac` | Dokumentationssatz |
| [famtask_installer](https://github.com/nanoemilios/famtask_installer) | `810f5ea8` | Simplify installer: single installer.php file, remove build.php and famtask.sql |
| [famtask_landingpage](https://github.com/nanoemilios/famtask_landingpage) | `c1de3d95` | Landingpage |
| [famtask_proxmox](https://github.com/nanoemilios/famtask_proxmox) | `0cc8bda5` | Add README.md |

## Inhaltlicher Abgleich: App-Code lag zurueck

Der Ordner `famtask_split/famtask` enthielt **nicht** den aktuellen Entwicklungsstand. Beim Vergleich mit der Arbeitskopie fehlten saemtliche Neuerungen der Version 1.5.0:

- Taschengeld (CHF) in Rappen, Sparkonto, Wochenverguetung, Kaufbuchungen
- Neugestaltung des Jukebox-Players mit Kind-Auswahl und Album-Carousel
- Favoriten je Kind
- Pro-Kind-Jukebox-Zustand im Schluessel `ft-jb`
- Externe Musikquellen samt `remote.php`
- Korrektur des Eingabefokus-Fehlers
- Korrektur des Cover-Autoloadens

`APP_VERSION` stand in `famtask_split/famtask` auf `1.4`, in der Arbeitskopie auf `1.5`.

Vor dem Spiegeln wurde deshalb abgeglichen, welche Dateien betroffen sind: 22
geaenderte, 1 neue (`remote.php`), 17 unveraenderte. Der Ordner `audio/` war in
beiden Staenden identisch.

**Nicht** uebernommen wurden die Dateien `.famtask_cfg.php` (enthält
Datenbankpasswoerter und ist ohnehin ignoriert) sowie `README.md`, `.gitignore`
und `.famtask_cfg.php.example` (repo-spezifisch). Nach dem Spiegeln wurden
Hashes aller gespiegelten Dateien verglichen und `api.php`, `jukebox.php` sowie
`remote.php` mit `php -l` geprueft, `index.html` mit dem Babel-Parser.

## Docker-Setup: nur ergaenzt, nicht ersetzt

Beim Docker-Repository lag die Lage umgekehrt: `scripts/setup.sh` und
`scripts/setup.ps1` in `famtask_split/famtask_docker` sind **neuer und
vollstaendiger** als die der Arbeitskopie (1420 gegen 621 Byte beziehungsweise
2205 gegen 817 Byte). Sie klonen das App-Repository, laden `docker/.env` und
verwenden Variablen. Ein blindes Spiegeln haette diese Funktionen geloescht.

Deshalb wurden aus dem Docker-Setup nur die echten Aenderungen uebernommen:

- `php/Dockerfile`: `curl` und `ftp` ergaenzt (rein additiv, geprueft per Diff)
- `docker-compose.yml`: Bind-Mount-Beispiele fuer SMB/NFS und `FAMTASK_REMOTE_ROOTS` ergaenzt, **wobei der repo-eigene Pfad `- ./my:/var/www/html` erhalten blieb**. Die Arbeitskopie nutzt `../my`, weil dort das App-Repository neben dem Docker-Ordner liegt; im Docker-Repository klont `setup.sh` das App-Repo nach `./my`.

Die Setup-Skripte blieben unveraendert.

## Behobene Abweichung: Docker und Proxmox

**Diese Punkte sind seit dem 2026-10-15 behoben.** Vorher bestanden sie; die damalige
Fassung ist unter "Frueher offene Abweichungen" weiter unten festgehalten.

- Die Compose-Datei liegt im Repository-Stamm. Die Scripts in `famtask_proxmox`
  riefen `docker/docker-compose.yml` auf und wechselten in den Ordner `docker/`;
  beides ist jetzt entfernt. Compose wird aus dem Repository-Stamm gestartet.
- `scripts/setup.sh` liest `docker/.env`, Compose suchte `.env` im Arbeitsverzeichnis.
  Die Passwörter wären stillschweigend ignoriert worden. Jetzt wird die Datei
  explizit über `--env-file docker/.env` übergeben – in den Proxmox-Scripts, im
  Docker-README und in der Zusammenfassung der Installer-Ausgabe.
- `APP_PORT`, `PMA_PORT` und `DB_PORT` standen in `docker/.env`, wurden von
  `docker-compose.yml` aber nicht ausgewertet: die Host-Ports waren hart codiert.
  Ein Nutzer konnte den Wert ändern, ohne dass etwas passierte, während die
  Abschlussmeldung trotzdem den neuen Port nannte. `docker-compose.yml` wertet die
  Variablen nun aus.
- `proxmox-install.sh` überschrieb bei jedem Lauf die `.env` und vergab neue
  Passwörter. Bei einem Update hätte die App den Anschluss an die Datenbank
  verloren. Vorhandene `.env` wird jetzt nicht mehr überschrieben.

Geprüft mit `bash -n` (Syntax), einem Vergleich aller Compose-Aufrufe und einem
Test, der das Release-ZIP entpackt und die App daraus startet. **Ein Durchlauf auf
einem echten Proxmox-Host steht weiterhin aus.**

## Frueher offene Abweichungen (Stand vor dem 2026-10-15)

- `famtask_docker` versioniert `docker-compose.yml` im Repository-Stamm. Der Pfad
  `docker/docker-compose.yml` existiert nicht.
- Compose erwartet relative Pfade `./php` und `./db/init.sql` und muss daher vom
  Repository-Stamm aus verwendet werden.
- `scripts/setup.sh` liest `docker/.env` (`PROJECT_ROOT/docker/.env`). Compose laedt
  standardmaessig `.env` aus dem Arbeitsverzeichnis.

Betroffene Aufrufe, damals gezaehlt:

| Datei | Treffer auf `docker/docker-compose.yml` |
|---|---|
| `famtask_docker/README.md` | 1 |
| `famtask_proxmox/proxmox-install.sh` | 3 |
| `famtask_proxmox/proxmox-install-oneliner.sh` | 2 |
| `famtask_proxmox/README.md` | 5 |
| `famtask_proxmox/PROXMOX_INSTALL.md` | 5 |
| `famtask_docs/DEPLOYMENT.md` | 1 |

## Versionsstand

- App-Quellcode: `APP_VERSION` `1.5.0`, `manifest.json` `1.5.0`
- App-README und CHANGELOG auf 1.5.0
- Release-Artefakt fuer 1.5.0 gebaut, aber **noch nicht auf GitHub veroeffentlicht**:

  | Eigenschaft | Wert |
  |---|---|
  | Datei | `famtask-1.5.0.zip` |
  | Groesse | 1 134 131 Bytes (1,08 MB) |
  | SHA-256 | `820a97609f22e13a03b0d62982c634a05ba7bc5288c42febca31f8e49313979d` |
  | Eintraege | 40 (gegenueber 39 bei v1.4.0) |
  | Unterschied | ausschliesslich `remote.php` neu |
  | Quelle | `famtask`-Repository, Stand der Commits vom 2026-10-15 |

  Gegenueber dem Release `v1.4.0` (SHA-256
  `90f38dfab26a7e92e5c4f54741fda434928af31acf8d13e25d1d9e5846ec6f11`) enthaelt
  das neue Archiv exakt dieselbe Dateiliste plus `remote.php`. Keine Audio-Dateien,
  keine Konfiguration, keine Dokumentation.

  Damit der [Web-Installer](https://github.com/nanoemilios/famtask_installer) 1.5.0
  ausliefert, muss das Archiv als `famtask.zip` an ein Release-Tag `v1.5.0`
  gehaengt werden – der Installer laedt von `releases/latest/download/famtask.zip`.
  `APP_VERSION` im Installer steht bereits auf `v1.5.0`.

  Geprueft wurde das Archiv durch Entpacken, PHP-Syntaxpruefung aller drei
  PHP-Dateien, Laden aller 19 Sprachdateien und einen Start der App aus dem
  entpackten Stand (`api.php` meldet `app_version` `1.5.0`).

## Weitere Hinweise

- Die Compose-Datei verwendet MariaDB `10.11`, PHP `8.2` mit Apache und `phpmyadmin:latest`. Das PHP-Image enthaelt seit dieser Aktualisierung zusaetzlich `curl` und `ftp`.
- Die neuen Schluessel aus Version 1.5.0 existieren bisher **nur in `lang/de.js` und `lang/en.js`**. Damit keine rohen Schluessel wie `audio.for_child` in der Oberflaeche erscheinen, laedt `i18n.js` Deutsch zuerst als Rueckfallbasis; fehlt eine Uebersetzung, wird der deutsche Text angezeigt. Eine vollstaendige Uebersetzung der 17 uebrigen Sprachen steht noch aus.
- In den Projekt-READMEs ist weiterhin keine Softwarelizenz festgelegt. Eine Lizenz sollte vor einer Aussage ueber erlaubte Weiterverwendung festgelegt werden.
- FamTask hat keine echte Benutzerverwaltung. Der Familiencode ist keine Authentifizierung. Das ist in den READMEs nun ausdruecklich vermerkt.
