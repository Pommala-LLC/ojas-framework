# OJAS Base Framework Study

**Status:** Draft implementation spec — v0.1
**Scope:** Translation of the Spring Boot model into Python frameworks. Classification of every framework Ojas evaluated. Performance discipline derived from those choices.
**Companion to:** `OJAS_FRAMEWORK_DESIGN.md` (§3 borrow/reject split), `OJAS_MODULE_SECURITY.md` (Python-specific risks).

---

## 1. Purpose

Ojas borrows Spring Boot's packaging discipline. The Spring Boot framework as a whole has dozens of mechanisms (auto-configuration, `@ConfigurationProperties`, `@Conditional*`, `ApplicationContext`, `@Profile`, `BeanPostProcessor`, lifecycle hooks, testing support, etc.) and each one has a Python ecosystem analog or counterpart.

This document records the full study. It does not exclude any framework or pattern that was identified as relevant. It classifies each one's role for Ojas, names the performance tier each occupies, and — for patterns Ojas deliberately does not adopt — records the rejection with rationale rather than dismissing the pattern as "avoid."

The frozen rule that governs this study:

> **Ojas studies all relevant Python framework patterns that map to Spring Boot concepts. It adopts some directly, borrows some as design patterns, keeps some as optional modules, and explicitly rejects some activation behaviors while still documenting them. No framework is ignored.**

Or, more sharply:

> **Rejection without rationale is not a design decision. It is a preference.**

---

## 2. Classification model

Every framework or pattern this document discusses is classified along two orthogonal axes.

### 2.1 Role axis

| Role | Meaning |
|---|---|
| **Core dependency** | Ojas imports this library directly. Listed in `pyproject.toml`. Version compatibility tracked. |
| **Core pattern** | Ojas implements the design but not necessarily the library. May write its own version, may use an existing library, but the architectural commitment is to the pattern. |
| **Optional implementation module** | Useful for some deployments. Not required in the kernel. May be added by users without changing the framework. |
| **Future capability** | Architecturally accommodated but not implemented in v0.1. Will likely become a core pattern or core dependency in a later version. |
| **Reference framework** | Studied for design discipline. Not used directly. Cited where its discipline informed Ojas's choice. |
| **Deliberately rejected activation pattern** | Identified, evaluated, and explicitly not adopted because it conflicts with Ojas invariants. Documented because users will encounter it elsewhere and ask why Ojas is different. |

### 2.2 Performance tier axis

| Tier | Meaning |
|---|---|
| **Startup-time** | Runs once at process start. Cost amortized over the process lifetime. |
| **Activation-time** | Runs when a module is selected, validated, and activated. Per-deployment, not per-run. |
| **Runtime-boundary** | Runs at every boundary crossing (tool call, memory write, context inclusion, ledger event). Cumulative cost is real. |
| **Runtime-hot** | Runs inside the inner execution loop. Any cost here multiplies. Avoid wherever possible. |
| **CI/certification-time** | Runs only during testing or certification. No production cost. |
| **Conditional** | Tier depends on whether an optional surface is enabled (e.g., HTTP). |

The discipline behind these tiers:

> **Ojas uses base frameworks for structure, validation, discovery, and testing — but keeps the runtime hot path thin, explicit, and governed.**

---

## 3. Spring Boot concept → Python concept matrix

Every Spring Boot concept Ojas evaluated, every Python equivalent considered, and Ojas's chosen role.

