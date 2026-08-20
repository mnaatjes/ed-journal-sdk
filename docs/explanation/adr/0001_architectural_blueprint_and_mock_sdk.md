---
title: "ADR 0001: Architectural Blueprint and Mock SDK"
tags: ["adr", "architecture", "hexagonal", "sdk"]
created_at: "2026-08-20"
last_updated_at: "2026-08-20"
---

# ADR 0001: Architectural Blueprint and Mock SDK

## 1. Title
ADR 0001: Architectural Blueprint and Mock SDK

---

## 2. Status
**Proposed**

---

## 3. Context
We have separated the Elite Dangerous Journal/JSS SDK from the parent EDMarketConnector repository to manage its complexity independently and serve the broader developer community. We need to define the architectural blueprint for this new standalone library, specifying its package layout, Ports & Adapters boundaries, core methods, and a sample usage entry point.

---

## 4. Decision

### 4.1 Ports & Adapters Architecture
We will implement the SDK using **Ports & Adapters (Hexagonal)** architecture, segregating our files under the package directory `src/ed_journal_sdk/`:

* **Core Domain (`src/ed_journal_sdk/domain/`):**
  * Contains pure, dependency-free Python types and schemas.
  * We will use **Pydantic Models** to declare schemas for all events (e.g. `FSDJump`, `Docked`) and JSS files (e.g. `Status`, `Market`).
  * Exposes no filesystem, networking, or database calls.
* **Telemetry Adapter (`src/ed_journal_sdk/telemetry/`):**
  * Implements ports for reading and validating telemetry data.
  * Houses the log file watcher, log tailer, and JSS snapshot readers.
  * Implements the **Dual-Severity Validator** (Strict mode for testing, Lenient mode for production runtimes).
* **Testing Simulator Adapter (`src/ed_journal_sdk/testing/`):**
  * Implements ports for writing simulated game client data.
  * Houses `MockJournalWriter` and `MockValueGenerator`.

### 4.2 SDK Capabilities
The SDK will provide third-party developers with three primary systems:
1. **Mock Generator:** Creates simulated `Journal.log` files and JSS snapshots to test plugin events headlessly.
2. **Schema Validator:** Checks generated JSON against the official Pydantic models to catch schema drift.
3. **Local Ingestion Helpers:** Utilities to watch directory changes and parse logs.

### 4.3 Scratch Usage Blueprint
We define the target developer usage API for the SDK. A scratch entry-point script (`scratch/simulate_gameplay.py`) will demonstrate these methods:

```python
from pathlib import Path
from ed_journal_sdk.testing import MockJournalWriter, MockValueGenerator

def run_simulation() -> None:
    # 1. Initialize the mock writer and value generator
    output_directory = Path("./data/simulated")
    writer = MockJournalWriter(output_dir=output_directory)
    generator = MockValueGenerator()

    # 2. Simulate game client launch (writes Fileheader, LoadGame, Rank, Progress)
    writer.start_game(cmdr_name="Cmdr Jameson")

    # 3. Simulate sequential game events
    # Writes event to active Journal Log
    fsd_payload = generator.generate_fsd_jump_payload(system_name="LHS 3447")
    writer.write_event("FSDJump", fsd_payload)

    # 4. Simulate a station interaction involving JSS snapshots
    # Performs a dual-write: appends Market event to log and overwrites Market.json in-place
    market_payload = generator.generate_market_snapshot(market_id=12800100)
    writer.trigger_market_visit(market_id=12800100, commodities_data=market_payload)

    # 5. Simulate closing the game client (appends Shutdown event and closes handles)
    writer.stop_game()

if __name__ == "__main__":
    run_simulation()
```

---

## 5. Consequences

### Positive:
* **Decoupled Design:** Host applications can consume this SDK as a development library without cluttering their main repositories.
* **Readable API:** The code examples establish a clear, concise method naming convention for developers to follow.

### Negative:
* None. Establishing this blueprint guarantees consistency prior to the implementation phase.

---

## 6. Procedure & Implementation

1. **Scaffold Docs:** Create the Diataxis documentation directories.
2. **Implement Core Scaffolding:** Create `src/ed_journal_sdk/` packages.
3. **Write Scratch Example:** Create the `scratch/simulate_gameplay.py` script.
