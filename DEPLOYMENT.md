# Installation, Updates und Betrieb

## Installationswege

### Webspace

Der [Web-Installer](https://github.com/nanoemilios/famtask_installer) ist fuer Shared Hosting vorgesehen. Er prueft PHP-Voraussetzungen, laedt das App-ZIP aus dem FamTask-Release und bietet Installation beziehungsweise Update an. Voraussetzung laut Projekt: PHP 8.0+, ZipArchive, cURL oder `allow_url_fopen` sowie Schreibrechte.

Der Release `v1.4.0` enthaelt `famtask.zip`; der SHA-256-Wert ist im [Abgleichsbericht](WORKSPACE_COMPARISON.md) festgehalten. Die lokale Konfiguration `.famtask_cfg.php` und der Audio-Ordner sollen bei Updates erhalten bleiben.

### Manuelle App-Installation

1. [famtask](https://github.com/nanoemilios/famtask) auschecken.
2. `.famtask_cfg.php.example` als `.famtask_cfg.php` einrichten und Zugangsdaten einer MariaDB/MySQL-Datenbank eintragen.
3. `api.php?action=admin` im Browser aufrufen und die Administrations-/Installationsschritte durchlaufen.

Die Konfigurationsdatei enthaelt Geheimnisse und gehoert nicht in Git oder in dieses Doku-Repository.

### Docker und Proxmox

Die vorgesehenen Projekte sind [famtask_docker](https://github.com/nanoemilios/famtask_docker) und [famtask_proxmox](https://github.com/nanoemilios/famtask_proxmox). **Der dokumentierte Installationspfad ist aktuell blockiert:** Er referenziert eine nicht vorhandene `docker/docker-compose.yml`; ausserdem ist die `.env`-Ladequelle nicht konsistent. Details und betroffene Aufrufe: [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md).

Die Compose-Datei selbst definiert App `8888`, phpMyAdmin `8081` und Datenbank `3307` als Host-Ports. In der aktuellen Form sollten die Installer nicht als funktionierend vorausgesetzt werden, bevor die Pfade und Umgebungsvariablen korrigiert und getestet sind.

## Betrieb und Sicherheit

- Echte Passwoerter setzen; keine Beispielwerte oder Container-Defaults fuer produktive Systeme verwenden.
- Datenbank-Port `3307` und phpMyAdmin-Port `8081` nicht unnoetig oeffentlich freigeben. Zugriff auf App und Admin-Oberflaeche ueber HTTPS und geeignete Zugriffskontrollen absichern.
- `docker/.env`, `.famtask_cfg.php`, Datenbank-Dumps und persoenliche Familiendaten vertraulich behandeln.
- Vor Updates und Migrationen ein konsistentes Backup der Datenbank sowie benoetigter Nutzdaten wie Audiodateien erstellen.
- `docker compose down -v` entfernt das benannte Datenbankvolume und kann Daten loeschen. Nur ausfuehren, wenn der Datenverlust beabsichtigt und ein Backup vorhanden ist.
- Release- und Versionsnummern bei App-Updates abgleichen. Der App-Quellcode meldet Version `1.4`; der Installer verwendet `v1.4.0`.

## Bekannte Einschraenkung

Diese Dokumentation korrigiert keine Skripte und stellt keinen erfolgreichen Docker-/Proxmox-Test dar. Erst nach Behebung der in [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md) genannten Pfad- und Env-Datei-Konflikte sollte der Containerstart als verifiziert dokumentiert werden.