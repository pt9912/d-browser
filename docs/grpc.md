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

## Adressierung von Datenquellen

`d-browser` bindet relationale Quellen ueber den `source-d-migrate`-Adapter (d-migrate ueber JDBC) an.
Mehrere Quellen-Instanzen werden ueber ein Feld `source` in den jeweiligen Request-Messages adressiert.
Ohne explizites `source`-Feld wird die in der Service-Konfiguration hinterlegte Standardquelle verwendet.

## Service-Oberflaeche (Entwurf)

Die konkrete `.proto`-Definition ist noch offen.
Vorgesehen sind mindestens die folgenden Service-Gruppen, abgeleitet aus den Anwendungsfaellen in [lastenheft.md](lastenheft.md) Kapitel 13:

| Bereich         | Anwendungsfall                 | LF-Ref                       |
| --------------- | ------------------------------ | ---------------------------- |
| Schema          | Schema anzeigen                | LF-001, LF-002, Kap. 13.1    |
| Datensatz       | Datensatz laden                | LF-008, LF-009, Kap. 13.2    |
| Navigation      | Beziehungen navigieren         | LF-010 bis LF-013, Kap. 13.3 |
| Export          | Baum als JSON / YAML abrufen   | LF-020, LF-021               |
| Integration     | `d-migrate`-Quelle ansprechen  | LI-001 bis LI-005            |

### Methoden-Entwurf

Die folgenden Methoden setzen 1:1 auf dieselben fachlichen Use Cases auf wie die REST-Endpunkte in [openapi.md](openapi.md) und erfuellen damit LT-004.

| Service          | Methode            | Zweck                                                        | REST-Pendant                       | LF-Ref                                           |
| ---------------- | ------------------ | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------ |
| `SchemaService`  | `ListSchemas`      | Liste der verfuegbaren Schemas der aktiven Quelle            | `GET /schemas`                     | LF-001, Kap. 13.1                                |
| `SchemaService`  | `ListTables`       | Liste der Tabellen eines Schemas mit Basismetadaten          | `GET /schemas/{schema}/tables`     | LF-002, Kap. 13.1                                |
| `RowService`     | `ListRows`         | Paginierte Datensaetze einer Tabelle                         | `GET /tables/{table}/rows`         | LF-008, LF-019, LN-003                           |
| `RowService`     | `GetTree`          | Baumprojektion eines Datensatzes inklusive Beziehungen       | `GET /rows/{table}/{pk}/tree`      | LF-007, LF-010 bis LF-013, LF-017 bis LF-019, LF-020, Kap. 13.3 |
| `RowService`     | `GetTreeYaml`      | YAML-Serialisierung desselben Baumes                         | `GET /rows/{table}/{pk}/yaml`      | LF-021, Kap. 13.5                                |

Hinweise zum Entwurf:

* Navigationstiefe und Zyklenbehandlung von `GetTree` werden ueber Request-Felder gesteuert (`depth`, `include_incoming`, `on_revisit`) und entsprechen den gleichnamigen Query-Parametern in [openapi.md](openapi.md).
* Sammlungen (`ListSchemas`, `ListRows`) unterstuetzen Pagination gemaess Kapitel Querschnittsthemen.
* Streaming-Varianten (z.B. `server streaming` fuer grosse Sammlungen) sind vorstellbar und als offener Punkt vermerkt.

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
