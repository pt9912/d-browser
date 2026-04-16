# Lastenheft

**Projekt:** d-browser
**Dokumenttyp:** Lastenheft
**Version:** 1.8
**Stand:** 16.04.2026
**Status:** Entwurf
**Autor:** OpenAI / ChatGPT
**Auftraggeber:** *noch festzulegen*
**Freigabe durch:** *noch festzulegen*

---

## Dokumentenkontrolle

| Feld         | Wert                 |
| ------------ | -------------------- |
| Dokumentname | Lastenheft d-browser |
| Version      | 1.8                  |
| Datum        | 16.04.2026           |
| Status       | Entwurf              |
| Autor        | OpenAI / ChatGPT     |
| Prüfer       | *offen*              |
| Freigeber    | *offen*              |

---

## Änderungsverlauf

| Version | Datum      | Änderung                                 | Autor            |
| ------- | ---------- | ---------------------------------------- | ---------------- |
| 0.1     | 15.04.2026 | Erstfassung                              | OpenAI / ChatGPT |
| 1.0     | 15.04.2026 | Offizielle Strukturierung als Lastenheft | OpenAI / ChatGPT |
| 1.1     | 16.04.2026 | Erweiterung um Publishing- und Modulanforderungen | OpenAI / Codex |
| 1.2     | 16.04.2026 | Ergänzung um Monorepo- und Referenz-UI-Anforderungen | OpenAI / Codex |
| 1.3     | 16.04.2026 | Zielstruktur des Monorepos auf Basis von d-migrate ergänzt | OpenAI / Codex |
| 1.4     | 16.04.2026 | Nummerierte Anforderungen ergänzt und Design nach `docs/design.md` ausgelagert | OpenAI / Codex |
| 1.5     | 16.04.2026 | Anforderungen atomarisiert, Lastenheft bereinigt, Abnahme und Plattformgrenzen geschärft | OpenAI / Codex |
| 1.6     | 16.04.2026 | Docker als Entwicklungs- und Integrationsrandbedingung ergänzt | OpenAI / Codex |
| 1.7     | 16.04.2026 | MAUI-Plattformannahmen für Linux-Entwicklung präzisiert | OpenAI / Codex |
| 1.8     | 16.04.2026 | gRPC-Service als zusätzliche Schnittstelle ergänzt | OpenAI / Codex |

---

# 1 Einleitung

## 1.1 Zweck des Dokuments

Dieses Lastenheft beschreibt die fachlichen und organisatorischen Anforderungen an das System **d-browser**.
Es dient als Grundlage für Abstimmung, Planung, Umsetzung und spätere Abnahme.

## 1.2 Ziel des Vorhabens

Mit `d-browser` soll ein generischer Datenbank-Browser für relationale Datenquellen geschaffen werden.
Das System soll relationale Strukturen aus unterschiedlichen Quellen erfassen, in ein einheitliches internes Modell überführen und als hierarchisch navigierbare Struktur bereitstellen.

## 1.3 Abgrenzung

`d-browser` ist in der ersten Ausbaustufe:

* kein Migrationstool
* kein ORM
* kein vollwertiger SQL-Editor
* kein Ersatz für DB-spezifische Admin-Werkzeuge
* keine vollständig produktionsreife Endanwenderoberfläche

Der Schwerpunkt liegt auf **Analyse, Navigation, Darstellung und Export** relationaler Strukturen.
Referenz-UIs im Monorepo dienen in der ersten Ausbaustufe primär Demonstration, Integration und technische Validierung.

---

# 2 Ausgangssituation

Relationale Datenbanken bilden Informationen in Tabellen, Spalten, Primärschlüsseln und Fremdschlüsseln ab.
Für eine technische Analyse oder Browser-Oberfläche ist dieses Modell nur bedingt direkt nutzbar, weil:

