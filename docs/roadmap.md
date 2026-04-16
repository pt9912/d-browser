# Roadmap: d-browser

## Zweck

Diese Roadmap beschreibt die geplanten Ausbaustufen von `d-browser` auf hoher Ebene.
Sie ergaenzt das Lastenheft [lastenheft.md](lastenheft.md) und das Designdokument [design.md](design.md) um eine zeitliche und inhaltliche Entwicklungsrichtung.

## Leitgedanken

* Zuerst fachlichen Kern und Integrationsfaehigkeit absichern.
* Danach technische Einstiegspunkte und Referenz-Clients aufbauen.
* Zuschnitt klein halten und Erweiterungen erst nach belastbarer Basis angehen.

## Meilensteine

### 0.1.0 Grundlagen

| Bereich | Aufgabe | LF-Ref | Status |
| ------- | ------- | ------ | ------ |
| Struktur | Monorepo-Struktur herstellen | LT-012, LT-013 | offen |
| Hexagon | Module `hexagon/core`, `hexagon/ports` und `hexagon/application` aufsetzen | LT-001, LT-012 | offen |
| Integration | Integrationspfad ueber `source-d-migrate` vorbereiten | LF-022, LI-001 bis LI-005 | offen |
| Fachlogik | Grundlegende Baumprojektion und Datensatznavigation implementieren | LF-006 bis LF-019 | offen |
| Ausgabe | JSON- und YAML-Ausgabe bereitstellen | LF-020, LF-021 | offen |

Ergebnis:

* Ein minimal nutzbarer fachlicher Kern mit ersten Tests und klaren Schnittstellen

### 0.2.0 Einstiegspunkte

| Bereich | Aufgabe | LF-Ref | Status |
| ------- | ------- | ------ | ------ |
| Driving | CLI bereitstellen | LT-003, LN-008 | offen |
| Service | REST-Service bereitstellen | LF-026, LT-003, LT-004 | offen |
| Service | gRPC-Service bereitstellen | LF-026, LT-003, LT-004 | offen |
| Architektur | Gemeinsame Use-Case-Schicht unterhalb der Transportadapter absichern | LT-001, LT-004 | offen |

Ergebnis:

* Fachliche Funktionen sind ueber definierte technische Einstiegspunkte nutzbar

### 0.3.0 Referenz-Clients

| Bereich | Aufgabe | LF-Ref | Status |
| ------- | ------- | ------ | ------ |
| Example | Flutter-Example ueber REST oder gRPC an das Backend anbinden | LF-023, LF-024, LF-025 | offen |
| Example | MAUI-Example ueber REST oder gRPC an das Backend anbinden | LF-023, LF-024, LF-025, LN-010 | offen |
| Example | SvelteKit-Example ueber REST oder gRPC an das Backend anbinden | LF-023, LF-024, LF-025 | offen |
| Validierung | Schemauebersicht und einfache Datensatz- oder Tabellenansicht in allen Examples ueber REST oder gRPC demonstrieren | LF-024, LF-025 | offen |

Ergebnis:

* Die freigegebenen Schnittstellen sind aus mehreren Client-Technologien heraus validiert

### 0.4.0 Stabilisierung und Veroeffentlichung

| Bereich | Aufgabe | LF-Ref | Status |
| ------- | ------- | ------ | ------ |
| Build | Build-, Test- und Docker-Setup vereinheitlichen | LN-006, LN-011, LT-015 | offen |
| Publishing | Bibliotheken fuer Maven Central vorbereiten | LT-008 bis LT-011 | offen |
| Dokumentation | Dokumentation vervollstaendigen | - | offen |
| Abnahme | Abnahmekriterien aus dem Lastenheft systematisch nachweisen | Abschnitt 15 | offen |

Ergebnis:

* Eine stabile Basis fuer Nutzung, Weiterentwicklung und externe Einbindung

### 0.5.0 Erweiterungen

| Bereich | Aufgabe | LF-Ref | Status |
| ------- | ------- | ------ | ------ |
| Quellen | Weitere relationale Quellen priorisiert anbinden | LF-001, LT-005 | offen |
| Funktion | Suche, Filterung oder Navigationstiefe erweitern | - | offen |
| Produkt | Protokolle und Client-Funktionen auf Basis realer Nutzung nachschaerfen | LF-023 bis LF-026, LT-004 | offen |

Ergebnis:

* Das System entwickelt sich von einer validierten Basis zu einer breiter nutzbaren Arbeitsgrundlage

## Spaetere Ausbauoptionen

* Weitere relationale Quellen
* Erweiterte Suche und Filterung
* GraphQL
* Weitergehende Visualisierung
* Zusätzliche produktive Frontends

## Offene Punkte

* Priorisierung der ersten konkret zu liefernden Use Cases
* Reihenfolge von REST und gRPC im ersten Umsetzungszyklus
* Umfang der ersten Referenz-Clients
* Umfang der ersten veröffentlichbaren Bibliotheken
