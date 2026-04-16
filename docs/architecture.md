# Architecture: d-browser

## Zweck

Dieses Dokument beschreibt die angestrebte Architektur von `d-browser`.
Es ergaenzt das Lastenheft [lastenheft.md](lastenheft.md), das Designdokument [design.md](design.md) und die [roadmap.md](roadmap.md) um eine strukturierte logische Architektursicht.

`architecture.md` beschreibt dabei:

* Systemkontext
* fachliche und technische Schichten
* Verantwortlichkeiten und Abhaengigkeitsrichtungen
* Integrationsprinzipien

Das Dokument [design.md](design.md) beschreibt dagegen primaer den Modul- und Repository-Zuschnitt des Monorepos.

## Architekturziele

* Fachlogik von Infrastruktur und Transportprotokollen trennen
* Integrationsfaehigkeit ueber `d-migrate` sicherstellen
* Mehrere Einstiegspunkte wie CLI, REST und gRPC auf denselben Use Cases aufbauen
* Referenz-Clients ohne doppelte Fachlogik anbinden
* Testbarkeit und Erweiterbarkeit im Monorepo erhalten

## Systemkontext

```text
                  ┌───────────────────────────────────────────────┐
                  │                   d-browser                   │
Benutzer ──CLI───>│                                               │
                  │  ┌──────────────┐  ┌───────────────────────┐  │
Systeme ─REST────>│  │ Driving      │  │   Application /       │  │
                  │  │ Adapter      │  │   Use Cases           │  │
Systeme ─gRPC────>│  │ CLI / REST   │  │                       │  │
                  │  │ / gRPC       │  └──────────┬────────────┘  │
Examples ──REST/─>│  └──────┬───────┘             │               │
Examples ──gRPC──>│         │                     │               │
                  │         │                Inbound Ports        │
                  │         │                     │               │
                  │  ┌──────▼─────────────────────▼────────────┐  │
                  │  │               Core Domain               │  │
                  │  │ kanonisches Modell, Baumprojektion,     │  │
                  │  │ Navigation, Zyklenschutz                │  │
                  │  └──────┬─────────────────────┬────────────┘  │
                  │         │                Outbound Ports       │
                  │         │                     │               │
                  │  ┌──────▼───────┐   ┌────────▼─────────────┐  │
                  │  │ source-      │   │ formats              │  │
                  │  │ d-migrate    │   │ JSON / YAML Export   │  │
                  │  └──────┬───────┘   └────────┬─────────────┘  │
                  └─────────┼────────────────────┼────────────────┘
                            │                    │
                  ┌─────────▼────────────┐   ┌───▼──────┐
                  │ d-migrate Libraries  │   │ JSON     │
                  │ Driver / Metadata    │   │ YAML     │
                  │ / Mapping            │   │          │
                  └──────────────────────┘   └──────────┘
```

## Logische Architekturuebersicht

`d-browser` folgt einer Architektur mit fachlichem Kern, Inbound- und Outbound-Ports sowie klar getrennten Driving- und Driven-Adaptern.

Die logische Abhaengigkeitsrichtung ist:

```text
Externe Konsumenten
        |
        v
Driving Adapter
CLI | REST | gRPC
        |
        v
Inbound Ports
        |
        v
Application / Use Cases
        |
        v
Core Domain
        |
        v
Outbound Ports
        |
        v
Driven Adapter
source-d-migrate | formats
```

## Schichten

### 1. Core Domain

Die Core Domain enthaelt:

* das kanonische fachliche Modell
* Regeln fuer Baumprojektion
* Navigation ueber Beziehungen
* Zyklenerkennung und Wiederverweise

Die Core Domain kennt keine Transportprotokolle, keine UI-Technologien und keine konkreten Datenquellen.

### 2. Inbound Ports

Inbound Ports definieren die fachlichen Eingangsvertraege, ueber die Anwendungsfaelle von CLI, REST, gRPC oder anderen Backend-Konsumenten aufgerufen werden.

Dazu gehoeren insbesondere fachliche Vertraege fuer:

