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

In der ersten Ausbaustufe werden drei Fixtures priorisiert:

1. **Pagila** (PostgreSQL) — breites Schema mit vielfaeltigen FK-Mustern
2. **Sakila** (jOOQ, Multi-Dialekt) — identische Domaene ueber mehrere SQL-Dialekte, fuer Adapter-Gegenproben
3. **Employees** (datacharmer/test_db) — grosse Datenmengen fuer Pagination und Lazy Loading

Weitere Fixtures werden erst aufgenommen, wenn diese drei stabil genutzt werden und ein konkreter fachlicher oder technischer Bedarf besteht.

### Pagila (PostgreSQL) — primaere Fixture der ersten Ausbaustufe

**Quelle:** [neondatabase-labs/postgres-sample-dbs](https://github.com/neondatabase-labs/postgres-sample-dbs) (SQL-Dump `pagila.sql`)

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

* SQL-Dump: `https://raw.githubusercontent.com/neondatabase-labs/postgres-sample-dbs/main/pagila.sql`
* Bereitstellung ueber einen Docker-Compose-Service mit PostgreSQL und vorgeladenem Dump (genauer Ort und Name werden im Integrationspfad 0.1.0 festgelegt).
* Anbindung in `d-browser` ueber `source-d-migrate` mit einer lokalen Verbindungskonfiguration.

### Sakila (jOOQ, Multi-Dialekt) — zweite Fixture

**Quelle:** [jOOQ/sakila](https://github.com/jOOQ/sakila) (BSD-Lizenz; urspruenglich von Mike Hillyer / MySQL AB, portiert durch DB Software Laboratory)

**Domaene:** DVD-Verleih, identisch zur Sakila-Referenz; Pagila ist die PostgreSQL-Portierung davon.

**Kenngroessen:** 16 Tabellen, bewusst kleineres Schema als Pagila.

**Dialektabdeckung:** Das Repo stellt dasselbe Schema in acht SQL-Dialekten bereit:

* PostgreSQL
* MySQL
* SQLite
* Oracle
* DB2
* SQL Server
* CockroachDB
* YugabyteDB

**Warum geeignet:**

| Merkmal                                  | Beispiel in Sakila                                   | LF-Ref                 |
| ---------------------------------------- | ---------------------------------------------------- | ---------------------- |
| Multi-Dialekt-Validierung fuer d-migrate | identisches Schema ueber PostgreSQL, MySQL, SQLite   | LF-001, LT-005         |
| n:m ueber Join-Tabelle                   | `film_actor`, `film_category`                        | LF-016                 |
| zusammengesetzte Primaerschluessel       | `film_actor(actor_id, film_id)`                      | Kap. 13.2, LF-008      |
| Audit-Spalten                            | `last_update` in jeder Tabelle                       | LT-005                 |
| Vergleich mit Pagila                     | Paar aus kleinerem Sakila und erweitertem Pagila     | LF-004, LF-005         |

**Nutzung in `d-browser`:** primaer als Gegenprobe fuer den `source-d-migrate`-Adapter, damit identische Fachmodelle unabhaengig vom Dialekt entstehen (LF-004, LF-005). Dient gleichzeitig der Validierung der weiteren d-migrate-Treiber (MySQL, SQLite).

### Employees — dritte Fixture

**Quelle:** [datacharmer/test_db](https://github.com/datacharmer/test_db) (Creative Commons Attribution-Share Alike 3.0 Unported; Originaldaten Fusheng Wang und Carlo Zaniolo, Siemens Corporate Research)

**Domaene:** Personaldaten (Mitarbeiter, Abteilungen, Titel, Gehaelter, Fuehrungszuordnungen).

**Kenngroessen:** 6 Tabellen, ca. 2,8 Millionen Datensaetze insgesamt (u.a. ca. 300.024 `employees` und ca. 2.844.047 `salaries`).

**Zielsysteme:** MySQL 5.0+ (getestet bis 9.6) und PostgreSQL 12+ (getestet bis 17); MariaDB- und Percona-kompatibel.

**Warum geeignet:**

| Merkmal                                   | Beispiel in Employees                                       | LF-Ref                 |
| ----------------------------------------- | ----------------------------------------------------------- | ---------------------- |
| grosse Datenmengen                        | `salaries` mit ca. 2,8 Mio Zeilen                           | LF-019, LN-003         |
| Pagination und Lazy Loading unter Last    | grosse `to-many`-Sammlungen je Employee                     | LF-015, LF-019, LN-003 |
| historische Mehrfachzuordnung             | `salaries(emp_no, from_date)` als zusammengesetzter PK      | LF-008, Kap. 13.2      |
| Manager- und Abteilungsbeziehungen        | `dept_manager`, `dept_emp`                                  | LF-010 bis LF-013      |
| Multi-Dialekt (MySQL + PostgreSQL)        | gleicher Inhalt ueber beide Dialekte ladbar                 | LF-004, LT-005         |

**Nutzung in `d-browser`:** primaer als Fixture fuer volumenbezogene Anforderungen (LF-019, LN-003) und zum Pruefen von Pagination und Lazy Loading unter realistischen Zeilenzahlen. Sekundaer zur Validierung des MySQL-Treibers parallel zu Sakila.

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
