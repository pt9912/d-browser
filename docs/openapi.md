# OpenAPI-Vertrag: d-browser

## Zweck

Dieses Dokument beschreibt den REST-/OpenAPI-Vertrag von `d-browser`.
Es ergaenzt das Lastenheft [lastenheft.md](lastenheft.md), die Architektursicht [architecture.md](architecture.md) und das Designdokument [design.md](design.md) um eine konkrete Sicht auf REST als Driving Adapter.

Der gRPC-Vertrag wird in [grpc.md](grpc.md) beschrieben.
Beide Transports setzen auf denselben fachlichen Anwendungsfaellen auf (LT-004).

## Geltungsbereich

Dieses Dokument umfasst:

* den REST-Vertrag des REST-Driving-Adapters (`adapters/driving/service-rest`)
* die Abbildung fachlicher Use Cases auf HTTP-Ressourcen und -Methoden
* Vereinbarungen zu Fehlermodell, Medientypen und Versionierung

Nicht Teil dieses Dokuments:

* Implementierungsdetails des Service-Codes
* Transport- und Deployment-Konfiguration
* Referenz-Client-spezifische Bindings

## Leitprinzipien

* REST ist ein alternativer Transport neben gRPC (LF-026).
* Der Service darf keine eigene Fachlogik einfuehren (LT-003).
* Die fachlichen Vertraege entsprechen den Inbound Ports aus [architecture.md](architecture.md).
* Die OpenAPI-Spezifikation ist die verbindliche Quelle fuer den REST-Vertrag.

## Spezifikationsformat

* OpenAPI 3.1 wird als Spezifikationsformat vorgesehen.
* Der Speicherort der Spezifikation ist noch zu definieren (z.B. `adapters/driving/service-rest/openapi.yaml`).
* Schema- und Type-Definitionen sollen wiederverwendbar und an die fachlichen Modelle aus `hexagon/core` angelehnt sein.

## Ressourcenuebersicht (Entwurf)

Vorgesehen sind mindestens die folgenden Ressourcengruppen, abgeleitet aus den Anwendungsfaellen in [lastenheft.md](lastenheft.md) Kapitel 13:

| Bereich         | Anwendungsfall                 | LF-Ref          |
| --------------- | ------------------------------ | --------------- |
| Schema          | Schema anzeigen                | LF-006, Kap. 13.1 |
| Datensatz       | Datensatz laden                | LF-008, Kap. 13.2 |
| Navigation      | Beziehungen navigieren         | LF-010 bis LF-013, Kap. 13.3 |
| Export          | Baum als JSON / YAML abrufen   | LF-020, LF-021  |
| Integration     | `d-migrate`-Quelle ansprechen  | LI-001 bis LI-005 |

### Endpunkt-Entwurf

Die folgenden Endpunkte bilden die obigen Bereiche konkret ab.
Alle Endpunkte liegen unter dem versionierten Pfad-Praefix (z.B. `/v1`).

| Methode | Pfad                                | Zweck                                                           | LF-Ref          |
| ------- | ----------------------------------- | --------------------------------------------------------------- | --------------- |
| GET     | `/schemas`                          | Liste der verfuegbaren Schemas einer Quelle                     | LF-006, Kap. 13.1 |
| GET     | `/schemas/{schema}/tables`          | Liste der Tabellen eines Schemas mit Basismetadaten             | LF-006, Kap. 13.1 |
| GET     | `/tables/{table}/rows`              | Pagineerte Datensaetze einer Tabelle                            | LF-008, LF-019, LN-003 |
| GET     | `/rows/{table}/{pk}/tree`           | Baumprojektion eines Datensatzes inklusive Beziehungen (JSON)   | LF-006 bis LF-019, LF-020, Kap. 13.3 |
| GET     | `/rows/{table}/{pk}/yaml`           | YAML-Export desselben Baumes (`application/yaml`)               | LF-021, Kap. 13.5 |

Hinweise zum Entwurf:

* Zusammengesetzte Primaerschluessel werden als komma- oder schraegstrich-separierte Werte in `{pk}` abgebildet; die konkrete Syntax ist noch zu fixieren.
* Navigationstiefe und Zyklenbehandlung der Baumantworten werden ueber Query-Parameter gesteuert (z.B. `depth`, `include-incoming`, `revisit`).
* Sammlungen (`/schemas`, `/tables/{table}/rows`) unterstuetzen Pagination gemaess Kapitel Querschnittsthemen.
* Der JSON-Baum aus `/rows/{table}/{pk}/tree` und das YAML aus `/rows/{table}/{pk}/yaml` muessen denselben fachlichen Inhalt liefern und duerfen sich nur im Serialisierungsformat unterscheiden.
* Tabellenpfade ohne Schemasegment setzen einen eindeutigen Tabellennamen in der aktiven Quelle voraus; fuer mehrdeutige Faelle ist ein `schema`-Query-Parameter oder ein voll qualifizierter Tabellenname vorzusehen.

## Medientypen

* `application/json` ist der Standardmedientyp.
* `application/yaml` wird fuer den YAML-Export unterstuetzt (LF-021).
* Content-Negotiation erfolgt ueber den `Accept`-Header.

## Fehlermodell

* Fachliche Fehler werden auf sprechende HTTP-Status-Codes abgebildet (z.B. `404`, `409`, `400`).
* Technische Fehler werden als `500` oder `503` signalisiert.
* Fehlerdetails folgen einem einheitlichen Response-Schema (Problem-Details nach RFC 9457 wird als Kandidat betrachtet).
* Das gemeinsame Fehlermodell fuer CLI, REST und gRPC ist noch zu klaeren (siehe [architecture.md](architecture.md), Abschnitt Architekturentscheidungen mit spaeterem Klaerungsbedarf).

## Querschnittsthemen

* Korrelations-IDs werden ueber HTTP-Header transportiert.
* Pagination und Lazy Loading (LN-003) folgen einem transportunabhaengigen Konzept; die konkrete Abbildung (Cursor, Page-Parameter) ist noch zu definieren.
* Authentifizierung und Autorisierung sind in der ersten Ausbaustufe kein Pflichtbestandteil.

## Versionierung

* Die API wird versioniert, vorzugsweise ueber ein Pfad-Praefix (z.B. `/v1/`).
* Nicht abwaertskompatible Aenderungen erfordern eine neue Major-Version.
* Additive Aenderungen bleiben innerhalb derselben Major-Version moeglich.

## Offene Punkte

* Finale Entscheidung fuer Problem-Details (RFC 9457) als Fehlerformat
* Umfang der ersten Endpunkte im Milestone 0.2.0
* Gemeinsames Fehler- und Response-Modell fuer CLI, REST und gRPC
* Werkzeugkette fuer Spezifikation, Validierung und Code-Generierung
* Reihenfolge von REST und gRPC im ersten Umsetzungszyklus (siehe [roadmap.md](roadmap.md))
