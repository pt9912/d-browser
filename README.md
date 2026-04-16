# d-browser

`d-browser` ist ein generischer Browser fuer relationale Datenquellen.
Das Projekt fuehrt Schema- und Datensatzstrukturen in ein kanonisches fachliches Modell ueber und stellt Beziehungen als navigierbare Baumstruktur bereit.

## Status

Der aktuelle Stand ist dokumentationsgetrieben.
Die fachlichen und technischen Grundlagen werden derzeit in den Projektdokumenten festgehalten.

## Dokumente

* Lastenheft: [docs/lastenheft.md](docs/lastenheft.md)
* Design: [docs/design.md](docs/design.md)
* Roadmap: [docs/roadmap.md](docs/roadmap.md)

## Geplante Schwerpunkte

* fachlicher Kern fuer Baumprojektion und Navigation relationaler Strukturen
* Integration ueber `d-migrate`
* CLI sowie REST- und gRPC-Service
* Examples in Flutter, MAUI und SvelteKit
* Docker-basierte Entwicklungs- und Integrationsumgebungen

## Repository-Zuschnitt

Das Zielbild ist ein Monorepo mit:

* Kernmodulen in Kotlin
* technischen Adaptern und Services
* Examples fuer mehrere Client-Technologien
* begleitender Dokumentation in `docs/`
