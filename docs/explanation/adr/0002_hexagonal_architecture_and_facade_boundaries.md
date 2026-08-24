---
title: "ADR 0002: Hexagonal Architecture and Facade Boundaries"
tags: ["adr", "architecture", "hexagonal", "facade"]
created_at: "2026-08-24"
last_updated_at: "2026-08-24"
---

# ADR 0002: Hexagonal Architecture and Facade Boundaries

## 1. Title
ADR 0002: Hexagonal Architecture and Facade Boundaries

---

## 2. Status
**Proposed**

---

## 3. Context

To build a standalone Telemetry and Mocking SDK that is easy to maintain, extend, and run across different developer environments (e.g. Windows desktop, Linux command line, and headless CI pipelines), we must establish clean separation between our core logic and external dependencies (such as local file writing or watching).

---

## 4. Decision

We will implement a **Ports & Adapters (Hexagonal)** architecture combined with a front-facing **Facade** pattern to define the boundaries of the SDK.

### 4.1 Boundary Classifications
The SDK components will be segregated into three distinct architectural categories:

1. **Core Domain & Ports (Technology-Agnostic Core):**
   * **Domain Models:** Pure data structures (Pydantic classes) defining the structures of events and snapshots.
   * **Ports:** Abstract contracts (Python Typing Protocols) defining data retrieval and logging capabilities without dictating how those actions are completed.
2. **Adapters (Technology-Specific Translators):**
   * **Telemetry Ingestion:** Watches and parses active file streams (implements domain parser observer ports).
   * **Testing Simulation:** Drives mock generations (implements domain writing ports).
3. **Infrastructure (External Environment):**
   * The physical operating system environment (Linux file descriptors, Windows directory boundaries) that concrete adapters interface with.

### 4.2 The Developer Facade Gateway
To ensure high developer ergonomics, we will implement the **Facade** pattern as the primary SDK entry point. 
* All internal ports, adapters, and validation engines will remain hidden from the consumer.
* The developer will interact with a single, simplified public interface (exposed in the package root) that orchestrates the underlying components.

### 4.3 Structural Manifestation in the Directory Layout
These architectural boundaries will map directly to the package folder layout:

```text
src/
└── ed_journal_sdk/
    ├── __init__.py           <-- THE FACADE (Public SDK Tooling entry point)
    ├── cli.py                <-- THE FACADE (Developer CLI entry point)
    │
    ├── domain/               <-- THE CORE DOMAIN & PORTS
    │   ├── models/           <-- Pure data models (Pydantic classes)
    │   └── ports.py          <-- Abstract Port Protocols (e.g. TelemetryWriter)
    │
    ├── telemetry/            <-- THE INGESTION ADAPTERS
    │   ├── engine.py         <-- Internal validation engine
    │   └── parser.py         <-- Concrete log parser (implements Observer Port)
    │
    └── testing/              <-- THE SIMULATION ADAPTERS
        ├── io.py             <-- Concrete File I/O (implements TelemetryWriter Port)
        ├── writer.py         <-- Simulated log writer
        └── generator.py      <-- Mock value generator
```

---

## 5. Consequences

### Positive:
* **Clean Encapsulation:** Developers only interact with the facade layer, allowing the internal architecture to change without breaking downstream code.
* **Testability:** Core validators can be fully verified in memory by injecting mock adapters that satisfy the ports, eliminating disk I/O overhead.
* **Platform Independence:** Concrete adapters handle OS differences (e.g. Windows vs Linux paths) while keeping the core domain completely portable.

### Negative:
* Introduces abstraction layers (ports and facade interfaces) which require maintaining structural boundaries.
