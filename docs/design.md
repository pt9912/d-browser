# Design: d-browser

## 1 Zweck

Dieses Dokument beschreibt den aktuellen Modul-, Struktur- und Repository-Entwurf fuer `d-browser`.
Es ergaenzt das Lastenheft [lastenheft.md](lastenheft.md) um konkrete Designentscheidungen, die nicht Teil der fachlichen Anforderungsbeschreibung sein sollen.

Die logische Schichtenarchitektur, Verantwortlichkeiten und Integrationsprinzipien werden in [architecture.md](architecture.md) beschrieben.

## 2 Ausgangspunkt

Die Zielstruktur orientiert sich an der im Repository [`pt9912/d-migrate`](https://github.com/pt9912/d-migrate) etablierten Trennung in:

* `hexagon`
* `adapters`

In `d-migrate` sind diese Bereiche derzeit unter anderem wie folgt gegliedert:

* `hexagon:core`
* `hexagon:ports`
* `hexagon:application`
* `hexagon:profiling`
* `adapters:driven:*`
* `adapters:driving:cli`

Dieses Strukturmuster wird fuer `d-browser` uebernommen und um Referenz-Clients erweitert.

## 3 Leitprinzipien

* Referenz-Clients greifen ueber REST oder gRPC auf das `d-browser`-Backend zu und enthalten keine fachliche Sonderlogik.
* Veroeffentlichbare Bibliotheken werden von Beispielanwendungen und technischen Einstiegspunkten sauber getrennt.
* Die Modulstruktur bleibt nah an `d-migrate`, damit Integrationen und gemeinsame Konzepte konsistent bleiben.
* Code und Klassen sollen testfreundlich entworfen werden, mit klaren Verantwortlichkeiten, geringer Kopplung und austauschbaren Abhaengigkeiten.
* Fuer reproduzierbare Entwicklungs-, Test- und Integrationsumgebungen wird Docker bevorzugt eingesetzt.
* Plattformabhaengige Toolchains sollen pragmatisch behandelt werden; dabei koennen auch Community-Loesungen genutzt werden, wenn sie fuer den vorgesehenen Zweck ausreichend tragfaehig sind.

## 3.1 Testbarkeit im Design

Testbarkeit ist ein explizites Entwurfsziel fuer `d-browser`.
Daraus folgen insbesondere:

* Klassen und Module sollen eine klar abgegrenzte Verantwortung haben.
* Fachlogik soll moeglichst ohne Infrastruktur oder UI testbar sein.
* Abhaengigkeiten sollen ueber Ports, Interfaces oder klar austauschbare Komponenten angebunden werden.
* Seiteneffekte wie I/O, Netzwerkzugriffe, Datenbankzugriffe und Zeitbezug sollen an den Rand des Systems verschoben werden.
* Komplexe Logik soll in kleinen, deterministischen und isoliert pruefbaren Einheiten aufgebaut werden.
* Referenz-Clients sollen das Verhalten des Backends ueber REST oder gRPC demonstrieren, aber keine schwer testbare fachliche Parallel-Logik einfuehren.

## 4 Zielstruktur des Monorepos

```text
d-browser/
├── hexagon/
│   ├── core/
│   ├── ports/
│   └── application/
├── adapters/
│   ├── driven/
│   │   ├── source-common/
│   │   ├── source-d-migrate/
│   │   └── formats/
│   └── driving/
│       ├── cli/
│       ├── service-rest/
│       └── service-grpc/
├── examples/
│   ├── flutter/
│   ├── maui/
│   └── sveltekit/
├── docs/
├── scripts/
└── packaging/
```

## 5 Modulrollen

### 5.1 Hexagon

* `hexagon/core` enthaelt die fachlichen Kernmodelle und Regeln zur Projektion relationaler Strukturen.
* `hexagon/ports` enthaelt die abstrahierten Schnittstellen fuer Quellen, Tree-Aufbau, Datensatzzugriff und Ausgabe.
* `hexagon/application` enthaelt Use Cases und Orchestrierung zwischen Ports und Domaene.

### 5.2 Adapter

* `adapters/driven/source-common` enthaelt gemeinsam nutzbare technische Bausteine fuer Quellenanbindung und Integration.
* `adapters/driven/source-d-migrate` ist der bevorzugte Integrationspfad fuer relationale Quellen, sofern `d-migrate` die benoetigten Datenbankadapter bereits als wiederverwendbare Libraries bereitstellt.
* `adapters/driven/formats` enthaelt projektspezifische technische Formatadaptionen fuer fachliche Exportformate wie JSON und YAML.
* `adapters/driving/cli`, `adapters/driving/service-rest` und `adapters/driving/service-grpc` enthalten die technischen Einstiegspunkte fuer Benutzung und externe Ansteuerung.

### 5.3 Referenz-Clients

* `examples/flutter` demonstriert die Anbindung an das `d-browser`-Backend aus einem Flutter-Client.
* `examples/maui` demonstriert die Anbindung an das `d-browser`-Backend aus einem .NET-MAUI-Client.
* `examples/sveltekit` demonstriert die Anbindung an das `d-browser`-Backend aus einem SvelteKit-Client.

Die Referenz-Clients sind nicht als veroeffentlichbare Bibliotheken gedacht.
Sie dienen der Validierung von REST- und gRPC-Anbindung, Integrationsfaehigkeit und Monorepo-Zuschnitt.

## 6 Designhinweise

* Die genaue Zahl der Module kann sich im Projektverlauf aendern.
* Die Haupttrennung in `hexagon`, `adapters` und getrennte Referenz-Clients bleibt das verbindliche Zielbild.
* Weitergehende Detailentscheidungen sollten bei Bedarf in ADRs oder ergaenzenden Designnotizen dokumentiert werden.
* Lokale Infrastruktur, Integrationsabhaengigkeiten und wiederholbare Entwicklungsumgebungen sollen nach Moeglichkeit ueber Docker bereitgestellt werden.
* Fuer MAUI-bezogene Entwicklungs- und Validierungsschritte koennen Linux-basierte Community-Ansaetze wie GTK-basierte Renderpfade beruecksichtigt werden, sofern ihre Einsatzgrenzen transparent bleiben.
* Direkte datenbankspezifische Source-Adapter in `d-browser` sind nur dann vorgesehen, wenn `d-migrate` die benoetigte Funktionalitaet nicht in ausreichend wiederverwendbarer Form bereitstellt oder ein begruendeter technischer Sonderfall dies erforderlich macht.
* Zusaetzliche Module wie `profiling` oder `streaming` sollen erst aufgenommen werden, wenn dafuer ein klarer fachlicher oder technischer Bedarf nachgewiesen ist.
