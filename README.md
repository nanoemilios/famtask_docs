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
| [famtask_landingpage](https://github.com/nanoemilios/famtask_landingpage) | Oeffentliche Projektseite |
| [famtask_docs](https://github.com/nanoemilios/famtask_docs) | Diese zentrale Dokumentation |

## Dokumentationsstand

Geprueft am **2026-10-15** fuer die sechs lokalen Projektordner in `famtask_split`.
Alle Arbeitsstaende waren zu Beginn sauber und identisch mit `origin/main`.

Bei dieser Pruefung zeigte sich, dass der Ordner `famtask_split/famtask` nicht
den aktuellen Entwicklungsstand enthielt: `APP_VERSION` stand dort auf `1.4`, waehrend
die Arbeitskopie bereits `1.5` enthielt. Der fehlende Stand wurde gespiegelt und
dokumentiert – Einzelheiten in [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md).

Der dokumentierte Stand betrifft die Quellstände, **keinen** getesteten
Produktionsbetrieb. Zwei Widersprueche im Docker-/Proxmox-Installationspfad sind
weiterhin offen und dort ausdruecklich als offen markiert.

## Pflege

README-Dateien und Quellcode in den jeweiligen Projekt-Repositories sind die Quelle der Wahrheit fuer Implementierungsdetails. Bei Code- oder Installationsaenderungen bitte die betroffenen Seiten hier aktualisieren und Datum sowie Commit-IDs im Abgleichsbericht nachziehen. Keine Passwoerter, produktiven Konfigurationen, Datenbank-Dumps, privaten Familieninformationen oder Audioinhalte in dieses Repository laden.

Die Dokumentation beschreibt den geprueften Stand, nicht automatisch einen getesteten Produktionsbetrieb. Bekannte Abweichungen sind ausdruecklich als offen markiert.