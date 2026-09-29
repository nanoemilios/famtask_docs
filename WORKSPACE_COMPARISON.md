# Workspace- und Online-Abgleich

Geprueft am **2026-09-29**. Die vier lokalen Projektordner wurden mit `git fetch origin --prune` aktualisiert und anschliessend mit `origin/main` verglichen. `HEAD...origin/main` ergab jeweils `0 0`; die Arbeitsbaeume waren sauber.

## Commit-Stand

| Projekt | Lokaler Commit = GitHub `main` | Betreff |
|---|---|---|
| [famtask](https://github.com/nanoemilios/famtask) | `26c34b9f11b83ecb2579625456a903a3eb846f76` | Fix Docker install instructions (reference famtask_proxmox) |
| [famtask_docker](https://github.com/nanoemilios/famtask_docker) | `3c72e8a32b1670c23365898c497de301082ce942` | Remove Proxmox installer (moved to famtask_proxmox), update README |
| [famtask_installer](https://github.com/nanoemilios/famtask_installer) | `810f5ea847a329919eb3524e743b7d466954aa97` | Simplify installer: single installer.php file, remove build.php and famtask.sql |
| [famtask_proxmox](https://github.com/nanoemilios/famtask_proxmox) | `0cc8bda51c0d1f0adb80a3e320e2e08427c840ac` | Add README.md |

Die lokalen README-Inhalte stimmen mit den oeffentlichen Repository-Seiten ueberein. Fuer die App ist `v1.4.0` der aktuelle Release-Tag; `main` steht auf demselben Commit. Der Release wurde am 2026-09-24 veroeffentlicht und enthaelt `famtask.zip` (SHA-256 `90f38dfab26a7e92e5c4f54741fda434928af31acf8d13e25d1d9e5846ec6f11`). Der Installer erwartet den Download unter `releases/latest/download/famtask.zip` und meldet `v1.4.0`.

Das GitHub-Repository `famtask_docs` war bei der Pruefung online leer; dieser Dokumentationssatz ist fuer seinen initialen Inhalt bestimmt.

## Offene Abweichungen: Docker und Proxmox

Die Docker-Anleitungen und Proxmox-Installationsskripte passen nicht zur versionierten Dateistruktur. Da lokale und Online-Commits identisch sind, besteht die Abweichung auch online.

- `famtask_docker` versioniert `docker-compose.yml` im Repository-Stamm, `db/init.sql` und `php/Dockerfile` auf Stamm-Ebene sowie `docker/.env.example`.
- Die Compose-Datei erwartet relative Pfade `./php` und `./db/init.sql`; sie muss daher vom Repository-Stamm aus verwendet werden.
- README und Proxmox-Skripte rufen dagegen `docker/docker-compose.yml` auf oder wechseln in den Ordner `docker/`. Diese Compose-Datei existiert nicht. Der Docker-/Proxmox-Installationspfad kann so nicht funktionieren.
- Das Setup-Skript liest `docker/.env`. Compose laedt standardmaessig `.env` aus dem Arbeitsverzeichnis; ein Start aus dem Repository-Stamm liest daher nicht automatisch `docker/.env`. Bei angepassten Datenbankwerten koennen Compose und App-Setup unterschiedliche Zugangsdaten verwenden.
- Die dokumentierten Update-, Log- und Stop-Kommandos referenzieren ebenfalls `docker/docker-compose.yml` und muessen korrigiert werden.

**Status:** In dieser Doku festgehalten, aber nicht in den Installations-Repositories behoben. Vor einer produktiven Docker-/Proxmox-Installation sollten Compose-Aufruf, Arbeitsverzeichnis und `.env`-Quelle gemeinsam korrigiert und getestet werden.

## Weitere Hinweise

- Die Compose-Datei verwendet MariaDB `10.11`, PHP `8.2` mit Apache und `phpmyadmin:latest`.
- Die README verweist auf die jeweils passende Aufteilung in App, Installer, Docker-Setup und Proxmox-Installer.
- In den Projekt-READMEs ist keine Softwarelizenz festgelegt. Eine Lizenz sollte vor einer Aussage ueber erlaubte Weiterverwendung festgelegt werden.