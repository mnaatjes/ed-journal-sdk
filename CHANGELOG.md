# Changelog - Elite Dangerous Journal SDK (ed-journal-sdk)

This changelog records the milestones, accepted architectural changes, and documentation additions completed during the development of the standalone `ed-journal-sdk` library. Entries are in reverse-chronological order.

---

## [2026-08-20] - Repository Scaffolding and Core Architecture Blueprint

This milestone represents the initialization of the standalone repository, configuration of development guidelines, and formatting of the baseline telemetry engine decision records.

### Added
* **ADR 0001 (Accepted):** Documented the architectural blueprint for the SDK, establishing the Ports & Adapters packaging boundaries, defining the bi-directional schema-driven `TelemetryEngine` (data-to-schema, schema-to-data, and validation), and specifying the public method contracts.
* **ADR 0002 (Accepted):** Documented the schema ingestion pipeline design rules, mapping out the automated `schema_compiler.py` workflow to compile local community JSON schemas into Pydantic models, and establishing package-level version control rules (`__journal_version__ = "37"`).
* **Development Scaffolding:**
  * Created project [README.md](file:///home/michael/src/github.com/mnaatjes/ed-journal-sdk/README.md) and [GEMINI.md](file:///home/michael/src/github.com/mnaatjes/ed-journal-sdk/GEMINI.md) outlining project directives.