* Beziehungen graphartig und nicht rein baumförmig sind
* Rekursionen und Zyklen auftreten können
* n:m-Beziehungen nicht direkt als Baum darstellbar sind
* verschiedene Datenquellen unterschiedliche Metadatenmodelle mitbringen

Es besteht Bedarf an einem Werkzeug, das relationale Modelle quellenunabhängig analysiert und in eine kontrolliert navigierbare Struktur überführt.

---

# 3 Zielbestimmung

## 3.1 Muss-Ziele

`d-browser` muss:

1. relationale Schemata aus unterschiedlichen Quellen erfassen können
2. ein kanonisches internes Metamodell verwenden
3. Fremdschlüsselstrukturen in Tree-Nodes projizieren können
4. Datensätze mit Beziehungen navigierbar machen
5. JSON und YAML als Ausgabeformate unterstützen
6. zyklische Beziehungen kontrolliert behandeln
7. per Adapter um neue Quellen erweiterbar sein
8. die Integration von `d-migrate` ermöglichen
9. als Monorepo strukturiert sein, das Kernmodule, Adapter, Service-Komponenten und Referenz-Clients sauber trennt
10. Referenz-Clients in Flutter, MAUI und SvelteKit enthalten
11. die für `d-browser` benötigten Bibliotheken so strukturieren, versionieren und bauen, dass eine Veröffentlichung über Maven Central möglich ist

## 3.2 Soll-Ziele

`d-browser` soll:

1. mehrere relationale Datenquellen unterstützen
2. als Service über REST- und gRPC-Schnittstellen verfügbar sein
3. Pagination und Lazy Loading bereitstellen
4. n:m-Beziehungen gesondert behandeln
5. über eine stabile API in unterschiedliche UI-Technologien integrierbar sein

## 3.3 Kann-Ziele

`d-browser` kann später erweitert werden um:

* weitere produktive Frontends
* GraphQL-Schnittstelle
* grafische Beziehungsvisualisierung
* erweiterte Suche
* rollenbasierte Zugriffskonzepte

---

# 4 Produkteinsatz

## 4.1 Anwendungsbereiche

Das Produkt soll eingesetzt werden für:

* Analyse relationaler Datenmodelle
* technische Exploration von Datenbeständen
* Navigation über FK-Beziehungen
* Sichtbarmachung von Schema- und Datensatzstrukturen
* Export technischer Baumdarstellungen
* Unterstützung bei Integrations-, Reverse-Engineering- und Migrationsaufgaben
* Referenzimplementierung für unterschiedliche UI-Clients

## 4.2 Zielgruppen

* Softwareentwickler
* Datenbankadministratoren
* Plattform- und DevOps-Teams
* technische Analysten
* Integrations- und Migrationsteams
* Frontend- und Client-Entwickler

---

# 5 Produktübersicht

`d-browser` ist ein System zur quellenunabhängigen Darstellung relationaler Modelle.
Es besteht fachlich aus folgenden Kernbereichen:

1. **Quellenanbindung**
   Einlesen von Metadaten und Daten aus verschiedenen Quellen

2. **Kanonisches Metamodell**
   Vereinheitlichung aller eingehenden Strukturen

3. **Tree-Projektion**
   Umwandlung relationaler Graphstrukturen in kontrollierte Navigationsbäume

4. **Ausgabe / Bereitstellung**
   Bereitstellung über JSON, YAML, CLI sowie REST- und gRPC-basierte Ansteuerung

5. **Referenz-Clients**
   Beispielhafte UIs in Flutter, MAUI und SvelteKit zur Demonstration und Validierung der Schnittstellen

---

# 6 Systemkontext

## 6.1 Eingangsquellen

`d-browser` soll Daten aus folgenden Quelltypen verarbeiten können:

* relationale Datenbanken über JDBC
* durch `d-migrate` gelieferte Schema- oder Exportdaten
* YAML-Dateien mit relationaler Struktur
* JSON-Dateien mit relationaler Struktur

## 6.2 Bereitstellungs- und Ausgabeschnittstellen

