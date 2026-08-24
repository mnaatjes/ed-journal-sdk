# Notes for Elite Dangerous SDK

## SDK Definition

1. A **Library** is a collection of Helper Methods - an SDK is NOT a Library
2. **Comprehensive Platform Kit** with domain models, adapters, testing mocks, schema compilers, templates, code verification tooling

## Tooling of SDK

1. Clean documentation enforced via Diataxis practices in `docs/` directory.

2. **SDK Tooling Ecosystem:** Must support the entire lifecycle of integration

  * [ ] Define **Lifecycle of Integration:** the tools we need to accomplish the 4 main parts of the SDK: 
    - Data Acquisition (parsing) 
    - Mock Synthesis(generation) 
    - Contract Validation (verification)
    - Developer Diagnostics (debugging/CLI) - or API?

3.  **Headless Architecture:** Decouple Core Business-Logic from Network-Logic and Display-Logic

  * **Environment Agnostic:** Designed with no GUI in-mind

  * **Runtime:** the Container, Engine, Environment (python interpreter), loaded memory dependencies
    - WHERE and HOW the program runs
    - Static and Dynamic Environment which supports Execution

  * **Execution:** the active, step-by-step processing of CPU instructions and State-Changes
    - Active Process of running instructions
    - e.g. run `pytest -v` triggers the *Execution* of files in `tests/` within the Python-Runtime

4.  [ ] **Other Tooling:**

  * **Anonymization & Sanitization Utility:** Ingests raw, genuine player log-file and scrubs private data

  * **CLI Diagnostic Suite:** the interface allowing devs to quickly test schemas and files without writing any python code

  * **Dynamic Watcher & Event Dispatcher:** real-time file-watcher for log events. Also dispatches events.

  * **Schema Compiler:** Development-time script that rebuilds the Pydantic Models when a new game-version is released

  * **Speculative and Undefined Tooling:**
    - I/O Layer Tool for interacting with the filesystem (Windows and Linux)

### Architecture of the SDK

1. **Hexagonal Architecture (Ports & Adapters):** Segregates software into technology-agnostic *Core Domain*, abstract-boundaries, and implementation packages that interact with the physical operating environment
 
  * **Abstract Boundaries (Ports):** 

  * **Pluggable Implementation Packages (Adapters):**

  * **Physical Operating Environment (Infrastructure):**

### Follow-up Questions

- What is the difference between *SDK Tooling* and the code-base? They seem conceptually seperate: e.g. *Telemetry Adapter* (internal code) vs the *Dynamic Watcher* (tooling)
- IS there a difference between *Ports*, *Infrastructure*, and *Adapters*?
- It seems that SDK Tooling is the front-facing collection of methods, classes, etc - the facade?
- How are these differences manifested in the directory structure, architecture, coding patterns?

## API

- Where does an API fit in the SDK?
- Is the API meant to contain, encapsulate, and/or perform the *Network Logic* of the application?
- Can an SDK - or this SDK - be developed "API First" or around an API instead of a CLI - then a CLI can be placed on-top as a pre-packaged method of communicating with the API?
- How do the API and DI Container infrastructure interact?
- Is the API part of the Developer Experience or some-other category?

## How to prioritize Developer Experience?

- **Ergonomics** How do ergonomics translate to tooling?
- What does `self-documenting via static-types` mean?
- What kind of logging and error informing is recommended and/or expected of an SDK?

- Defining the **Public Contract**
  * Should an SDK be a collection of functions or a collection of classes?
  * Is it best to deploy an SDK with a DI Container and perhaps a domain registry also? - What other infrastructure is recommended?

- How do we design-in the ability to `maintain strict type-hinting`? Is this done via CI/CD helpers like Ruff or elsewhere in the SDK via other tooling?


### Facade Pattern (Adopted)


1. **Unified Entry Point:** A single Class gatekeeper (e.g. class JournalSDK) which exposes common workflows
  - How do devs define the workflows which flow from the gatekeeper-Class for our SDK (e.g. sdk.listen(), sdk.mock(), sdk.validate())?

- What is the difference between the `facade` and the `entry-point` or are they the same?

2. **DI Container**
  - What is meant by `Inversion of Control` and is that what a DI Container does?
  - What are `abstract ports` vs `ports` - assuming in the context of our Ports & Adapters Architecture?
  - How do we enable the developer to `inject their own, custom adapters`? Is this done via json files or another way (register() and boot() methods like in php Composer Laravel)?


3. **Tooling Orchestrator:** the ergonomic entry-gate orchestrating the internal *Ports & Adapters*
  - Serves as a Unified Interface for the developer-consumer
  - All the developer-consumer sees

### Registry/Observer Pattern

- What is this syntax `@sdk.on_event("FSDJump")` and what does it enable?
- What does it mean to allow the consumer-dev to *register* callback-functions? Is this an event bus or something else? What is implied by "callback functions"?


## CI/CD Pipeline 

- Are there CI/CD tools to ensure we are developing a *Headless Architecture*?
- Are theree CI/CD tools to aid in the development and organization of the SDK API?
