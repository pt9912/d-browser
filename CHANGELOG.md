# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

* initial project documentation with lastenheft, design and roadmap
* architecture view in `docs/architecture.md` describing layers, ports and integration principles
* gRPC service draft in `docs/grpc.md`
* OpenAPI / REST contract draft in `docs/openapi.md` with initial endpoint outline
* reference fixtures document `docs/fixtures.md` with three prioritised fixtures for the first milestone: Pagila (PostgreSQL), Sakila (jOOQ multi-dialect) and Employees (datacharmer/test_db, volume-focused)
* repository scaffolding files: `README.md`, `CHANGELOG.md`, `.gitignore`, `.dockerignore`

### Changed

* project documentation refined with roadmap milestones, gRPC service scope and Docker-based development assumptions
* OpenAPI and gRPC drafts aligned on snake_case query/request fields, shared source addressing and one-to-one method mapping
* lastenheft section 6.1 consolidated: relational sources are reached exclusively through the `d-migrate` adapter (JDBC underneath)
* documented the coupling assumptions negotiated with the d-migrate repo (Profiling/Read-Write split, diagnostics projection, FK topo-sort scope) across architecture, design, roadmap and lastenheft section 9.2

### Fixed

* clarified platform assumptions for MAUI-related development and validation
