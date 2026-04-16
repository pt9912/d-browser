# Design: d-browser

## 1 Zweck

Dieses Dokument beschreibt den aktuellen Architektur- und Strukturentwurf fuer `d-browser`.
Es ergaenzt das Lastenheft [lastenheft.md](/Development/d-browser/docs/lastenheft.md) um konkrete Designentscheidungen, die nicht Teil der fachlichen Anforderungsbeschreibung sein sollen.

## 2 Ausgangspunkt

Die Zielstruktur orientiert sich an der in `/Development/d-migrate` etablierten Trennung in:

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

* Die fachliche Kernlogik bleibt von Infrastruktur, UI und konkreten Datenquellen getrennt.
* Referenz-Clients konsumieren definierte Schnittstellen oder Services und enthalten keine fachliche Sonderlogik.
* Veroeffentlichbare Bibliotheken werden von Beispielanwendungen und technischen Einstiegspunkten sauber getrennt.
* Die Modulstruktur bleibt nah an `d-migrate`, damit Integrationen und gemeinsame Konzepte konsistent bleiben.
* Code und Klassen sollen testfreundlich entworfen werden, mit klaren Verantwortlichkeiten, geringer Kopplung und austauschbaren Abhaengigkeiten.
* Fuer reproduzierbare Entwicklungs-, Test- und Integrationsumgebungen wird Docker bevorzugt eingesetzt.
* Plattformabhaengige Toolchains sollen pragmatisch behandelt werden; dabei koennen auch Community-Loesungen genutzt werden, wenn sie fuer den vorgesehenen Zweck ausreichend tragfaehig sind.
* Unterschiedliche Transportprotokolle wie REST und gRPC sollen auf denselben fachlichen Anwendungsfaellen aufsetzen.

## 3.1 Testbarkeit im Design

Testbarkeit ist ein explizites Entwurfsziel fuer `d-browser`.
Daraus folgen insbesondere:

* Klassen und Module sollen eine klar abgegrenzte Verantwortung haben.
* Fachlogik soll moeglichst ohne Infrastruktur oder UI testbar sein.
* Abhaengigkeiten sollen ueber Ports, Interfaces oder klar austauschbare Komponenten angebunden werden.
* Seiteneffekte wie I/O, Netzwerkzugriffe, Datenbankzugriffe und Zeitbezug sollen an den Rand des Systems verschoben werden.
* Komplexe Logik soll in kleinen, deterministischen und isoliert pruefbaren Einheiten aufgebaut werden.
* Referenz-Clients sollen Integrationsverhalten demonstrieren, aber keine schwer testbare fachliche Parallel-Logik einfuehren.

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

* `adapters/driven/source-d-migrate` ist der bevorzugte Integrationspfad fuer relationale Quellen, sofern `d-migrate` die benoetigten Datenbankadapter bereits als wiederverwendbare Libraries bereitstellt.
* `adapters/driven/formats` enthaelt browser-spezifische technische Formatadaptionen, insbesondere fuer Ausgabe und Serialisierung.
* `adapters/driving/cli`, `adapters/driving/service-rest` und `adapters/driving/service-grpc` enthalten die technischen Einstiegspunkte fuer Benutzung und externe Ansteuerung.
* REST- und gRPC-Adapter sollen dieselben fachlichen Anwendungsfaelle adressieren und sich nur im Transport und Vertragsformat unterscheiden.

### 5.3 Referenz-Clients

* `examples/flutter` demonstriert die Anbindung aus einem Flutter-Client.
* `examples/maui` demonstriert die Anbindung aus einem .NET-MAUI-Client.
* `examples/sveltekit` demonstriert die Anbindung aus einem SvelteKit-Client.

Die Referenz-Clients sind nicht als veroeffentlichbare Bibliotheken gedacht.
Sie dienen der Validierung von API, Integrationsfaehigkeit und Monorepo-Zuschnitt.

## 6 Designhinweise

* Die genaue Zahl der Module kann sich im Projektverlauf aendern.
* Die Haupttrennung in `hexagon`, `adapters` und getrennte Referenz-Clients bleibt das verbindliche Zielbild.
* Weitergehende Detailentscheidungen sollten bei Bedarf in ADRs oder ergaenzenden Designnotizen dokumentiert werden.
* Lokale Infrastruktur, Integrationsabhaengigkeiten und wiederholbare Entwicklungsumgebungen sollen nach Moeglichkeit ueber Docker bereitgestellt werden.
* Fuer MAUI-bezogene Entwicklungs- und Validierungsschritte koennen Linux-basierte Community-Ansaetze wie GTK-basierte Renderpfade beruecksichtigt werden, sofern ihre Einsatzgrenzen transparent bleiben.
* Falls sowohl REST als auch gRPC angeboten werden, sollen gemeinsame Service- und Use-Case-Schichten unterhalb der Protokolladapter wiederverwendet werden.
* Direkte datenbankspezifische Source-Adapter in `d-browser` sind nur dann vorgesehen, wenn `d-migrate` die benoetigte Funktionalitaet nicht in ausreichend wiederverwendbarer Form bereitstellt oder ein begruendeter technischer Sonderfall dies erforderlich macht.
* Zusaetzliche Module wie `profiling` oder `streaming` sollen erst aufgenommen werden, wenn dafuer ein klarer fachlicher oder technischer Bedarf nachgewiesen ist.
