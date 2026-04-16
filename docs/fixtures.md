# Referenz-Fixtures: d-browser

## Zweck

Dieses Dokument listet die Referenz-Datenbanken (Fixtures), die `d-browser` fuer Entwicklung, Tests und Validierung verwendet.
Es ergaenzt das Lastenheft [lastenheft.md](lastenheft.md), die Architektursicht [architecture.md](architecture.md) und die [roadmap.md](roadmap.md) um eine konkrete Sicht auf die Daten, an denen fachliche Anforderungen abgeprueft werden.

## Leitprinzipien

* Fixtures sind keine Produktdaten, sondern kontrollierte Testgrundlagen fuer Schema, Navigation, Export und Integration.
* Fixtures werden ueber den bevorzugten Integrationspfad `source-d-migrate` geladen (d-migrate ueber JDBC).
* Fixtures werden containerisiert bereitgestellt, damit Entwicklungs- und CI-Umgebungen reproduzierbar bleiben (LN-011).
* Die Auswahl deckt bewusst unterschiedliche relationale Muster ab (FK-Tiefe, n:m, zusammengesetzte PKs, Typvielfalt).

## Referenz-Fixtures

### Pagila (PostgreSQL) — primaere Fixture der ersten Ausbaustufe

**Quelle:** [neondatabase/postgres-sample-dbs](https://github.com/neondatabase/postgres-sample-dbs) (SQL-Dump `pagila.sql`)

**Domaene:** DVD-Verleih (Filme, Schauspieler, Kategorien, Filialen, Kunden, Zahlungen). Pagila ist die PostgreSQL-Portierung der MySQL-Sakila-Datenbank mit erweitertem Schema.

**Kenngroessen:** 33 Tabellen, ca. 62.000 Datensaetze.

**Warum geeignet:**

| Merkmal                              | Beispiel in Pagila                                                    | LF-Ref                 |
| ------------------------------------ | --------------------------------------------------------------------- | ---------------------- |
| mehrstufige FK-Navigation            | `customer → rental → inventory → film → language`                     | LF-010 bis LF-012      |
| n:m ueber Join-Tabelle               | `film ↔ film_actor ↔ actor`, `film ↔ film_category ↔ category`        | LF-016                 |
| Selbstreferenz / Wiederverweis       | `staff.store_id` ↔ `store.manager_staff_id`                           | LF-013, LF-017, LF-018 |
| grosse Sammlungen (Pagination)       | ca. 16.000 `rental`, ca. 16.000 `payment`                             | LF-019, LN-003         |
| zusammengesetzte Primaerschluessel   | `film_actor(actor_id, film_id)`, `film_category(film_id, category_id)` | Kap. 13.2, LF-008      |
| Typvielfalt                          | `tsvector` (`film.fulltext`), Enums, Arrays, `numeric`, `date`        | LT-005                 |

**Abgedeckte Anwendungsfaelle aus dem Lastenheft:**

* Kap. 13.1 "Schema anzeigen" — 33 Tabellen im Schema `public`
* Kap. 13.2 "Datensatz laden" — z.B. `customer` mit `customer_id = 1`
* Kap. 13.3 "Beziehungen navigieren" — absteigen von Kunde ueber `rental` zu `inventory`/`film`
* Kap. 13.4 / 13.5 "Export nach JSON / YAML" — Baumexport der geladenen Struktur

**Ladepfad (Entwurf):**

* SQL-Dump: `https://raw.githubusercontent.com/neondatabase/postgres-sample-dbs/main/pagila.sql`
* Bereitstellung ueber einen Docker-Compose-Service mit PostgreSQL und vorgeladenem Dump (genauer Ort und Name werden im Integrationspfad 0.1.0 festgelegt).
* Anbindung in `d-browser` ueber `source-d-migrate` mit einer lokalen Verbindungskonfiguration.

### Weitere Fixtures (vorgesehen)

Weitere Fixtures werden erst aufgenommen, wenn die primaere Fixture stabil genutzt wird und ein konkreter fachlicher oder technischer Bedarf besteht. Kandidaten:

* **Sakila (MySQL)** — gleiche Domaene wie Pagila, aber in MySQL; dient der Validierung des MySQL-Treibers.
* **Chinook (SQLite)** — deutlich kleineres Schema (Musikshop), geeignet als Leichtgewicht-Fixture fuer schnelle Tests und SQLite-Treibervalidierung.

Diese werden hier erst detailliert dokumentiert, wenn sie aktiv genutzt werden.

## Nutzung in Tests und Entwicklung

* Automatisierte Tests gegen die Fachlogik verwenden primaer Pagila als Referenz fuer FK-Navigation, n:m-Behandlung und Export.
* Integrationstests fuer `source-d-migrate` pruefen, dass Pagila ueber den Adapter vollstaendig in das kanonische Modell ueberfuehrt wird.
* Manuelle Validierung (CLI, REST, gRPC) setzt ebenfalls auf Pagila auf, damit Verhalten zwischen Transporten vergleichbar bleibt (LT-004).

## Abgrenzung

* Fixtures werden nicht fuer Leistungstests realer Datenmengen herangezogen; dafuer gelten gesonderte Festlegungen.
* Fixtures ersetzen keine produktionsnahe Quelle und sind bewusst klein und reproduzierbar gehalten.

## Offene Punkte

* Endgueltiger Ablageort der Docker-Definitionen (voraussichtlich unter `packaging/` oder `scripts/`).
* Genauer Bezugs-Commit des Pagila-Dumps fuer reproduzierbare Builds.
* Umgang mit optionalen Pagila-Erweiterungen (Partitionen, Ansichten) im ersten Integrationsschritt.
