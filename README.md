# FamTask Dokumentation

Zentrale Dokumentation fuer FamTask. Diese Sammlung beschreibt die Anwendung, ihre technische Struktur, Installation und den Abgleich der getrennten GitHub-Repositories. Sie enthaelt absichtlich keine App-Kopien, Audiodateien, Konfigurationen oder Zugangsdaten.

## Einstieg

- [Produkt und Funktionsumfang](FEATURES.md)
- [Architektur und Datenmodell](ARCHITECTURE.md)
- [Installation, Updates und Betrieb](DEPLOYMENT.md)
- [Workspace- und Online-Abgleich](WORKSPACE_COMPARISON.md)

## Projekt-Repositories

| Repository | Zweck |
|---|---|
| [famtask](https://github.com/nanoemilios/famtask) | Web-App: PHP-Backend, HTML/React-Frontend, Sprachdateien und PWA-Dateien |
| [famtask_installer](https://github.com/nanoemilios/famtask_installer) | Web-Installer und Updater fuer Shared Hosting |
| [famtask_docker](https://github.com/nanoemilios/famtask_docker) | Docker Compose, PHP-Image, Datenbank und Setup-Skripte |
| [famtask_proxmox](https://github.com/nanoemilios/famtask_proxmox) | Installationsskripte fuer Proxmox, VM oder LXC |
| [famtask_docs](https://github.com/nanoemilios/famtask_docs) | Diese zentrale Dokumentation |

## Dokumentationsstand

Die vier lokalen Projekt-Repositories wurden am 2026-09-29 gegen ihre jeweiligen `origin/main`-Branches geprueft. Die Arbeitsstaende waren sauber und auf denselben Commits wie GitHub. Die konkrete Versions- und Pfadpruefung steht in [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md).

## Pflege

README-Dateien und Quellcode in den jeweiligen Projekt-Repositories sind die Quelle der Wahrheit fuer Implementierungsdetails. Bei Code- oder Installationsaenderungen bitte die betroffenen Seiten hier aktualisieren und Datum sowie Commit-IDs im Abgleichsbericht nachziehen. Keine Passwoerter, produktiven Konfigurationen, Datenbank-Dumps, privaten Familieninformationen oder Audioinhalte in dieses Repository laden.

Die Dokumentation beschreibt den geprueften Stand, nicht automatisch einen getesteten Produktionsbetrieb. Bekannte Abweichungen sind ausdruecklich als offen markiert.