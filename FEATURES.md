# FamTask: Produkt und Funktionen

FamTask ist eine Familien-Aufgaben- und Medienzeit-App. Die Weboberflaeche laeuft im Browser; ein PHP-Backend speichert gemeinsame Familiendaten in MariaDB/MySQL.

Stand dieses Dokuments: **Version 1.5.0**.

## Aufgaben und Belohnungen

- Aufgaben fuer Kinder mit taeglicher Wiederholung, Faelligkeitszeiten und Bild-Upload
- Minuten-Belohnungen fuer erledigte Aufgaben
- Strafenkatalog mit Minuten-Verrechnung und Erfassung von Strafereignissen
- Zuweisung an einzelne Kinder oder mehrere Kinder

## Sackgeld

Ab Version 1.5.0 verwalten Kinder zusaetzlich ein Taschengeld in Schweizer Franken.

- Guthaben, Sparkonto und Wochenverguetung je Kind
- Betraege werden in Rappen als ganzzahlige Werte gespeichert, nicht als Gleitkommazahl; dadurch entstehen keine Rundungsfehler
- Negative Guthaben sind zulaessig; Kinder koennen in die Schuld geraten
- Kontoauszug ueber alle Buchungen
- Kaufbuchungen: Guthaben wird gegen Medienzeit eingetauscht
- Ueber die Admin-Oberflaeche einstellbare Wochenbetraege und Spiel-Belohnungen

## Medienzeit

- Individuelle Limits je Kind und Medienart, beispielsweise TV, Tablet und Nintendo
- Tages- und Wochenlimits sowie erlaubte Wochentage
- Konfigurierbarer Uebertrag nicht verbrauchter Minuten
- Laufende Timer und Alarmton
- Kauf von Medienzeit gegen Taschengeld

## Kalender und Spiele

- Familienkalender mit woechentlichen, monatlichen und jaehrlichen Wiederholungen
- iCal-Import, ganztaegige Termine und Filter nach Kind
- 14 Lernspiele laut App-README, darunter Mathematik, Memory, Wort-Quiz, Geographie, Uhr-Lesen und Tier-Quiz
- Altersabhaengige Schwierigkeitsanpassung, Minuten-Belohnungen und Bestenliste
- Spiel-Belohnungen koennen in Freizeitminuten oder Taschengeld ausgezahlt werden

## Jukebox

Ab Version 1.5.0 ist der Player pro Kind personalisiert. Die Musikbibliothek bleibt
gemeinsam, jeder Knabe hat aber einen eigenen Player-Zustand.

- **Kind-Auswahl** oben; die Farbe des aktiven Kindes wird zur Akzentfarbe der ganzen Ansicht
- **Album-Cover-Carousel** waagerecht mit Scroll-Snap
- **Schlanke Titelliste** statt der bisherigen Task-Kacheln
- **Schwebender Mini-Player** mit Cover, Fortschritt und Vor/Zurueck
- **Favoriten je Kind** ueber ein Herz an jedem Titel, gebuendelt im Reiter „Meine Lieder"
- Je Kind gespeichert: letzter Titel, letztes Album, eigene Album-Reihenfolge, Favoriten
- Cover-Art-Suche ueber iTunes, Deezer und MusicBrainz (Cover Art Archive); ein in
  der Datei eingebettetes Bild hat Vorrang
- Titel, Interpret und Album werden aus den Datei-Tags gelesen (ID3v2.3/2.4,
  ID3v1 fuer MP3, iTunes-Atome fuer M4A/MP4). Fehlen die Tags, gelten Dateiname
  und Ordnername.
- Erlaubte Formate: mp3, wav, ogg, m4a, flac, aac, opus

Der Jukebox-Zustand liegt serverseitig im Schluessel `ft-jb` und wird wie die
uebrigen Familiendaten zwischen den Geraeten synchronisiert.

## Externe Musikquellen

Ab Version 1.5.0 laesst sich Musik von ausserhalb holen. Verwaltet wird das
unter **Admin → Musikquellen**, technisch ueber `remote.php`.

| Art | Funktioniert ohne Mount | Benoetigt |
|---|---|---|
| WebDAV | ja | `curl` |
| FTP / FTPS | ja | `ftp` |
| HTTPS / S3 | ja | `curl` |
| Gemountetes Laufwerk (SMB, NFS) | nein | Host-Mount, `symlink` |

- Zwei Importmodi: **Kopieren** in `my/audio/<Album>/` (danach offline verfuegbar) oder **Verlinken** als Symlink (kein Speicherbedarf, Aenderungen wirken sofort)
- Wiederholte Importe ueberspringen Dateien unveraenderter Groesse statt Dubletten anzulegen
- Zugangsdaten liegen ausschliesslich in der Datenbank und werden nie an den Client zurueckgegeben
- Abgewiesen werden `file://`, Zugangsdaten in der URL, Cloud-Metadaten-Endpunkte sowie lokale Pfade ausserhalb der konfigurierten Stammverzeichnisse

**Technische Grenze:** PHP kann weder SMB/CIFS noch NFS selbst mounten. Fuer
diese Freigaben mountet der Host, Docker reicht den Pfad nur durch. WebDAV, FTP
und HTTPS spricht PHP dagegen selbst.

## Administration

- Familienverwaltung und globale Einstellungen
- Admin-PIN-Schutz serverseitig: administrative `api.php`-Aktionen und `remote.php` verlangen eine Anmeldung (Session-Cookie oder Header `X-Admin-Pin`) – siehe [DEPLOYMENT.md](DEPLOYMENT.md)
- Reiter „Musikquellen" mit Verbindungs-Test, Ordner-Ansicht und Import
- Backup/Restore, Statistiken, Installation und Datenbank-Updates ueber `api.php`

## Plattform

- Progressive Web App (PWA) mit Manifest und Service Worker
- Mehrsprachige Oberflaeche; Sprachdateien liegen im App-Ordner `lang/`
- Responsive, fuer Mobilgeraete ausgelegte Oberflaeche
- Echtzeit-Synchronisierung zwischen Geraeten ueber die gemeinsame Datenbank

## Voraussetzungen

- PHP 8.0 oder neuer und PDO-MySQL
- `curl` und `ftp` fuer externe Musikquellen
- MariaDB/MySQL
- Apache/Nginx oder ein passendes Container-Setup

## Bekannte Einschraenkungen

- `AudioPlayerScreen` und die Spiele werden bei jedem Rendern neu erzeugt. Das ist
  performanceunkritisch, zwingt aber UI-Zustand in `FamTask` statt in die
  Komponente zu legen.
- Ein Ordnername dient nur dann als Album-Suchbegriff, wenn er wie ein Album
  aussieht (Ziffer oder mindestens zwei Woerter). Enthaelt ein Ordner nur einen
  Namen wie `Mami` oder `Papi`, wird stattdessen der Titel gesucht.
- Ohne Album-Cover wird pro Titel gesucht; der Dateiname wird dafuer in
  `Interpret - Titel` zerlegt und Zusaetze wie `(Lyrics)` werden entfernt.
- Bleibt ein Treffer unklar, zeigt FamTask lieber einen farbigen Platzhalter als
  ein falsches Bild. Bei sehr allgemeinen Namen ("Universum") oder erfundenen
  Angaben bleibt es deshalb ohne Cover.
- S3 wird nur ueber direkte oder presigned HTTPS-URLs unterstuetzt, nicht ueber
  SigV4-Signierung mit Bucket und Access Key.
- Ein automatischer Re-Sync externer Quellen (Cronjob) fehlt; Importe laufen von Hand.