`d-browser` soll Informationen ausgeben über:

* JSON
* YAML
* CLI
* REST-API
* gRPC-Service

## 6.3 Konsumenten und Referenz-Clients

Die Referenz-Clients in Flutter, MAUI und SvelteKit sind Konsumenten definierter Systemfunktionen und Schnittstellen.
Sie gehören zum Projektzuschnitt des Monorepos, sind jedoch keine Ausgabeschnittstellen des Kernsystems.

Produktionsreife Endanwenderoberflächen sind nicht Bestandteil der ersten Ausbaustufe.

---

# 7 Fachliche Anforderungen

## 7.1 Schema und Metadaten

**LF-001** Das System muss relationale Schemata aus unterstützten Quellen erfassen können.

**LF-002** Das System muss mindestens folgende Strukturelemente fachlich abbilden können:

* Datenquelle
* Schema
* Tabelle
* Spalte
* Primärschlüssel
* Fremdschlüssel

**LF-003** Zusätzliche Strukturen wie Indizes oder Constraints sollen ohne grundlegenden Umbau der Kernlogik ergänzbar sein.

## 7.2 Kanonisches fachliches Modell

**LF-004** Alle unterstützten Quellen müssen in ein gemeinsames fachliches Modell überführt werden.

**LF-005** Die zentrale Fachlogik darf nicht direkt von einem spezifischen Datenbanksystem oder einem quellspezifischen Toolmodell abhängen.

## 7.3 Baumprojektion und Navigation

**LF-006** Das System muss relationale Informationen als hierarchische Navigationsstruktur darstellen können.

**LF-007** Dabei muss fachlich zwischen mindestens folgenden Knotentypen unterschieden werden können:

* Schema-Knoten
* Tabellen-Knoten
* Datensatz-Knoten
* Feld-Knoten
* Referenz-Knoten
* Collection-Knoten

**LF-008** Das System muss einzelne Datensätze anhand definierter Schlüssel laden können.

**LF-009** Das System muss für geladene Datensätze direkte Feldwerte darstellen können.

**LF-010** Das System muss für geladene Datensätze ausgehende Fremdschlüsselbeziehungen darstellen können.

**LF-011** Das System muss für geladene Datensätze eingehende Fremdschlüsselbeziehungen darstellen können.

**LF-012** Das System muss Sammlungen abhängiger Datensätze darstellen können.

**LF-013** Bereits besuchte Knoten müssen in der Navigation als Referenz oder Wiederverweis darstellbar sein.

## 7.4 Beziehungen, Zyklen und Laststeuerung

**LF-014** Das System muss fachlich zwischen `to-one`-Beziehungen unterscheiden können.

**LF-015** Das System muss fachlich zwischen `to-many`-Beziehungen unterscheiden können.

**LF-016** Das System muss indirekte n:m-Beziehungen über Join-Tabellen behandeln können.

**LF-017** Bei rekursiven oder zyklischen Beziehungen darf keine Endlosrekursion entstehen.

**LF-018** Bereits besuchte Knoten müssen erkannt und entsprechend gekennzeichnet werden.

**LF-019** Das System muss große Datenmengen kontrolliert verarbeiten können und darf Beziehungen oder Knoten nicht grundsätzlich vollständig vorladen.

## 7.5 Export, Integration und Referenz-Clients

**LF-020** Das System muss Baumstrukturen als JSON exportieren können.

**LF-021** Das System muss Baumstrukturen als YAML exportieren können.

**LF-022** Das System muss von `d-migrate` bereitgestellte Schema- oder Exportinformationen als Quelle nutzen können.

**LF-023** Referenz-Clients müssen definierte Systemfunktionen über dieselben freigegebenen Schnittstellen oder Services konsumieren.

**LF-024** Jeder Referenz-Client muss mindestens die Anzeige einer Schemaübersicht demonstrieren können.