* Schema anzeigen
* Datensatz laden
* Beziehungen navigieren
* fachliche Ergebnisse fuer Export oder Services bereitstellen

Inbound Ports werden von der Application-Schicht implementiert und von Driving Adaptern genutzt.

### 3. Outbound Ports

Outbound Ports definieren die technischen oder integrationsbezogenen Abhaengigkeiten, die der fachliche Kern nach aussen benoetigt.

Dazu gehoeren insbesondere Schnittstellen fuer:

* Bezug von Schema- und Datensatzinformationen
* Export oder fachliche Ausgabeerzeugung
* weitere technische Integrationen, die nicht Teil des Kerns sind

Outbound Ports werden von Driven Adaptern implementiert und von Application/Core genutzt.

Zusammen bilden Inbound und Outbound Ports die vertragliche Grenze zwischen Kern und Adaptern.

### 4. Application / Use Cases

Die Application-Schicht orchestriert fachliche Anwendungsfaelle.
Sie implementiert Inbound Ports und nutzt Outbound Ports, ohne selbst transport- oder UI-spezifische Logik zu enthalten.

Typische Aufgaben:

* Schema anzeigen
* Datensatz laden
* Beziehungen navigieren
* Ergebnisse fuer Ausgabe oder Services bereitstellen

### 5. Driven Adapter

Driven Adapter binden technische Quellen und Ausgabemechanismen an.

Im aktuellen Zielbild sind insbesondere vorgesehen:

* `source-d-migrate` als bevorzugter Integrationspfad fuer relationale Quellen
* `formats` fuer fachliche Exportformate wie JSON und YAML

Direkte datenbankspezifische Quellenadapter in `d-browser` sind nur als Ausnahmefall vorgesehen.
Transport- oder Protokollserialisierung fuer REST und gRPC gehoert nicht in `formats`, sondern in die jeweiligen Driving Adapter.

### 6. Driving Adapter

Driving Adapter stellen die fachlichen Funktionen nach aussen bereit.

Vorgesehen sind:

* `cli`
* `service-rest`
* `service-grpc`

Alle Driving Adapter sollen auf denselben Use Cases aufsetzen und keine eigene Fachlogik duplizieren.

### 7. Examples

Die Examples in Flutter, MAUI und SvelteKit sind Frontends fuer das `d-browser`-Backend.
Sie greifen ueber REST oder gRPC auf dieselben fachlichen Anwendungsfaelle zu und dienen der Validierung von API, Integrationsfaehigkeit und ergonomischer Nutzbarkeit.

## Integrationsprinzipien

### d-migrate

`d-browser` soll bestehende Funktionalitaet aus `d-migrate` bevorzugt wiederverwenden.
Die Integration erfolgt ueber einen dedizierten Adapter und darf die Fachlogik nicht an interne Toolmodelle koppeln.

### REST und gRPC

REST und gRPC sind alternative Transportwege fuer dieselben fachlichen Anwendungsfaelle.
Unterschiede sollen auf Transport, Vertrag und transportspezifische Serialisierung begrenzt bleiben.

### Docker

Docker wird fuer reproduzierbare Entwicklungs-, Test- und Integrationsumgebungen bevorzugt eingesetzt.
Die Architektur soll deshalb lokale Infrastruktur und Integrationsabhaengigkeiten sauber containerisierbar halten.

## Architekturrisiken

* zu viel Fachlogik in CLI, REST, gRPC oder Examples
* implizite Kopplung an `d-migrate`-Interne statt an stabile Integrationsvertraege
* parallele Implementierung derselben Logik ueber mehrere Transportpfade
* zu fruehe Aufspaltung in Module ohne klaren fachlichen oder technischen Bedarf

## Architekturentscheidungen mit spaeterem Klaerungsbedarf

* Zuschnitt des kanonischen Modells
* exakte Port-Vertraege fuer Lesen, Navigation und Ausgabe
* Umfang und Form des gRPC-Service-Vertrags
* gemeinsame Fehler- und Response-Modelle fuer CLI, REST und gRPC
* Bedarf fuer spaetere Zusatzmodule wie `profiling` oder `streaming`
