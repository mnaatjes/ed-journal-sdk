# Directives and Context for ed-journal-sdk Development

This document defines the structural directives and context guidelines for the Elite Dangerous Journal SDK project.

---

## 1. Purpose and Structure

* **Goal:** Build a robust, standalone Python SDK that provides schemas, validation engines, and mock log simulators for Elite Dangerous telemetry.
* **Target Version:** Adhere to Player Journal v37 (Odyssey Live client) as the primary schema baseline.

## 2. Diataxis Documentation Structure Layout
All documentation files under `docs/` must be structured into the following Diataxis directories:
* **`docs/tutorials/`**: Guided learning pathways for beginners.
* **`docs/how-to/`**: Practical, step-by-step guides (e.g. how to write custom plugin tests using the mock SDK).
* **`docs/reference/`**: Technical specifications, JSON schemas, and class reference mappings.
* **`docs/explanation/`**: High-level background concepts, design rationales, and Architectural Decision Records (ADRs).

## 3. Metadata Frontmatter Syntax
All documentation pages must include the standard YAML frontmatter metadata format:
```yaml
---
title: "Document Title"
tags: ["tag-name-1", "tag-name-2"]
created_at: "YYYY-MM-DD"
last_updated_at: "YYYY-MM-DD"
---
```

## 4. Architectural Directives
All SDK code must adhere to the following design directives:
1. **Ports & Adapters (Hexagonal) Architecture:** Segregate the codebase into a dependency-free Core Domain (`domain/`) and external, pluggable Adapters (`telemetry/`, `testing/`).
2. **Single Source of Truth:** Telemetry and event schemas must be written as **Pydantic Models** inside `domain/`. JSON Schema file exports for validation must be generated dynamically from these classes via `.model_json_schema()`.
3. **Strict vs Lenient Validation:** The validator inside `telemetry/` must support dual-severity flags. Strict validation (failing tests on schema mismatch) must be enforced in tests, while lenient validation (graceful sanitization) must be available for production runtimes.
4. **Isolate Testing Data:** Raw player log files must be kept in the git-ignored `/data/raw/` staging area. Only sanitized, anonymized test fixtures are permitted inside `/tests/data/`.

---

**Active Conversation UUID**: 854aafe7-4a26-4761-b59b-074b9c871b80