**LF-025** Jeder Referenz-Client muss mindestens das Laden einer Datensatz- oder Tabellenansicht über eine definierte Schnittstelle oder einen definierten Service demonstrieren können.

**LF-026** Servicebasierte Integrationen sollen wahlweise über REST oder gRPC angebunden werden können.

---

# 8 Technische Anforderungen

## 8.1 Trennung und Schnittstellen

**LT-001** Die Lösung muss Fachlogik, Infrastruktur, Bereitstellungsschnittstellen und Client-spezifische Belange technisch klar voneinander trennen.

**LT-002** Externe Datenquellen und Ausgabeformate müssen über dokumentierte technische Schnittstellen angebunden werden.

**LT-003** Technische Einstiegspunkte wie CLI, REST-Service oder gRPC-Service dürfen die Fachlogik nicht duplizieren.

**LT-004** REST- und gRPC-Schnittstellen müssen auf denselben fachlichen Anwendungsfällen aufsetzen können.

## 8.2 Erweiterbarkeit und Testbarkeit

**LT-005** Neue Quellen oder Ausgabeformate müssen integrierbar sein, ohne die fachliche Kernlogik grundlegend umzubauen.

**LT-006** Die Logik zur Projektion und Navigation relationaler Strukturen muss isoliert testbar sein.

**LT-007** Fehler beim Laden, Mappen, Navigieren oder Exportieren müssen nachvollziehbar und reproduzierbar gemeldet werden.

## 8.3 Veröffentlichbare Bibliotheken

**LT-008** Die dafür vorgesehenen Bibliotheken von `d-browser` müssen so gebaut und versioniert werden, dass sie über Maven Central veröffentlicht und als reguläre Maven-/Gradle-Abhängigkeiten eingebunden werden können.

**LT-009** Veröffentlichbare Bibliotheken müssen reproduzierbare Build-Artefakte, versionierte Releases, POM-Metadaten sowie Source- und Javadoc-Artefakte bereitstellen.

**LT-010** Veröffentlichbare Bibliotheken müssen über eindeutige Group- und Artifact-IDs verfügen.

**LT-011** Veröffentlichbare Bibliotheken und nicht veröffentlichbare Beispielanwendungen müssen build- und paketierungstechnisch sauber voneinander getrennt sein.

## 8.4 Monorepo und Multi-Client-Struktur

**LT-012** Das Projekt muss als Monorepo aufgebaut sein.

**LT-013** Das Monorepo muss veröffentlichbare Bibliotheken, Adapter, technische Einstiegspunkte und Referenz-Clients organisatorisch und build-technisch klar trennen.

**LT-014** Referenz-Clients dürfen die fachliche Kernlogik nicht duplizieren.

**LT-015** Gemeinsame Build-, Test- und Versionsmechanismen müssen die koordinierte Entwicklung mehrerer Toolchains im Monorepo unterstützen.

---

# 9 Anforderungen an die d-migrate-Integration

## 9.1 Integrationsanforderungen

**LI-001** `d-browser` muss die Integration von `d-migrate` als Datenquelle unterstützen.

**LI-002** `d-migrate` muss mindestens als Quelle für Schema-Metadaten nutzbar sein.

**LI-003** `d-migrate` soll zusätzlich als Quelle für Exportdaten oder vorverarbeitete YAML-/JSON-Strukturen nutzbar sein.

**LI-004** Die Integration mit `d-migrate` darf nicht zu einer Kopplung der Fachlogik an interne `d-migrate`-Toolmodelle führen.

**LI-005** Die Integration mit `d-migrate` muss über eine dedizierte und dokumentierte Integrationsschnittstelle oder einen dedizierten Integrationsadapter erfolgen.

---

# 10 Nicht-Funktionsanforderungen

## 10.1 Qualitätsanforderungen

**LN-001** Das System muss klar gegliedert und nachvollziehbar aufgebaut sein.

