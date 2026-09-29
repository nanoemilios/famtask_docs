# FamTask: Produkt und Funktionen

FamTask ist eine Familien-Aufgaben- und Medienzeit-App. Die Weboberflaeche laeuft im Browser; ein PHP-Backend speichert gemeinsame Familiendaten in MariaDB/MySQL.

## Aufgaben und Belohnungen

- Aufgaben fuer Kinder mit taeglicher Wiederholung, Faelligkeitszeiten und Bild-Upload
- Minuten-Belohnungen fuer erledigte Aufgaben
- Strafenkatalog mit Minuten-Verrechnung und Erfassung von Strafereignissen
- Zuweisung an einzelne Kinder oder mehrere Kinder

## Medienzeit

- Individuelle Limits je Kind und Medienart, beispielsweise TV, Tablet und Nintendo
- Tages- und Wochenlimits sowie erlaubte Wochentage
- Konfigurierbarer Uebertrag nicht verbrauchter Minuten
- Laufende Timer und Alarmton

## Kalender und Spiele

- Familienkalender mit woechentlichen, monatlichen und jaehrlichen Wiederholungen
- iCal-Import, ganztaegige Termine und Filter nach Kind
- 14 Lernspiele laut App-README, darunter Mathematik, Memory, Wort-Quiz, Geographie, Uhr-Lesen und Tier-Quiz
- Altersabhaengige Schwierigkeitsanpassung, Minuten-Belohnungen und Bestenliste

## Jukebox und Administration

- Musik- und Hoerspiel-Player mit Alben, seitenuebergreifendem Mini-Player und lokalen Audio-Uploads
- Cover-Art-Suche ueber MusicBrainz und iTunes
- Familienverwaltung und globale Einstellungen
- Admin-PIN-Schutz laut README
- Backup/Restore, Statistiken, Installation und Datenbank-Updates ueber `api.php`

## Plattform

- Progressive Web App (PWA) mit Manifest und Service Worker
- Mehrsprachige Oberflaeche; Sprachdateien liegen im App-Ordner `lang/`
- Responsive, fuer Mobilgeraete ausgelegte Oberflaeche
- Echtzeit-Synchronisierung zwischen Geraeten ueber die gemeinsame Datenbank

## Voraussetzungen

- PHP 8.0 oder neuer und PDO-MySQL
- MariaDB/MySQL
- Apache/Nginx oder ein passendes Container-Setup

Diese Funktionsliste fasst die Projektbeschreibung zusammen. Fuer die tatsaechlich ausgelieferte Version sind Quellcode und Release unter [famtask](https://github.com/nanoemilios/famtask) massgeblich.