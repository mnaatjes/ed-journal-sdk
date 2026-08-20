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
We have separated the Elite Dangerous Journal/JSS SDK from the parent EDMarketConnector repository to manage its complexity independently. We need to define the architectural blueprint for this new standalone library, specifically addressing the decoupling of file I/O operations from core telemetry generation, and detailing our class structures.

---

## 4. Decision

### 4.1 Decoupling File I/O from Generation Logic
To prevent filesystem dependencies from polluting the core SDK, we will isolate file I/O from the mock generation engine. 

* **InMemoryGenerator (Core Engine):** 
  * Responsible strictly for formatting and producing data in memory.
  * Yields serialized JSONL strings, parsed dictionaries, or individual parameter values.
  * Contains no filesystem dependencies, allowing rapid memory-only test executions.
* **FileJournalWriter (I/O Adapter):**
  * Responsible for actual filesystem operations.
  * Consumes payloads from `InMemoryGenerator` and writes them as unbuffered binary appends to `Journal.log` files or overwrites snapshot files like `Market.json`.

### 4.2 Ports & Adapters Architecture
We will implement the package layout under `src/ed_journal_sdk/`:

* **Core Domain (`src/ed_journal_sdk/domain/`):**
  * Houses pure Pydantic models matching the Player Journal v37 specification.
* **Telemetry Adapter (`src/ed_journal_sdk/telemetry/`):**
  * Houses the directory watchers and validators.
* **Testing Simulator Adapter (`src/ed_journal_sdk/testing/`):**
  * Houses `InMemoryGenerator`, `FileJournalWriter`, and `MockValueGenerator`.

### 4.3 Scratch Usage Blueprint
We define the target developer usage API demonstrating the split between memory generation and file writing:

```python
from pathlib import Path
from ed_journal_sdk.testing import InMemoryGenerator, FileJournalWriter

def run_simulation() -> None:
    # 1. Initialize Generator (In-Memory Only) and File Writer (Filesystem Adapter)
    generator = InMemoryGenerator()
    writer = FileJournalWriter(output_dir=Path("./data/simulated"))

    # 2. Generate granular / primitive values
    mock_coords = generator.generate_value("FSDJump", "StarPos") # Returns [12.5, -4.2, 300.1]
    mock_jsonl = generator.generate_event_string("FSDJump", SystemName="Sol") # Returns JSONL string

    # 3. Simulate game launch using the File Writer
    writer.start_game(cmdr_name="Cmdr Jameson")

    # 4. Simulate a log event
    event_payload = generator.generate_event_payload("FSDJump")
    writer.write_event("FSDJump", event_payload)

    # 5. Simulate JSS snapshots (Dual-writes)
    market_payload = generator.generate_event_payload("Market")
    writer.trigger_market_visit(market_id=12800100, commodities_data=market_payload)

    # 6. Stop game
    writer.stop_game()

if __name__ == "__main__":
    run_simulation()
```

---

## 5. Consequences

### Positive:
* **Headless Speed:** Memory-only tests run in microseconds without triggering filesystem locks or I/O overhead.
* **Clean Boundaries:** The generator can be easily adapted to feed network sockets or terminal print statements.

### Negative:
* Slightly more boilerplate to pass payloads between the generator and the file writer.

---

## 6. Procedure & Implementation

1. **Implement Core Scaffolding:** Create `src/ed_journal_sdk/` packages.
2. **Write Scratch Example:** Create the `scratch/simulate_gameplay.py` script.
