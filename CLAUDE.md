# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Marquez is an open source metadata service for collection, aggregation, and visualization of a data ecosystem's metadata. It implements the [OpenLineage](https://openlineage.io) standard and tracks dataset/job/run lineage.

## Modules

- **`api/`** – Core Java/Dropwizard HTTP API server (main application)
- **`web/`** – React/TypeScript web UI
- **`clients/java/`** – Java client library implementing the HTTP API
- **`clients/python/`** – Python client library
- **`chart/`** – Helm chart for Kubernetes deployment

## Build & Run Commands

### API (Java)

```bash
# Build entire project
./gradlew build

# Run the API server (requires marquez.yml config)
./gradlew :api:runShadow

# Build shadow jar (output: api/build/libs/)
./gradlew :api:shadowJar
```

### Tests

```bash
# Run all tests
./gradlew test

# Run specific test class
./gradlew :api:test --tests marquez.api.OpenLineageResourceTest

# Run by category
./gradlew :api:testUnit         # unit tests only (tagged UnitTests)
./gradlew :api:testIntegration  # integration tests only (tagged IntegrationTests)
./gradlew :api:testDataAccess   # data access tests only (tagged DataAccessTests)
```

### Code Formatting

```bash
# Apply Google Java Format (required before pushing)
./gradlew spotlessApply

# Check formatting without applying
./gradlew spotlessJavaCheck
```

### Web UI

```bash
cd web
npm install
npm run dev      # dev server with hot reload
npm run build    # production build
npm test         # run Jest tests
npm run eslint-fix  # fix lint issues
```

### Docker

```bash
# Start all services (API + DB + Web)
./docker/up.sh

# With options
./docker/up.sh --build --seed    # build from source and seed sample data
./docker/up.sh --api-port 9000   # use alternate port (needed on macOS)
```

## Architecture

### API Layer (`api/src/main/java/marquez/`)

- **`MarquezApp.java`** – Dropwizard application entry point; wires up all components
- **`MarquezConfig.java`** – Configuration class mapping `marquez.yml`
- **`api/`** – JAX-RS resource classes (REST endpoints): `NamespaceResource`, `DatasetResource`, `JobResource`, `RunResource`, `OpenLineageResource`, `SearchResource`, etc.
- **`service/`** – Business logic layer; one service per domain entity (`DatasetService`, `JobService`, `RunService`, `OpenLineageService`, etc.). `ServiceFactory` creates and wires services together.
- **`db/`** – Data access layer using JDBI3 with SQL Object API. DAOs map directly to PostgreSQL tables. Database migrations managed by Flyway (`db/migrations/`).
- **`common/`** – Shared models and utilities

**Request flow:** HTTP request → Resource → Service → DAO → PostgreSQL

OpenLineage events arrive via `POST /api/v1/lineage` → `OpenLineageResource` → `OpenLineageService` → multiple DAOs (persists runs, jobs, datasets, facets).

### Web UI (`web/src/`)

React + Redux + TypeScript SPA using Material UI, D3 for lineage graph visualization, and i18next for internationalization.

- **`store/`** – Redux store with sagas for async API calls
- **`routes/`** – Page-level components
- **`components/`** – Reusable UI components
- **`helpers/`** – Utility functions

### Database

PostgreSQL 14+ with Flyway migrations. Integration tests use Testcontainers to spin up a real PostgreSQL instance. Unit tests are tagged `@Tag("UnitTests")`, integration tests `@Tag("IntegrationTests")`, and data access tests `@Tag("DataAccessTests")`.

## Configuration

Copy `marquez.example.yml` → `marquez.yml` and set env vars: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`.

Default ports: `8080` (API), `8081` (admin/healthcheck). Docker quickstart uses `5000`/`5001`.

## Code Style Requirements

- Java files must use **Google Java Style** (enforced by Spotless/spotlessApply)
- No wildcard imports
- All `.java`, `.bash`, and `.py` files must include the SPDX license header:
  ```java
  /* Copyright 2018-2022 contributors to the Marquez project
   * SPDX-License-Identifier: Apache-2.0 */
  ```
- All commits must be signed off: `git commit -s`
- Uses Lombok; avoid manual getters/setters where Lombok annotations apply
