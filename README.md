# Gemeinde-Gmunden

## Inhaltsverzeichnis
- [Projektübersicht](#projektübersicht)
- [Technischer Stack](#technischer-stack)
- [Installation](#installation)
- [Verzeichnisstruktur](#verzeichnisstruktur)
- [Datenquellen und Import](#datenquellen-und-import)
- [Datenbankschema](#datenbankschema)
- [Schnittstellen (APIs, Skripte, OCR)](#schnittstellen-apis-skripte-ocr)
- [Hauptmenü und Untermenüs](#hauptmenü-und-untermenüs)
- [OCR-Agent](#ocr-agent)
- [Backup & Restore](#backup--restore)
- [Web-Applikation (UI im Browser)](#web-applikation-ui-im-browser)
- [Wichtige Skripte im Detail](#wichtige-skripte-im-detail)
- [Arbeitsabläufe](#arbeitsabläufe)
- [Hinweise und Best Practices](#hinweise-und-best-practices)

## Projektübersicht
Das Projekt **Gmunden Datenbank Transparenz** ermöglicht es, Finanz- und Verwaltungsdaten der Stadt Gmunden ab dem Jahr 2004 (Schwerpunkt ab 2010) zu sammeln, zu strukturieren und öffentlich bereitzustellen. Die Daten werden in einer MongoDB-Datenbank gespeichert. PDFs, CSV-Dateien und andere Quellen werden automatisiert importiert, OCR-verarbeitet und über ein browserbasiertes UI zugänglich gemacht.

**Ziele:**
- Transparenz für Bürger und Gemeinderat.
- Automatisierte Verarbeitung von Verwaltungsdokumenten.
- Benutzerfreundliche Darstellung als Web-Applikation.

## Technischer Stack
- macOS (Apple Silicon M3 Max getestet)
- Colima (VM für Docker, `vmType=vz`)
- Docker & Docker Compose
- MongoDB 7.0 als zentrale Datenbank
- Zsh- und Bash-Skripte zur Automatisierung
- OCR-Agent (Python 3.11, Tesseract, qpdf, Poppler, Ghostscript)
- Node.js Backend (Express, API-Endpunkte für Datenzugriff)
- React Frontend (Web-UI zur Darstellung im Browser)

## Installation
1. Abhängigkeiten installieren (z. B. mit Homebrew): `colima`, `docker`, `docker-compose`, `qpdf`, `poppler`, `tesseract`, `imagemagick`, `ghostscript`, `python@3.11`, `node`, `npm`.
2. Projekt initialisieren mit `pack.zsh`.
3. Bootstrap der Datenbank mit `tools/bootstrap.zsh`.
4. Menü starten mit `menu.zsh`.
5. Web-Applikation starten (Backend + Frontend).

## Verzeichnisstruktur
```
gmunden-mongo-stack
├── compose/ – Docker-Compose-Konfiguration
├── backups/ – Backup & Restore-Skripte
├── cron/ – Automatische Backups (Launchd/Cron)
├── tools/ – Import- und Prüfskripte
├── ocr-agent/ – OCR-Dienst (Docker + Python)
├── input/ – Ordner für neue Dateien (PDFs, CSVs)
├── exports/ – Exporte (Backups, Dumps)
├── reports/ – Reports und Prüfungen
├── web/ – Web-Applikation (React + Node.js)
├── menu.zsh – Steuerungsmenü
└── README.md – Dokumentation
```

## Datenquellen und Import
**Primäre Quellen:**
- [data.gv.at](https://www.data.gv.at/) (Open Data Österreich)
- Land Oberösterreich (OGD2-Citi, Finanzgebarung)
- Statistik Austria (Open Data Katalog)
- Gemeinde Gmunden (Amtsblatt, Sitzungsprotokolle, Jahresabschlüsse)

**Fallback-Quellen:**
- Gespeicherte PDFs aus Amtsblatt-Archiven
- Landesarchiv Oberösterreich (gescannte Dokumente)
- Transparenzinitiativen (z. B. Transparency International Austria)

**Import-Szenarien:**
- CSV/JSON → Direktimport in MongoDB Collections.
- PDF mit eingebettetem Text → Extraktion mit `pdfminer` / `pdftotext`.
- PDF mit Bildseiten → OCR mit Tesseract.
- Gesperrte PDF → Entsperren mit `qpdf` oder Ghostscript.

## Datenbankschema
**Zentrale Datenbank:** `appdb`

**Collections:**
- `jahre`: Übersicht aller Jahre mit Status (verfügbar, teilweise, nicht verfügbar).
- `files`: Metadaten und Inhalte hochgeladener Dokumente (GridFS).
- `ocr_documents`: Ergebnisse der OCR-Verarbeitung.
- `meta`: Importierte Metadaten (z. B. Quellenangaben).
- `reports`: Reports und Prüfungen.

## Schnittstellen (APIs, Skripte, OCR)
**REST-API (Node.js Express im Verzeichnis `web/backend`):**
- `GET /api/jahre` → Liste aller Jahre und Status.
- `GET /api/docs/:jahr` → Alle Dokumente eines Jahres.
- `GET /api/search?q=…` → Volltextsuche in OCR-Daten.
- `GET /api/reports` → Generierte Reports.

**Zsh-Skripte im Verzeichnis `tools/` für:**
- Import CSV (data.gv.at, OÖ API).
- Import PDF (einzeln oder stapelweise).
- Synchronisierung Jahresstatus.

**OCR-Agent (Python, läuft als Docker-Service):**
- Überwacht `input/` auf neue Dateien.
- Führt OCR, Entsperrung und Text-Extraktion aus.
- Speichert Ergebnisse in Mongo.

## Hauptmenü und Untermenüs
Das Menü ist farbig und in Kategorien gegliedert:
- **Basis:** Server starten/stoppen, Status, Logs.
- **Backup & Restore:** Dateien, Dump, Full.
- **Tools:** Grunddaten importieren, Bootstrap, Jahresabgleich, Finanzdaten, PDF-Import.
- **PDF-Werkzeuge:** Bilder extrahieren, PDFs entsperren.
- **System:** Jahresstatus synchronisieren, Repack ZIP.
- **OCR-Agent:** Starten, Logs ansehen.
- **Cron/Exports/Reports:** Anzeigen und ausführen.

## OCR-Agent
- Entsperrt gesicherte PDFs automatisch.
- OCR mit Tesseract für gescannte Dokumente.
- Hash-basierte Erkennung, um doppelte Dokumente zu vermeiden.
- Bewertet Textqualität mit Keyword-Scoring.
- Ergebnisse werden in Mongo gespeichert.

## Backup & Restore
Drei Modi:
1. **Dateien:** Nur Projektordner.
2. **Normal:** Dateien + MongoDump.
3. **Voll:** Dateien, MongoDump, Config, Docker-Images.

Automatische Backups: wöchentlich Montag 03:00 über Launchd-Job.

## Web-Applikation (UI im Browser)
Das Verzeichnis `web/` enthält eine vollständige Anwendung:
- **Backend (Node.js + Express):** REST-API für Datenbankzugriff.
- **Frontend (React):** Benutzeroberfläche.

**Funktionen der UI:**
- Dashboard: Überblick über verfügbare, teilweise und nicht verfügbare Jahre.
- Jahrgangsansicht: Liste aller Dokumente eines Jahres.
- Dokumenten-Viewer: Vorschau von PDFs, OCR-Textdarstellung.
- Suche: Volltextsuche über alle importierten Dokumente.
- Download: Export von Daten und Dokumenten.
- Reports: Tabellen und Diagramme (z. B. Einnahmen/Ausgaben).

Benutzer können die Daten direkt im Browser durchsuchen, filtern und herunterladen.

## Wichtige Skripte im Detail
- `pack.zsh`: Setzt das Projekt auf, erstellt Docker-Umgebung, startet Mongo.
- `bootstrap.zsh`: Initialisiert Collections.
- `menu.zsh`: Hauptmenü zur Steuerung aller Funktionen.
- `ingest_file.zsh`: Einzeldateien (PDF/PNG/JPG) importieren.
- `tools/*`: Hilfsskripte für Jahresabgleich, CSV-Import, Reports.
- `ocr-agent/*`: Python-Skripte für OCR.

## Arbeitsabläufe
1. Stack starten über Menü.
2. Dokumente (PDFs, CSVs) in `input/` ablegen oder über Menü importieren.
3. OCR-Agent verarbeitet automatisch neue Dateien.
4. Jahresstatus synchronisieren.
5. Daten im Browser-UI abrufen.
6. Backups manuell oder automatisch sichern.

## Hinweise und Best Practices
- Immer sicherstellen, dass Colima und Docker laufen, bevor Import gestartet wird.
- OCR-Agent kontinuierlich laufen lassen, um neue Dokumente sofort zu verarbeiten.
- Gesperrte PDFs zuerst entsperren lassen, bevor OCR startet.
- Backups regelmäßig prüfen.
- Web-Applikation für Bürger so gestalten, dass die Suche und Anzeige intuitiv sind.