**LN-002** Das System muss so ausgelegt sein, dass weitere Quellsysteme oder Ausgabeformate mit vertretbarem Aufwand ergänzt werden können.

**LN-003** Zur Vermeidung unnötiger Last müssen Lazy Loading, Pagination, begrenzte Rekursion und selektive Datennachladung unterstützt werden.

**LN-004** Das System muss unvollständige oder fehlerhafte Metadaten kontrolliert behandeln können.

**LN-005** Jeder Knoten der Baumstruktur soll eindeutig identifizierbar sein.

**LN-006** Fachlogik und Integrationspunkte sollen automatisiert testbar und stabil verifizierbar sein.

**LN-007** Änderungen an Kernmodulen, Schnittstellen und Referenz-Clients müssen im Monorepo nachvollziehbar koordiniert werden können.

## 10.2 Plattform- und Toolchain-Anforderungen

**LN-008** Kernsystem, Bibliotheken, CLI und Services müssen in Linux-Umgebungen entwickelbar und lauffähig sein.

**LN-009** Für Referenz-Clients und deren CI dürfen zusätzliche Plattformen oder Runner eingesetzt werden, wenn dies durch die jeweilige Toolchain erforderlich ist.

**LN-010** Für .NET-MAUI-bezogene Build-, Test- oder Validierungsschritte müssen geeignete Umgebungen vorgesehen werden. Dabei können Linux-basierte Ansätze einschließlich Community-Lösungen genutzt werden, sofern sie für den vorgesehenen Entwicklungs- oder Validierungszweck ausreichend belastbar sind.

**LN-011** Für reproduzierbare Entwicklungs-, Test- und Integrationsumgebungen soll Docker eingesetzt werden.

---

# 11 Randbedingungen

Für das Projekt gelten folgende Randbedingungen:

* Kern-, Service- und veröffentlichbare Bibliotheksmodule werden in Kotlin implementiert.
* Kernsystem, CLI und Services müssen in Linux-Umgebungen entwickelbar und lauffähig sein.
* Für reproduzierbare Entwicklungs-, Test- und Integrationsumgebungen wird Docker eingesetzt.
* Containerfähiger Betrieb soll möglich sein
* JSON und YAML sind verpflichtende Ausgabeformate
* Die erste Ausbaustufe konzentriert sich auf relationale Kernkonzepte
* Das Projekt wird als Monorepo geführt.
* Im Monorepo dürfen zusätzliche Technologien für Referenz-Clients eingesetzt werden.
* Referenz-Clients in Flutter, MAUI und SvelteKit sind Bestandteil des Repositories.
* Build- und CI-Struktur müssen die benötigten Toolchains für Kotlin, Flutter, .NET MAUI und SvelteKit unterstützen.
* Für Toolchains mit plattformspezifischen Anforderungen müssen geeignete Entwicklungs- oder CI-Umgebungen vorgesehen werden.
* Für MAUI-bezogene Entwicklungs- oder Validierungsschritte können Linux-basierte Community-Lösungen eingesetzt werden, sofern ihre Eignung für den konkreten Zweck sichergestellt ist.
* Veröffentlichbare `d-browser`-Bibliotheken dürfen nicht ausschließlich an lokale oder private Repository-Nutzung gebunden sein.
* Wenn Service-Schnittstellen bereitgestellt werden, sollen REST und gRPC als vorgesehene Integrationspfade unterstützt werden können.

---

# 12 Abgrenzung / Nicht-Ziele

Nicht Bestandteil der ersten Ausbaustufe sind:

* vollwertige Datenmigration
* generische SQL-Optimierung
* vollständige Unterstützung aller proprietären DB-Eigenheiten
* Business-Logik-Verständnis aus Datenstrukturen
* vollwertige Benutzer- und Rechteverwaltung
* umfassende grafische Visualisierung
* perfektes Handling schemaloser Daten in JSON-Spalten
* vollständige Feature-Parität aller Referenz-Clients

---

