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

## Offene Abweichungen: Docker und Proxmox

**Diese Punkte bestehen weiterhin.** Sie wurden am 2026-10-15 erneut am Dateisystem geprueft und nicht behoben.

- `famtask_docker` versioniert `docker-compose.yml` im Repository-Stamm. Der Pfad `docker/docker-compose.yml` existiert nicht.
- Compose erwartet relative Pfade `./php` und `./db/init.sql` und muss daher vom Repository-Stamm aus verwendet werden.
- `scripts/setup.sh` liest `docker/.env` (`PROJECT_ROOT/docker/.env`). Compose laedt standardmaessig `.env` aus dem Arbeitsverzeichnis; ein Start aus dem Repository-Stamm liest `docker/.env` daher nicht automatisch. Bei angepassten Datenbankwerten koennen Compose und App-Setup unterschiedliche Zugangsdaten verwenden.

Betroffene Aufrufe, geprueft am 2026-10-15:

| Datei | Treffer auf `docker/docker-compose.yml` |
|---|---|
| `famtask_docker/README.md` | 1 (korrigiert, Befehl ohne `-f`) |
| `famtask_proxmox/proxmox-install.sh` | 3 |
| `famtask_proxmox/proxmox-install-oneliner.sh` | 2 |
| `famtask_proxmox/README.md` | 5 |
| `famtask_proxmox/PROXMOX_INSTALL.md` | 5 |
| `famtask_docs/DEPLOYMENT.md` | 1 (korrigiert) |

**Status:** In `famtask_docker/README.md` und `DEPLOYMENT.md` ist jetzt vermerkt,
dass die Skripte in `famtask_proxmox` weiterhin den falschen Pfad verwenden. Die
Skripte selbst sind **nicht** korrigiert worden. Vor einer produktiven
Docker-/Proxmox-Installation sollten Compose-Aufruf, Arbeitsverzeichnis und
`.env`-Quelle gemeinsam in `famtask_proxmox` und `famtask_docker/scripts`
angeglichen und getestet werden.

## Versionsstand

- App-Quellcode: `APP_VERSION` `1.5.0`, `manifest.json` `1.5.0`
- App-README und CHANGELOG wurden auf 1.5.0 gebracht
- Der Release-Tag `v1.4.0` und das Release-Artefakt `famtask.zip` sind **nicht**
  neu gebaut worden. Der [Web-Installer](https://github.com/nanoemilios/famtask_installer)
  laedt weiterhin von `releases/latest/download/famtask.zip` und meldet daher
  noch `v1.4.0`, bis ein neues Release veroeffentlicht wird.

## Weitere Hinweise

- Die Compose-Datei verwendet MariaDB `10.11`, PHP `8.2` mit Apache und `phpmyadmin:latest`. Das PHP-Image enthaelt seit dieser Aktualisierung zusaetzlich `curl` und `ftp`.
- Die neuen Schluessel aus Version 1.5.0 existieren bisher **nur in `lang/de.js` und `lang/en.js`**. Damit keine rohen Schluessel wie `audio.for_child` in der Oberflaeche erscheinen, laedt `i18n.js` Deutsch zuerst als Rueckfallbasis; fehlt eine Uebersetzung, wird der deutsche Text angezeigt. Eine vollstaendige Uebersetzung der 17 uebrigen Sprachen steht noch aus.
- In den Projekt-READMEs ist weiterhin keine Softwarelizenz festgelegt. Eine Lizenz sollte vor einer Aussage ueber erlaubte Weiterverwendung festgelegt werden.
- FamTask hat keine echte Benutzerverwaltung. Der Familiencode ist keine Authentifizierung. Das ist in den READMEs nun ausdruecklich vermerkt.
