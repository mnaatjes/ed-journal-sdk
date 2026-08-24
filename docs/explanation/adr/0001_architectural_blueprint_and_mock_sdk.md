---
title: "ADR 0001: Repository Charter and Decoupling Rationale"
tags: ["adr", "architecture", "migration", "decoupling"]
created_at: "2026-08-20"
last_updated_at: "2026-08-24"
---

# ADR 0001: Repository Charter and Decoupling Rationale

## 1. Title
ADR 0001: Repository Charter and Decoupling Rationale

---

## 2. Status
**Accepted**

---

## 3. Context

During the initial review and refactoring phases of the main `EDMarketConnector` repository, it was determined that the planned testing SDK (responsible for schema validation, telemetry generation, and mock game client output simulation) carried significant structural complexity. 

This complexity stems from:
1. Managing schemas and examples for 250+ distinct game events.
2. Implementing schema compilers and local ETL pipelines to process testing logs.
3. Managing separate validation rules (Strict vs. Lenient) across the testing and production lifecycles.

Developing this SDK directly inside the host `EDMarketConnector` code tree would clutter the main codebase, bloat the version control history with raw telemetry files, and introduce tight coupling.

---

## 4. Decision

We will extract and develop the Elite Dangerous Journal/JSS SDK as a standalone, isolated repository (`ed-journal-sdk`).

### 4.1 Purpose of the SDK
The `ed-journal-sdk` is defined as a developer-facing toolbelt designed to:
* Act as the single source of truth for Elite Dangerous Player Journal and JSS (Journal State Snapshot) schemas.
* Provide dynamic validation engines to check game log files for schema drift.
* Expose mock generators to write simulated, compliant game telemetry, enabling headless plugin testing.

### 4.2 Decoupling Rationale
Extracting the SDK to a separate repository achieves:
* **Separation of Concerns:** EDMarketConnector remains focused on processing, displaying, and transmitting data, while the SDK manages telemetry models and test simulations.
* **Community-Wide Utility:** A standalone library can be imported by any third-party tool developer in the Elite Dangerous community to validate their own integrations, accelerating community-wide testing.
* **Isolated Testing Pipelines:** Testing telemetry and schemas can run independently without triggering host client builds.

---

## 5. Architectural Decision Index
As we build the SDK, specific architectural decisions will be detailed in sequential decision records linked below:

* **[ADR 0002: Hexagonal Architecture and Facade Boundaries](file:///home/michael/src/github.com/mnaatjes/ed-journal-sdk/docs/explanation/adr/0002_hexagonal_architecture_and_facade_boundaries.md)**: Establishes the Ports & Adapters packaging boundaries, directory structures, and the public developer gateway.

---

## 6. Consequences

### Positive:
* **Cleaner Host Codebase:** Completely removes JSON schemas, mock builders, and compiler utilities from the EDMarketConnector repository.
* **Modular Packaging:** Exposes the SDK as a standard developer package that can be imported dynamically (e.g. via editable installs during development).

### Negative:
* Requires maintaining two repositories, including synchronizing issue tracking and versions.
