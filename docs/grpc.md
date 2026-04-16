# gRPC-Service: d-browser

## Zweck

Dieses Dokument beschreibt den gRPC-Service-Vertrag von `d-browser`.
Es ergaenzt das Lastenheft [lastenheft.md](lastenheft.md), die Architektursicht [architecture.md](architecture.md) und das Designdokument [design.md](design.md) um eine konkrete Sicht auf den gRPC-Transport als Driving Adapter.

Der REST-Vertrag wird in [openapi.md](openapi.md) beschrieben.
Beide Transports setzen auf denselben fachlichen Anwendungsfaellen auf (LT-004).

## Geltungsbereich

Dieses Dokument umfasst:

* den Service-Vertrag des gRPC-Driving-Adapters (`adapters/driving/service-grpc`)
* die Abbildung fachlicher Use Cases auf gRPC-Methoden
* Vereinbarungen zu Fehlermodell, Metadaten und Versionierung

Nicht Teil dieses Dokuments:

* Implementierungsdetails des Service-Codes
* Transport- und Deployment-Konfiguration
* Referenz-Client-spezifische Bindings

## Leitprinzipien

* gRPC ist ein alternativer Transport neben REST (LF-026).
* Der Service darf keine eigene Fachlogik einfuehren (LT-003).
* Die fachlichen Vertraege entsprechen den Inbound Ports aus [architecture.md](architecture.md).
* Fehler werden transportunabhaengig im Kern modelliert und auf gRPC-Status-Codes abgebildet.

## Service-Oberflaeche (Entwurf)

Die konkrete `.proto`-Definition ist noch offen.
Vorgesehen sind mindestens die folgenden Service-Gruppen, abgeleitet aus den Anwendungsfaellen in [lastenheft.md](lastenheft.md) Kapitel 13:

| Bereich         | Anwendungsfall                 | LF-Ref          |
| --------------- | ------------------------------ | --------------- |
| Schema          | Schema anzeigen                | LF-006, Kap. 13.1 |
| Datensatz       | Datensatz laden                | LF-008, Kap. 13.2 |
| Navigation      | Beziehungen navigieren         | LF-010 bis LF-013, Kap. 13.3 |
| Export          | Baum als JSON / YAML abrufen   | LF-020, LF-021  |
| Integration     | `d-migrate`-Quelle ansprechen  | LI-001 bis LI-005 |

## Fehlermodell

* Fachliche Fehler werden auf sprechende gRPC-Status-Codes abgebildet (z.B. `NOT_FOUND`, `FAILED_PRECONDITION`, `INVALID_ARGUMENT`).
* Technische Fehler werden als `INTERNAL` oder `UNAVAILABLE` signalisiert.
* Details werden ueber `google.rpc.Status` / `error_details` transportiert, damit Kontext nicht verloren geht (LT-007).
* Das gemeinsame Fehlermodell fuer CLI, REST und gRPC ist noch zu klaeren (siehe [architecture.md](architecture.md), Abschnitt Architekturentscheidungen mit spaeterem Klaerungsbedarf).

## Metadaten und Querschnittsthemen

* Korrelations-IDs sollen ueber gRPC-Metadata-Header transportiert werden.
* Pagination und Lazy Loading (LN-003) folgen einem transportunabhaengigen Konzept; die konkrete Abbildung (Cursor, Page-Tokens) ist noch zu definieren.
* Authentifizierung und Autorisierung sind in der ersten Ausbaustufe kein Pflichtbestandteil.

## Versionierung

* Paket- und Service-Namen enthalten eine Versionssegmentierung (z.B. `dbrowser.v1`).
* Nicht abwaertskompatible Aenderungen erfordern eine neue Major-Version.
* Feld-Nummern in Proto-Dateien werden nicht wiederverwendet.

## Offene Punkte

* Wahl der Proto-Syntax-Version und des Build-Werkzeugs (z.B. `protoc`, Gradle-Plugin)
* Umfang der ersten Service-Methoden im Milestone 0.2.0
* Gemeinsames Fehler- und Response-Modell fuer CLI, REST und gRPC
* Streaming-Bedarf fuer Sammlungen grosser Datensaetze
* Reihenfolge von REST und gRPC im ersten Umsetzungszyklus (siehe [roadmap.md](roadmap.md))
