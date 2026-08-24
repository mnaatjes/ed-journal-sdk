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

1. **Hexagonal Architecture (Ports & Adapters):** Segregates software into technology-agnostic *Core Domain*, abstract-boundaries, and implementation packages that interact with the physical operating environment. Isolates Business-Logic from External Technologies.

  * **Core Domain (Business Logic):** the Intellectual Center of the Software. Contains Business-Rules, Calculations, Data-Shapes, Entities.
    - Contains Business Logic and other items which are TRUE regardless of how the application is run, deployed or stored.
    - Independent of the web, filesystems, databases, or CLIs
    - **Domains:**
      * **Domain Models:** Models and Entities (Data Objects) with specific identity and validation rules (e.g. FSDJump)
      * **Value Objects:** Immutable Attributes (e.g. StellarCoordinates)
      * **Domain Services:** Functions containing calculations that don't naturally belong to a single entity (e.g. `calculate_3d_coordinates()`)
      * **Domain Exceptions:** Error definitions representing Business-Logic Violations
    - [ ] Define each of these for our SDK FIRST

  * **Abstract Boundaries (Ports):** Interfaces or Contracts defined *Strictly* inside the Core Domain layer. 
    - Define WHAT actions the system does to perform (e.g. write a string, read a byte-offset.
    - Contain ZERO implementation details
    - Defined using Python *Abstract Base Classes* ABC or *Structural Typing Protocols* (e.g. typing.protocol)
    - e.g. TelemetryWriter class: a port defining how telemetry payloads are exported

  * **Pluggable Implementation Packages (Adapters):** Concrete Classes written OUTSIDE the Core Domain layer
    - Implement Port Interfaces
    - Translate core demands into technology-specific actions
    - Fulfills the Contract defined by the Port
    - Act as *plugs* which connect the Core Layer to the Outside World
    - e.g. FileJournalIO class is the adapter implementing local file writes; the concrete implementation of the Port 

  * **Physical Operating Environment (Infrastructure):** Represents the 3rd-party Software, file-hardware, OS, DBs that Concrete Adapters talk to
    - Domain has ZERO knowledge of the Infrastructure Layer
    - Typically constructed by leveraging 3rd-party libraries or standard Python modules; e.g. `import os` or `import pathlib`
    - e.g. Linux Filesystem or Windows file locking mechanisms

  > Note: Network Logic is an Infrastructure/Adapter concern.

2. **Composition Root Pattern:** an Application Runtime pattern

  * **Parts:** Typical Composition Root contains the following
    - Bootstrapper (Gatekeeper): file or function which executes first; e.g. `src/ed_journal_sdk/__init__.py`
    - Configurator (Settings Ingestion): Reads settings, CLI arguments, env vars, to determine which adapters to run
    - Registry (Mappings): defines the rules that map Ports to concrete Adapters (e.g. binds interface to concrete class)
    - Object-Graph Composer (Constructor): Instantiates objects in-order (topology sorting: base adapters first > injecting them into domain services)

  * **Roles:**
    - Instantiate the Object Graph: Constructs all objects from the outside; instead of classes instantiating own dependencies
    - Injecting Dependencies (Wiring): Passes concrete instances into constructors of the consuming classes - satisfies Port protocols
    - Managing Resource Lifecycles: Controls resources (e.g db-connect, file I/O) are open, shared (singletons), or safely closed via **Context Managers**
      > Note: NOT limited to a single file. Distributed accross Ports and Adapters
    - Responsible for implementing and distributing the **Mediator/Bus Pattern (Event Bus):**
      1. Instantiate Event Bus mediator
      2. Inject Event Bus into adapters so they can publish and subscribe to telemetry changes

    > Note: Composition Root contains the Programmatic Entry Point AND the Execution Entry Point as part of its logic

3. **Infrastructure Delivery Pipeline:**

4. **Context Management Pattern:** in Python, resource lifecycles are professionally managed by Context Managers

  * **Centrally Orchestrated:** via the Facade which has classes/methods which implement `__enter__` and `__exit__` and scope of `with` block

  * **Contractually Distributed:** via Ports (define cleanup `close()`)

  * **Physically Distributed:** via Adapters which implements its own specific cleanup logic

5. **Mediator/Bus Pattern:** Decoupled Ports & Adapters need a way to communicate WITHOUT importing each-other


### Follow-up Questions

## API

1. **API as a Facade Contract:** in an SDK, "API" refers to the public-facing python classes, methods, functions exposed to the dev-consumer

  * **Method Signatures** are explicit Python signatures containing parameters and types and return annotations

  * **Programamtic Entry Point:** the physical representation of the API/Facade Layer
    - Location: `src/ed_journal_sdk/__init__.py` root gatekeeper; which allows for `import ed_journal_sdk`
    - Exploses public models, classes, and methods and Facade Classes
    - Developed First: used BY other things like the Execution Entry Point

  * **Execution Entry Point:** Exposes the CLI
    - Location: `src/ed_jouranl_sdk/__main__.py`
    - Module execution command `python -m ed_journal_sdk`
    - Developed Second: Uses Programmatic Entry Point

2. **API First Design:** A pattern where one designs the Public Program Interfaces (method names, parameter types, returns) BEFORE writing execution code

  - Represents the *Outer Shell* of the Hexagon
  - [ ] Implement the API/Core **Design Cycle:** 
    1. Define Core Domain models (data-structures)
    2. Define Public Program Interfaces (API Methods) that accept and run models from step 1
    3. Implement the internal adapters which do the work

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
- Are Domain Exceptions leveraged in the CI/CD pipeline? Is it appropriate to integrate Business Rules into the CI/CD pipeline in this way?
- How can we integrate and ensure Composition Root rules and practices are enforced (e.g. preventing classes from instantiating their own dependencies - see `Instantiate the Object Graph` from *Composition Root*?