# 13 Anwendungsfälle

## 13.1 Schema anzeigen

Ein Benutzer möchte alle Schemas und Tabellen einer Quelle anzeigen.

**Erwartetes Ergebnis:**
Das System liefert eine strukturierte Übersicht über die vorhandenen Schemas, Tabellen und deren Basismetadaten.

## 13.2 Datensatz laden

Ein Benutzer möchte einen Datensatz anhand seines Primärschlüssels laden.

**Erwartetes Ergebnis:**
Das System liefert den Datensatz mit seinen Feldwerten sowie den relevanten Beziehungen.

## 13.3 Beziehungen navigieren

Ein Benutzer möchte von einem Datensatz zu verbundenen Datensätzen navigieren.

**Erwartetes Ergebnis:**
Das System liefert referenzierte und abhängige Knoten kontrolliert als Tree-Struktur.

## 13.4 Export nach JSON

Ein Benutzer möchte einen geladenen Struktur- oder Datensatzbaum als JSON exportieren.

## 13.5 Export nach YAML

Ein Benutzer möchte einen geladenen Struktur- oder Datensatzbaum als YAML exportieren.

## 13.6 Nutzung mit d-migrate

Ein Benutzer möchte von `d-migrate` bereitgestellte Schema- oder Exportinformationen im Browser verwenden.

## 13.7 Nutzung über Referenz-Clients

Ein Benutzer oder Entwickler möchte `d-browser` über die im Monorepo enthaltenen Referenz-Clients in Flutter, MAUI oder SvelteKit ansprechen.

**Erwartetes Ergebnis:**
Die Referenz-Clients können definierte Systemfunktionen gegen dieselben fachlichen Schnittstellen oder Services demonstrieren, ohne eigene fachliche Sonderlogik einzuführen.

## 13.8 Nutzung über gRPC

Ein Benutzer oder ein externes System möchte `d-browser` über einen gRPC-Service ansprechen.

**Erwartetes Ergebnis:**
Die relevanten Systemfunktionen sind über definierte gRPC-Endpunkte nutzbar, ohne dass fachliche Logik separat für den gRPC-Zugang implementiert werden muss.

---

# 14 Qualitätskriterien

Die Lösung gilt als qualitativ geeignet, wenn sie:

* fachlich nachvollziehbar modelliert ist
* modular erweiterbar bleibt
* gegen zyklische Strukturen robust ist
* mit großen Tabellen kontrolliert umgehen kann
* testbar aufgebaut ist
* Quellunabhängigkeit im Kern wahrt
* eine stabile und sauber abgegrenzte Bibliotheksstruktur für Wiederverwendung ermöglicht
* Referenz-Clients konsistent an denselben fachlichen Schnittstellen ausrichtet
* den Monorepo trotz mehrerer Toolchains wartbar hält

---

# 15 Abnahmekriterien

Die Abnahme ist möglich, wenn nachgewiesen ist, dass:

1. mindestens eine relationale Quelle in ein kanonisches Modell überführt werden kann
2. aus Fremdschlüsselbeziehungen eine Baumstruktur erzeugt werden kann
3. Zyklen korrekt behandelt werden
4. JSON-Export funktioniert
5. YAML-Export funktioniert
6. Kernlogik und Adapter klar getrennt sind
7. mindestens ein Adapter produktiv oder prototypisch angebunden ist
8. die Integration eines `d-migrate`-Adapters architektonisch vorgesehen ist
9. automatisierte Tests für die zentrale Tree-Logik vorhanden sind
10. die vorgesehenen `d-browser`-Bibliotheken in einer Form gebaut werden können, die für eine Veröffentlichung über Maven Central geeignet ist
11. eine Monorepo-Struktur vorhanden ist, die Kernmodule, Adapter, Service-Komponenten und Referenz-Clients klar trennt
12. Referenz-Clients in Flutter, MAUI und SvelteKit im Repository vorhanden und an definierte Schnittstellen oder Services angebunden sind
13. jeder Referenz-Client mindestens eine Schemaübersicht demonstrieren kann
14. jeder Referenz-Client mindestens eine Datensatz- oder Tabellenansicht über dieselbe definierte Schnittstelle oder denselben definierten Service demonstrieren kann
15. bei bereitgestellten Service-Schnittstellen dieselben fachlichen Funktionen über REST und gRPC konsistent zugreifbar sind

