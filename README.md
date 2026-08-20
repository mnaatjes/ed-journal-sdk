# Elite: Dangerous Journal SDK (ed-journal-sdk)

This repository houses the standalone **Elite Dangerous Journal and JSS (Journal State Snapshot) SDK**. It is designed to provide developers with a single source of truth for game telemetry schemas, type-checking, data validation, and simulated game client log writing (mocking).

---

## 1. Project Scope & Purpose

* **Schema Single Source of Truth:** Exposes strongly-typed Python models (via Pydantic) matching the official Player Journal v37 specification (Odyssey Live client) and JSS snapshots.
* **Automatic JSON Schema Export:** Dynamically generates JSON Schema files from code definitions to drive validation engines.
* **Mock Simulation SDK:** Exposes `MockJournalWriter` to write valid JSONL game logs and JSS snapshot files to temporary directories, enabling community developers to test third-party plugins headlessly.
* **Telemetry Validation:** Supports Strict validation in testing pipelines and provides rules for Lenient sanitization at runtime.

---

## 2. Directory Structure

This repository follows a **Ports & Adapters (Hexagonal)** layout under the `src/` directory:

```text
src/
└── ed_journal_sdk/
    ├── __init__.py      # Package facade exporting generator, writer, and types
    ├── domain/          # Pure domain models (Pydantic models for FSDJump, Docked, Status, etc.)
    ├── telemetry/       # Core parsing and validation logic
    └── testing/         # MockJournalWriter and MockValueGenerator SDK utilities
```

---

## 3. Data Storage (ETL & Staging)

To ensure that the repository remains clean of raw developer data:
* **Staging Area (`/data/raw/`):** A root-level staging folder added to `.gitignore` where developers can place un-sanitized player log files.
* **Sanitized Test Fixtures (`/tests/data/v37/`):** Sanitized, anonymized golden test logs and snapshot files committed to version control for CI/CD pipeline verification.
