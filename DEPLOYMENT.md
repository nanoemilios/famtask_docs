# Installation, Updates und Betrieb

## Installationswege

### Webspace

Der [Web-Installer](https://github.com/nanoemilios/famtask_installer) ist fuer Shared Hosting vorgesehen. Er prueft PHP-Voraussetzungen, laedt das App-ZIP aus dem FamTask-Release und bietet Installation beziehungsweise Update an. Voraussetzung laut Projekt: PHP 8.0+, ZipArchive, cURL oder `allow_url_fopen` sowie Schreibrechte.

Der Release `v1.4.0` enthaelt `famtask.zip`; der SHA-256-Wert ist im [Abgleichsbericht](WORKSPACE_COMPARISON.md) festgehalten. Die lokale Konfiguration `.famtask_cfg.php` und der Audio-Ordner sollen bei Updates erhalten bleiben.

> **Stand 1.5.0:** Der Quellcode im App-Repository ist auf `1.5.0`, das Release-Artefakt
> `famtask.zip` ist aber weiterhin `v1.4.0`. Der Installer laedt von
> `releases/latest/download/famtask.zip` und meldet deshalb noch `v1.4.0`, bis ein
> neues Release veroeffentlicht wird. Fuer die neuen Funktionen ist daher entweder
> ein neues Release noetig oder eine manuelle Installation aus dem Git-Repository.

### Manuelle App-Installation

1. [famtask](https://github.com/nanoemilios/famtask) auschecken.
2. `.famtask_cfg.php.example` als `.famtask_cfg.php` einrichten und Zugangsdaten einer MariaDB/MySQL-Datenbank eintragen.
3. `api.php?action=admin` im Browser aufrufen und die Administrations-/Installationsschritte durchlaufen.

Die Konfigurationsdatei enthaelt Geheimnisse und gehoert nicht in Git oder in dieses Doku-Repository.

### Externe Musikquellen einrichten

WebDAV, FTP/FTPS und HTTPS/S3 brauchen keine Shell-Zugriffe – Quelle unter
**Admin → Musikquellen** anlegen. Benoetigt werden die PHP-Extensions `curl` und
`ftp`, die im Docker-Image enthalten sind.

Fuer **SMB/CIFS und NFS** gilt: PHP kann Netzwerkdateisysteme nicht selbst
mounten. Der Host mountet, Docker reicht den Pfad nur durch:

```bash
# 1. Auf dem Host mounten
sudo mkdir -p /srv/famtask-music
sudo mount -t cifs //192.168.1.20/musik /srv/famtask-music -o username=NAS,password=GEHEIM,uid=0,gid=0,ro
ls /srv/famtask-music

# 2. In docker-compose.yml die auskommentierte Zeile aktivieren
#    - /srv/famtask-music:/mnt/music:ro

# 3. Neu starten
docker compose up -d

# 4. In FamTask: Art "Gemountetes Laufwerk", Pfad /mnt/music, Modus "Verlinken"
```

Erlaubte Stammpfade stehen in `FAMTASK_REMOTE_ROOTS`, Standard
`/mnt,/media,/srv,/net,/Volumes`. Ausserhalb davon liefert `remote.php` HTTP 403.

### Docker und Proxmox

Die vorgesehenen Projekte sind [famtask_docker](https://github.com/nanoemilios/famtask_docker) und [famtask_proxmox](https://github.com/nanoemilios/famtask_proxmox).

**Befehle** (Compose-Datei liegt im Repository-Stamm, `.env` unter `docker/`):

```bash
cd /opt/famtask_docker
cp docker/.env.example docker/.env     # Passwoerter anpassen
docker compose --env-file docker/.env up -d --build
bash scripts/setup.sh
```

`--env-file docker/.env` ist **noetig**: Compose sucht `.env` sonst im
Arbeitsverzeichnis und wuerde die in `docker/.env` hinterlegten Passwoerter
stillschweigend ignorieren.

Stand 2026-10-15: der frueher dokumentierte Widerspruch – die Scripts in
`famtask_proxmox` riefen die nicht existierende Datei `docker/docker-compose.yml`
auf – ist behoben. Ein Durchlauf auf einem echten Proxmox-Host steht weiterhin
aus. Historie: [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md).

Die Compose-Datei selbst definiert App `8888`, phpMyAdmin `8081` und Datenbank
`3307` als Host-Ports, jeweils ueber die Variablen `APP_PORT`, `PMA_PORT` und
`DB_PORT` aus `docker/.env` steuerbar.

## Betrieb und Sicherheit

- **FamTask hat keine echte Benutzerverwaltung.** Der Familiencode ist keine Authentifizierung; er steht im Klartext im lokalStorage jedes Geraets, und jeder, der ihn kennt, kann ueber `api.php` Daten lesen und schreiben. **Nicht ungeschuetzt ins Internet stellen** – nur im Heimnetz betreiben oder einen Reverse-Proxy mit Basic-Auth davorsetzen.

### Admin-Authentifizierung (Sicherheitsfix vom 2026-10-17)

- **Administrative Endpunkte verlangen jetzt eine Anmeldung.** Betroffen sind in
  `api.php`: `settings`, `save_settings`, `families`, `rename_family`,
  `delete_family`, `backup`, `restore`, `update`, `stats`, `ytplaylist` sowie
  `install`/`testdb` bei bestehender Installation. In `remote.php` gilt das fuer
  alle Aktionen ausser `status` (`list`, `save`, `delete`, `browse`, `test`,
  `import`). Vorher waren diese Endpunkte **ohne** Anmeldung erreichbar: der
  Admin-PIN war im Klartext auslesbar, Familien liessen sich loeschen und die
  Datenbank-Konfiguration ueberschreiben.
- **Sitzung statt vorhersagbarem Cookie.** Die Anmeldung nutzt den Admin-PIN und
  legt die Sitzung als Zufalls-Token in der neuen Tabelle
  `famtask_admin_sessions` ab (30 Tage gueltig). Das frueher verwendete, aus dem
  vierstelligen PIN berechenbare Cookie `sha1('ft_admin_' + PIN)` wird nur noch
  uebergangsweise akzeptiert.
- **Brute-Force-Bremse:** maximal 8 Fehlversuche pro IP in 15 Minuten (neue
  Tabelle `famtask_auth_guard`). Ein korrekter PIN funktioniert auch waehrend
  einer Sperre, damit sich niemand selbst ausschliesst.
- Nach dem Update einmal neu anmelden unter `api.php?action=admin`. Die App kann
  die Admin-PIN alternativ pro Anfrage im Header `X-Admin-Pin` senden; das nutzt
  sie fuer „Familie umbenennen/loeschen" und den Reiter Musikquellen.
- **Weiterhin ohne Anmeldung:** die Familiendaten selbst (`api.php` ohne
  `action`, zum Beispiel `api.php?fc=FAM001`) sowie `status`, `track`, `ical`,
  `verify` und `register` – diese braucht die App im normalen Betrieb. Wer den
  Familiencode kennt, kann diese Daten daher weiterhin lesen und schreiben.
- **Vor dem Update** ein Backup der Datenbank anlegen. Die beiden neuen Tabellen
  legt die App beim ersten Zugriff selbst an.
- Echte Passwoerter setzen; keine Beispielwerte oder Container-Defaults fuer produktive Systeme verwenden.
- Datenbank-Port `3307` und phpMyAdmin-Port `8081` nicht unnoetig oeffentlich freigeben. Zugriff auf App und Admin-Oberflaeche ueber HTTPS und geeignete Zugriffskontrollen absichern.
- `docker/.env`, `.famtask_cfg.php`, Datenbank-Dumps und persoenliche Familiendaten vertraulich behandeln.
- Zugangsdaten externer Musikquellen liegen ausschliesslich in `famtask_remotes` und werden von `remote.php` nie an den Client ausgeliefert. Sie gelten trotzdem als Geheimnis: das Volume `db_data` mit den Datenbank-Dumps schuetzen.
- Vor Updates und Migrationen ein konsistentes Backup der Datenbank sowie benoetigter Nutzdaten wie Audiodateien erstellen.
- `docker compose down -v` entfernt das benannte Datenbankvolume und kann Daten loeschen. Nur ausfuehren, wenn der Datenverlust beabsichtigt und ein Backup vorhanden ist.
- Release- und Versionsnummern bei App-Updates abgleichen. Der App-Quellcode meldet Version `1.5.0`; der Installer verwendet weiterhin `v1.4.0`, bis ein neues Release veroeffentlicht wird.

## Bekannte Einschraenkung

Diese Dokumentation korrigiert keine Skripte und stellt keinen erfolgreichen Docker-/Proxmox-Test dar. Der Containerstart sollte erst nach Behebung der in [WORKSPACE_COMPARISON.md](WORKSPACE_COMPARISON.md) genannten Pfad- und Env-Datei-Konflikte als verifiziert dokumentiert werden.