---

# 16 Risiken

## 16.1 Unterschiedliche DB-Eigenheiten

Unterschiedliche Systeme stellen Metadaten verschieden bereit.

## 16.2 Fehlende Fremdschlüssel

In realen Datenmodellen fehlen Beziehungen oft oder sind nicht sauber gepflegt.

## 16.3 Datenvolumen

Ohne Paging und Lazy Loading wird das System bei realistischen Datenmengen unbrauchbar.

## 16.4 Überdehnung des Projektumfangs

Der Versuch, jede Spezialfunktion jeder Datenbank sofort abzudecken, gefährdet Wartbarkeit und Lieferfähigkeit.

## 16.5 Kopplung an Fremdmodelle

Eine direkte Abhängigkeit von externen Tool- oder Quellmodellen würde die Kernarchitektur schwächen.

## 16.6 Toolchain-Komplexität im Monorepo

Die Kombination aus Kotlin, Flutter, .NET MAUI und SvelteKit erhöht Build-, Test- und CI-Komplexität.
Ohne klare Modulgrenzen und standardisierte Schnittstellen steigt der Wartungsaufwand unverhältnismäßig an.

---

# 17 Erfolgsfaktoren

Wesentliche Erfolgsfaktoren des Projekts sind:

* ein stabiles kanonisches Metamodell
* klare fachliche und technische Schnittstellen
* saubere Tree-Projektion mit Zyklenschutz
* kontrollierter Umgang mit großen Datenmengen
* bewusste Begrenzung des Scopes in der ersten Ausbaustufe
* veröffentlichbare, klar abgegrenzte Bibliotheksmodule mit stabiler API
* disziplinierte Monorepo-Struktur mit sauber abgegrenzten Referenz-Clients

---

# 18 Zusammenfassung

`d-browser` soll ein generischer, adapterfähiger Datenbank-Browser für relationale Quellen werden.
Das System soll unterschiedliche Quellen in ein gemeinsames Modell überführen und daraus navigierbare Baumstrukturen erzeugen.

Die erste Ausbaustufe konzentriert sich auf:

* relationale Kernkonzepte
* Fremdschlüssel-Navigation
* JSON/YAML-Ausgabe
* saubere Trennung von Fachlogik, Infrastruktur und Clients
* Integrationsfähigkeit für `d-migrate`
* veröffentlichbare Bibliotheksmodule für externe Einbindung
* Monorepo-Struktur mit Referenz-UIs in Flutter, MAUI und SvelteKit
* REST- und gRPC-basierte Service-Ansteuerung

Damit entsteht eine belastbare Grundlage für einen technischen Browser-Service, der später um API, UI und weitere Quellen erweitert werden kann.

---

# 19 Freigabe

## 19.1 Prüfung

| Rolle              | Name | Datum | Unterschrift |
| ------------------ | ---- | ----- | ------------ |
| Fachliche Prüfung  |      |       |              |
| Technische Prüfung |      |       |              |

## 19.2 Freigabe

| Rolle                | Name | Datum | Unterschrift |
| -------------------- | ---- | ----- | ------------ |
| Auftraggeber         |      |       |              |
| Projektverantwortung |      |       |              |

---

# 20 Anlagen

* Architektur- und Strukturentwurf in [design.md](design.md)
* Entwurf des kanonischen Domänenmodells
* Schnittstellenentwurf für Adapter und Use Cases
* Konzept zur `d-migrate`-Integration
* Exportbeispiele JSON / YAML