| Spring Boot concept | Python concepts evaluated | Ojas adoption |
|---|---|---|
| Starters (dependency bundles) | `ojas-starter-*` packages, setuptools entry points | Core packaging mechanism |
| Auto-configuration (classpath-as-intent) | Stevedore eager loading, pluggy auto-registration, decorator-based registration, manifest-driven explicit selection | Manifest-driven selection adopted; classpath-as-intent rejected (§5.1) |
| `@ConfigurationProperties` | Pydantic Settings, Dynaconf, Hydra structured config | Pydantic Settings adopted; Dynaconf as reference; Hydra concepts borrowed |
| `@Conditional*` activation predicates | Manifest-driven explicit selection, Pydantic Settings flags, Pyramid configurator, Spring Boot-style classpath conditions | Manifest-driven adopted; classpath-style conditions rejected (§5.1) |
| `ApplicationContext` (DI container) | dependency-injector, injector, Pyramid component registry, ZCA, kernel-as-service-locator | Kernel-as-service-locator adopted; full DI containers rejected (§5.3) |
| `@Bean` factory methods | Module `on_module_activation` registering capabilities | Adopted as Django-AppConfig-style core pattern |
| Bean lifecycle (`@PostConstruct`, `@PreDestroy`) | Django `AppConfig.ready()`, FastAPI lifespan events, context managers, explicit lifecycle methods | Django-style explicit lifecycle methods adopted as core pattern |
| `BeanPostProcessor` | (No clean Python analog) | Not pursued — would conflict with no-monkey-patching pledge |
| `@Profile` (environment overlays) | Pydantic Settings, Dynaconf environments, Hydra config groups | Pydantic Settings layered overlay adopted |
| `@Autowired` ubiquity | dependency-injector, FastAPI `Depends()`, kernel-as-service-locator | Kernel-as-service-locator + per-run resolution adopted; ubiquitous DI rejected |
| SPI interfaces | `typing.Protocol`, `abc.ABC`, ZCA interfaces, pluggy hookspecs | `typing.Protocol` adopted as core dependency for SPIs; pluggy used for ledger subscribers only |
| Testing support (`@SpringBootTest`) | pytest fixtures + plugins, hypothesis, testcontainers | pytest-based conformance adopted as core dependency |
| Event listeners (`ApplicationListener`) | pluggy hooks, Django signals, Blinker/Flask signals, Celery signals | pluggy hooks for ledger subscribers adopted; signals rejected for activation |
| Backend abstraction (`*-spring-boot-starter`) | Celery backends, Airflow providers, stevedore drivers | Celery-style backend identifier adopted as core pattern |
| Runtime isolation | Ray actors, multiprocessing, sidecar / control-plane runtime | Ray-style actor isolation marked as future capability for Regulated |
| Decorator-based registration (Spring's `@Component`) | `@app.route` (Flask), `@router.get` (FastAPI), `@click.command`, `@app.task` (Celery), `@pytest.fixture` | Rejected (§5.2) — conflicts with metadata-only discovery |
| Web/HTTP surface (Spring MVC) | FastAPI, Flask, Django, Pyramid | FastAPI as optional implementation module |
| Property override via CLI | argparse + Pydantic Settings, Hydra CLI override syntax, Dynaconf | argparse + Pydantic adopted; Hydra CLI override behavior rejected for governance config |

---

## 4. Frameworks Ojas adopts

### 4.1 Core dependencies

These are imported by `ojas-core` directly. Tracked for version compatibility. Listed in `pyproject.toml`.

| Library | Role | Performance tier | Why |
|---|---|---|---|
| `importlib.metadata` (stdlib) | Module discovery | Startup-time | Standard library; metadata-only entry-point discovery; no `.load()` required |
| Pydantic v2 | Artifact validation, schema export | Runtime-boundary (only at trust boundaries) | Type-validated models, JSON Schema export, discriminated unions |
| Pydantic Settings | Module configuration | Activation-time | Layered config from env, files, secrets; type-validated |
| `typing.Protocol` (stdlib) | SPI contract definitions | Compile-time / startup-time | Structural typing without forced inheritance |
| pytest | Conformance test runner | CI/certification-time only | Mature plugin model; fixture composition; pytest-plugins for shared conformance suites |
| pluggy | Ledger event subscriber hooks **only** | Runtime-boundary (one call per ledger event per subscriber) | Hook spec/impl pattern from pytest's plugin system |

### 4.2 Core patterns (architectural commitment, library implementation may vary)

These are patterns Ojas commits to. The library implementing the pattern may be borrowed, adapted, or written from scratch — the architectural shape is what's locked.

| Pattern | Reference implementation | Performance tier | Why |
|---|---|---|---|
| Stevedore-style driver/extension manager | stevedore (OpenStack) | Startup-time / activation-time | Discovery separated from instantiation; explicit driver selection |
| Django-AppConfig-style activation hook | Django `AppConfig.ready()` | Activation-time (once per module) | Single explicit `on_module_activation(kernel)` per module; no implicit setup |
| Celery-style backend identifier | Celery broker/result backends | Activation-time | Backend referenced by stable string in manifest; switching backends is a config change |
| FastAPI-style per-request resolution | FastAPI `Depends()` | Runtime-boundary | Per-run kernel access pulled at run start; no global container |
| ZCA-style adapter pattern | `zope.interface` + `zope.component` | Activation-time | Framework adapters convert backend-native primitives to Ojas artifacts; one adapter per backend module |
| Airflow-style explicit backend contracts | Airflow executor / scheduler / DB backends | Documentation discipline | Each backend's contract is documented in code, not just in prose; conformance test verifies the contract |
| Twisted/setuptools entry-point declaration | `pyproject.toml` `[project.entry-points]` | Startup-time | Plugin metadata in package, no auto-import |

### 4.3 Optional implementation modules

These are useful for some deployments. They are not required in the kernel. They may be added by users or by Ojas-published companion packages without changing the framework's invariants.

| Library | Role | Performance tier | Where it fits |
|---|---|---|---|
| FastAPI | HTTP/control-plane/admin surfaces | Conditional (only if HTTP enabled) | Local dev console, control-plane HTTP service, admin API |
| Hydra (selective) | Hierarchical config composition | Activation-time | Borrow YAML composition with `defaults:` lists; do **not** adopt CLI override syntax |
| Dynaconf | Multi-source settings | Activation-time | Reference for environment overlay patterns; alternative if Pydantic Settings is insufficient for a deployment |

### 4.4 Future capabilities

Architecturally accommodated. Not implemented in v0.1. Each will likely become a core pattern or core dependency in a future version.

| Pattern | Reference implementation | Triggers adoption when |
|---|---|---|
| Actor / process isolation | Ray, multiprocessing, custom IPC | Regulated process isolation transport micro-spec lands |
| DI container (limited) | dependency-injector, injector | A future use case demonstrably benefits from container-based resolution that service-locator cannot satisfy |
| A2A inter-agent protocol surface | A2A spec (Google) | A2A reaches broad multi-vendor adoption |
| MCP tool result schema validation | MCP Python SDK | MCP becomes a primary tool path in production deployments |

### 4.5 Reference frameworks

Studied for design discipline. Not used directly. Cited where their discipline informed Ojas's choice.

| Framework | What Ojas studied | Influence on Ojas |
|---|---|---|
| Django | App registry, settings module, app loading order, middleware pipeline | `AppConfig.ready()` lifecycle pattern; settings.py-style explicit configuration |
| Flask | Extension pattern, application factory, blueprint registration | Explicit-wiring discipline; registration without auto-discovery |
| Pyramid | Configurator, ZCA component registry | Adapter-by-interface pattern; explicit configuration over convention |
| Zope Component Architecture | Interface/adapter registry | Generic adapter discipline borrowed conceptually |
| Celery | Broker/result backend pluggability | Backend-by-identifier discipline; long-term sustainability of multi-backend models |
| Airflow | Provider system, plugin governance, multi-tenant operations | Explicit backend contracts; provider package audit discipline |
| Prefect | Modern Python orchestration with typed contracts | Typed contract discipline at workflow boundaries |
| Twisted plugin system | Historical entry-point-based plugin discovery | Predecessor to setuptools entry points |
| dependency-injector | Provider/container DI | Reference for DI patterns; not adopted (§5.3) |
| injector | Guice-style modules and binders | Reference for explicit DI contrast with service-locator approach |
| `dependency-injector` and `injector` together | The Python DI container market | Market evidence that DI is a minority pattern in Python |
| Hypothesis | Property-based testing | Optional pairing with pytest for SPI conformance properties |
| Testcontainers | Docker-based integration test fixtures | Optional pairing for backend conformance tests requiring infrastructure |

---

## 5. Deliberately rejected activation patterns

These patterns were identified, evaluated, and not adopted. Each entry records the pattern, where it is idiomatic, why Ojas considered it, why Ojas rejected it, what Ojas uses instead, and what the rejection costs the developer.

This section is intentionally longer than the adoption sections. The reasoning is that future contributors, integrators, and reviewers will repeatedly encounter these patterns elsewhere and ask why Ojas is different. The documented rejection prevents perpetual re-litigation.

### 5.1 Spring Boot's classpath-as-intent auto-configuration

**Identified in:** Spring Boot, the dominant JVM enterprise framework. The pattern is exemplified by `@AutoConfiguration`, `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, and the `AutoConfiguration.imports` registry.
**Evaluated for Ojas:** Yes
**Status:** Deliberately not adopted

#### The pattern

Spring Boot scans the classpath at startup. When it finds a class on the classpath that matches an `@AutoConfiguration` predicate, it instantiates the corresponding beans automatically. The developer adds a starter dependency, Spring observes the classes that arrive, Spring wires the beans without the developer writing any wiring code.

The implicit assumption is: **classpath presence implies developer intent.**

#### Why Ojas considered it

- **Developer ergonomics.** Spring Boot's adoption comes substantially from this single feature. Asking developers to do explicit wiring is a tax. The market expectation in 2026 is that frameworks compose themselves from declared dependencies.
- **The Spring Boot analogy.** Ojas borrows Spring Boot's package layout — modules, starters, SPIs, dependency bundles. A reader will reasonably assume Ojas borrows the activation behavior too. Not adopting it makes the analogy partial and requires explanation.
- **Python equivalents exist.** `importlib.metadata` plus entry points plus a Python rewrite of `@ConditionalOn*` predicates could implement classpath-as-intent in Python. Stevedore's `invoke_on_load=True` is a partial implementation of the same pattern.

#### Why Ojas rejects it

**Conflict with the master invariant.** The Ojas master invariant says capability does not create authority. Auto-configuration converts capability — the presence of a class on the classpath — directly into operational behavior. Installing a package causes it to run. This is the inverse of what Ojas asks of every agent, tool, and memory backend it governs.

If Ojas itself auto-configured, the framework's own discipline would contradict its governance discipline. An agent author reading "installation does not create authority for tools" while observing the framework auto-activate any installed package would correctly conclude that the rule is inconsistent.

**Python-specific risk: import side effects.** In Python, importing a module runs its top-level code. Spring's `@ConditionalOnClass` is safe because Java classes don't execute on load. Python modules do. Auto-configuration in Python would mean running arbitrary code at startup based on what is installed, which is the supply-chain attack surface that the no-monkey-patching pledge and the metadata-only discovery rule are designed to prevent.

#### What Ojas uses instead

Manifest-driven explicit selection, validated by a registry, gated by certification, recorded in the ledger.

- Discovery uses `importlib.metadata` for metadata only. No `.load()` until activation.
- Activation requires a manifest entry naming the module, a registry entry approving it, a descriptor validating it, a variant eligibility check confirming it, and a certification check where required.
- Each step is a recorded ledger event.

#### Developer impact

A developer coming from Spring Boot will have to write a manifest. The manifest is to Ojas what the classpath was to Spring Boot — but it is an explicit, signed, validatable document, not an implicit consequence of dependencies.

The framing is: *in Spring Boot the classpath is the manifest; in Ojas the manifest is the manifest.*

For Open variants the friction is small (metadata discovery plus a manifest entry). For Regulated variants the friction is the point — every activation is a governance event with evidence.

---

### 5.2 Decorator-based registration that runs at import time

**Identified in:** Flask (`@app.route`), FastAPI (`@router.get`), Click (`@click.command`), Celery (`@app.task`), pytest (`@pytest.fixture`), pluggy (`@hookimpl`). The dominant Python registration pattern.
**Evaluated for Ojas:** Yes
**Status:** Deliberately not adopted

#### The pattern

A decorator at module top level registers a function, route, hook, task, or plugin into a global registry as a side effect of importing the module:

```python
@app.route("/login")              # Flask
def login(): ...

@router.get("/users")             # FastAPI
def list_users(): ...

@app.task                         # Celery
def send_email(to, body): ...

@pytest.fixture                   # pytest
def db_connection(): ...
```

The decorator runs when the module is imported. The registration is the side effect.

#### Why Ojas considered it

- **Maximum ergonomics.** Decorator registration is the lowest-friction way to expose a hook in Python. Anything more verbose loses the developer.
- **It composes with entry points.** A module could declare an entry point in `pyproject.toml`; importing the entry-point target would run the decorators; registration would happen.
- **It composes with `typing.Protocol`.** The decorator could even validate at registration time that the decorated callable satisfies a Protocol.

There were genuine reasons to consider it. Rejecting it without acknowledging why would be dishonest about the design space.

#### Why Ojas rejects it

**Import side effects break metadata-only discovery.** The discovery rule is explicit: Ojas MUST NOT call `entry_point.load()` during discovery. The reason is that `.load()` imports the module, and importing the module runs whatever the module's top-level code chooses to run. A malicious or buggy module could:

- monkey-patch the kernel during its own import
- register hooks before the registry has approved the module
- contact external services
- exfiltrate secrets from environment variables
- modify other modules' state

Decorator-based registration *requires* import-time execution. If Ojas accepted the pattern, it would have to call `.load()` during discovery, which is the exact behavior the security model prohibits.

**Decorator registration creates an audit gap.** The registration happens during import — before the manifest has been consulted, before the registry has been queried, before variant eligibility has been checked, before the descriptor has been validated. The order is wrong. By the time Ojas applies its activation discipline, the module has already registered itself.

Even if the registration is technically caught later (e.g., by an idempotent registry), the *act* of self-registration is a non-ledgered event. There is no `MODULE_REGISTRATION_REQUESTED` because there was no request. The module just did it.

#### What Ojas uses instead

Explicit registration via the SPI lifecycle hook.

A backend module implements `OjasModule.on_module_activation(kernel)`. The kernel calls this method *after* discovery, descriptor validation, registry trust, variant eligibility, and certification have all passed. Inside `on_module_activation`, the module can register its tools, hooks, or capabilities — but only at the point Ojas has authorized the module to do so.

#### Developer impact

The Python idiom most developers reach for first is unavailable. The substitute is more verbose. A backend author cannot write:

```python
@ojas.tool("invoice_lookup")       # NOT supported
def invoice_lookup(invoice_id): ...
```

They have to write:

```python
class InvoiceTool:
    tool_id = "invoice_lookup"
    declared_schema = {...}

    async def on_module_activation(self, kernel):
        kernel.tool_broker.register(self)

    async def invoke(self, scope, args):
        ...
```

The framing is: *Ojas asks for explicit lifecycle methods because decorator-time registration cannot be validated against the registry. The verbose form is more typing. The verbosity is the audit trail.*

---

### 5.3 Hidden DI containers with magical wiring

**Identified in:** Spring (`ApplicationContext` + `@Autowired`), Jakarta EE, ASP.NET Core. In Python: `dependency-injector`, `injector`, `lagom`, `punq`. A minority pattern in Python; the dominant pattern in JVM and .NET enterprise development.
**Evaluated for Ojas:** Yes
**Status:** Deliberately not adopted in v0.1; possible future consideration if a use case demonstrates need

#### The pattern

A dependency-injection container is an object that maintains a registry of providers (factories, classes, singletons) and resolves dependencies between them automatically. When a class is asked for, the container constructs it, recursively constructs its dependencies, and returns a fully-wired object — usually invisibly to the calling code.

The "magic" is that the wiring is invisible. The developer doesn't see when objects are constructed, in what order, or with what dependencies — the container handles it.

#### Why Ojas considered it

- **Spring developers expect it.** A reader coming from Spring Boot will look for the equivalent of `ApplicationContext` and `@Autowired`. Not having one creates a friction point and an explanation requirement.
- **It composes well with Protocol-based contracts.** A DI container with `typing.Protocol`-based registration could implement clean dependency resolution by interface.
- **It centralizes lifecycle.** The container becomes the single place where activation order, lifecycle hooks, and dependency resolution happen.

These are real benefits.

#### Why Ojas rejects it

**Wiring is not auditable in the way governance requires.** A DI container resolves dependencies through reflection or registration metadata. The actual order of construction, the actual dependencies pulled, the actual factory chosen — these are emergent properties of the container's resolution algorithm, not explicit decisions in code. For ordinary application code this is convenient. For a framework whose core promise is "every authority decision is recorded," it is the wrong abstraction.

**The activation point becomes implicit.** When the kernel needs a registered backend, it would ask the container. The container would construct the backend, possibly with cached singletons, possibly with new instances per request, possibly through proxies. The Ojas activation discipline says: manifest selects, registry verifies, variant checks, certification confirms, then the module activates — and each step emits a ledger event. A DI container would resolve the backend without going through this chain unless every container interaction was wrapped in activation logic. At which point the container is no longer doing the work; the activation logic is.

**Monkey-patching is hard to prevent through a container.** A module that registers a provider for the `Kernel` interface can replace the kernel for any consumer that asks the container for it. The no-monkey-patching pledge is enforceable in a service-locator model — the kernel is one object, asked for by name, immutable from module code. In a DI container, the kernel becomes whatever the container decides to provide, which is a different and weaker guarantee.

#### What Ojas uses instead

Kernel-as-service-locator with explicit lifecycle methods.

The kernel is a single object passed to each module's `on_module_activation`. The module holds the reference and pulls services from it as needed:

```python
async def on_module_activation(self, kernel):
    self._kernel = kernel
    self._ledger = kernel.ledger
    self._registry = kernel.registry
```

Every dependency is a method call on a known object. The kernel knows what was pulled. Lifecycle is one method per phase, called by the kernel at known points.

This is closer to Django's `AppConfig.ready()` or FastAPI's startup events than to Spring's `ApplicationContext`. It is recognizably Python.

#### Developer impact

A Spring developer will not find the equivalent of `@Autowired`. They will be slightly disappointed.

The framing is: *the alternative is more explicit, not less. Every dependency a module uses is a method call on a known object, not a hidden resolution from a container.*

---

### 5.4 Other rejected patterns (briefer)

The three patterns above are the load-bearing rejections. A handful of smaller patterns are also explicitly not adopted:

| Pattern | Why rejected |
|---|---|
| `pkg_resources` discovery | Slow and deprecated. Use `importlib.metadata` directly. |
| `entry_points(...).load()` during discovery | Loads code at discovery time, defeating metadata-only discovery. Always use `invoke_on_load=False` when adapting stevedore patterns. |
| Django `INSTALLED_APPS` global mutable state | Hidden global mutable state with implicit ordering. Ojas's manifest is explicit and validated. |
| Flask `g` (request-global state) | Hidden mutable state with unclear lifecycle. Ojas's per-run context object is explicit and immutable. |
| Hydra CLI override syntax | Allows arbitrary config mutation from the command line. Conflicts with manifest-as-source-of-truth for governance config. |
| Property flags as authority toggles (Spring `application.properties`) | Configuration cannot create authority. Ojas separates manifest (governs) from configuration (supplies values). |

Each of these is a pattern users will encounter in other Python frameworks and may propose for Ojas. Each is rejected for a stated reason.

---

## 6. Performance discipline

The base frameworks listed above will not materially hurt Ojas performance if used at the correct tier. The real runtime performance risks are Ojas-specific design choices, not framework choices.

### 6.1 Performance tier discipline

| Tier | What runs here | Discipline |
|---|---|---|
| **Startup-time** | `importlib.metadata` discovery, descriptor reading, registry lookups | Cost amortized over process lifetime. Optimize for correctness, not speed. |
| **Activation-time** | Pydantic Settings resolution, Protocol validation, certification checks, module `on_module_activation` | Once per module per deployment. Cost is acceptable. |
| **Runtime-boundary** | Pydantic validation of artifacts crossing SPI boundaries, ledger event emission, pluggy hook fan-out | Real cumulative cost. Use only at boundaries, not for internal objects. |
| **Runtime-hot** | Inside the inner execution loop | Minimize Pydantic validation here. Avoid ledger emission per token. Avoid pluggy hooks per step. |
| **CI/certification-time** | pytest conformance, hypothesis property tests, testcontainers fixtures | No production cost. |

### 6.2 Ojas-specific performance risks

These are not framework problems. They are design choices Ojas could make poorly.

| Risk | Mitigation |
|---|---|
| Synchronous governance per execution step | Batch governance decisions where possible; cache validated artifacts; defer non-critical checks |
| Ledger subscribers blocking execution | Critical ledger writes are kernel-owned and synchronous; secondary subscribers (SIEM, GRC) run async or out-of-band |
| Repeated Pydantic validation of the same artifact | Validate at boundary entry; pass validated objects within the kernel without re-validation |
| Synchronous memory admission for every write | Admit asynchronously where the variant permits; batch admission decisions for high-volume writes |
| Process isolation for every Open/Standard run | Process isolation is required only for Regulated; Open and Standard run in-process by default |
| `isinstance(obj, Protocol)` checks per call | Use Protocols for static typing; runtime checks at activation only, not per invocation |
| Pluggy hookspec fan-out on hot paths | Pluggy is for ledger event subscribers, not for per-step governance |

### 6.3 The frozen performance rule

> **Use base frameworks for structure, validation, discovery, and testing — but keep the Ojas runtime hot path thin, explicit, and governed.**

Concretely:

- Pydantic at boundaries, not interiors
- pluggy for ledger subscribers, not for per-step hooks
- Protocol for typing, not for repeated runtime checks
- Discovery once at startup, never in the runtime loop
- Activation once per deployment, never per run

---

## 7. Python-specific risks (cross-reference)

The base-framework choices interact with Python-specific risks documented in `OJAS_MODULE_SECURITY.md`. This document does not duplicate that material; the cross-reference table below maps the risks to the base-framework decisions that mitigate them.

| Python-specific risk | Mitigated by |
|---|---|
| Import side effects | Metadata-only discovery; `invoke_on_load=False`; rejection of decorator-time registration (§5.2) |
| Monkey-patching | Registry trust pledge; service-locator kernel access; rejection of DI containers (§5.3) |
| Async/sync split | Async-first SPI with `asyncio.to_thread()` adapters for sync backends |
| Runtime typing | Eager activation-time Protocol validation; pytest conformance for behavioral contracts |
| Open module internals | Trusted registry; package hash verification; optional process isolation for Regulated |
| Entry-point supply chain | Signed descriptors; registry approval before activation; rejection of classpath-style auto-configuration (§5.1) |

---

## 8. Final base-framework table

Single consolidated reference. Both axes for every framework.

| Framework / Pattern | Role | Performance tier |
|---|---|---|
| `importlib.metadata` | Core dependency | Startup-time |
| Pydantic v2 (boundary use) | Core dependency | Runtime-boundary |
| Pydantic Settings | Core dependency | Activation-time |
| `typing.Protocol` (typing) | Core dependency | Compile-time |
| pytest | Core dependency | CI/certification-time |
| pluggy (ledger subscribers) | Core dependency, limited use | Runtime-boundary |
| Stevedore-style manager | Core pattern | Startup-time / activation-time |
| Django-AppConfig-style activation | Core pattern | Activation-time |
| Celery-style backend identifier | Core pattern | Activation-time |
| FastAPI-style per-request resolution | Core pattern | Runtime-boundary |
| ZCA-style adapter pattern | Core pattern | Activation-time |
| Airflow-style explicit backend contracts | Core pattern (documentation discipline) | n/a |
| setuptools entry points | Core packaging mechanism | Startup-time |
| FastAPI | Optional implementation module | Conditional |
| Hydra (selective concepts) | Optional / pattern borrow | Activation-time |
| Dynaconf | Reference / alternative config | n/a |
| Ray actor pattern | Future capability | (Regulated isolation) |
| dependency-injector | Future capability / reference | (If a future case demands DI) |
| injector | Reference framework | n/a |
| Django | Reference framework | n/a |
| Flask | Reference framework | n/a |
| Pyramid | Reference framework | n/a |
| Zope Component Architecture | Reference pattern | n/a |
| Celery (broker/result backends) | Reference pattern | n/a |
| Airflow (provider system) | Reference pattern | n/a |
| Prefect | Reference pattern | n/a |
| Twisted plugin system | Reference pattern (historical) | n/a |
| Hypothesis | Optional pairing with pytest | CI/certification-time |
| Testcontainers | Optional pairing with pytest | CI/certification-time |
| Spring Boot classpath auto-configuration | Deliberately rejected (§5.1) | n/a |
| Decorator-time registration | Deliberately rejected (§5.2) | n/a |
| Hidden DI containers | Deliberately rejected (§5.3) | n/a |
| `pkg_resources` discovery | Rejected (§5.4) | n/a |
| `entry_points().load()` during discovery | Rejected (§5.4) | n/a |
| Django `INSTALLED_APPS` global state | Rejected (§5.4) | n/a |
| Flask `g` request-global state | Rejected (§5.4) | n/a |
| Hydra CLI override syntax | Rejected (§5.4) | n/a |
| Property flags as authority toggles | Rejected (§5.4) | n/a |

---

## 9. The frozen study principle

> **Ojas studies all relevant Python framework patterns that map to Spring Boot concepts. It adopts some directly, borrows some as design patterns, keeps some as optional modules, and explicitly rejects some activation behaviors while still documenting them. No framework is ignored. Every rejection is recorded with rationale.**

---

*End of OJAS_BASE_FRAMEWORK_STUDY.md v0.1.*
