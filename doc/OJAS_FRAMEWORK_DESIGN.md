# OJAS Framework Design

**Status:** Draft implementation spec  
**Scope:** Python-native modular framework design, package structure, activation philosophy, and implementation boundaries.

---

## 1. Purpose

This document defines the Ojas framework-design baseline.

It is an implementation-mechanics document. It does not define memory authority, context governance, review, revocation, or lifecycle semantics. Those are core governance topics.

---

## 2. Canonical framework statement

> **Ojas is a manifest-driven, Python-native governed module framework.**
>
> It adopts Spring Boot's packaging discipline: modules, starters, SPIs, and dependency bundles.
>
> It rejects Spring Boot's activation discipline: classpath presence and silent auto-configuration.
>
> Ojas activation is explicit and governed through manifest selection, metadata-only discovery, registry trust, descriptor validation, variant eligibility, certification, and ledgered activation.

---

## 3. Spring Boot analogy — accepted and rejected parts

### 3.1 Accepted from Spring Boot

Ojas borrows:

- module packaging discipline
- starter dependency bundles
- SPI contract separation
- convention-based package layout
- developer-friendly composition
- repeatable activation lifecycle

### 3.2 Rejected from Spring Boot

Ojas rejects:

- classpath presence as activation intent
- silent auto-configuration
- hidden fallback behavior
- starter installation as trust
- dependency presence as certification
- property flags as authority toggles

### 3.3 Python-native replacement

Ojas uses:

- `importlib.metadata` for metadata-only discovery
- manifest-driven explicit selection
- registry trust verification
- descriptor validation
- variant eligibility checks
- async `typing.Protocol` contracts
- Pydantic v2 validation
- pytest conformance tests
- ledgered activation events

---

## 4. Framework vs runtime model

Ojas is an **embeddable Python governance framework** with optional external control-plane services.

| Variant | Deployment posture |
|---|---|
| Open | in-process framework can be sufficient |
| Standard | in-process framework with shared services recommended |
| Regulated | external control-plane services and process-isolated backends required |

This creates a hybrid architecture:

```text
Developer application
  ↓
Ojas in-process framework
  ↓
Ojas control-plane services where required
  ↓
Backend frameworks / memory systems / tools
```

---

## 5. Package baseline

### 5.1 Core package

```text
ojas-core
```

Owns:

- manifest validation
- module registry
- activation lifecycle
- variant checks
- run context
- scope/context service access
- ledger service access
- memory/tool/review/evidence service access

### 5.2 Backend SPI packages

```text
ojas-spi-runtime
ojas-spi-memory
ojas-spi-tool
ojas-spi-review
ojas-spi-evidence
```

### 5.3 Unified control-plane SPI package

```text
ojas-spi-control-plane
```

Sub-services:

```text
.ledger
.registry
.review
.policy
.evidence
.secrets
```

### 5.4 Backend implementation packages

```text
ojas-backend-langgraph
ojas-backend-openai-agents
ojas-backend-crewai-restricted
ojas-memory-zep
ojas-memory-mem0
ojas-tool-mcp
ojas-tool-openapi
```

### 5.5 Starter packages

```text
ojas-starter-*
```

Starters are dependency bundles only.

> **Starters compose dependencies; they do not compose authority.**

### 5.6 Test packages

```text
ojas-test
ojas-test-runtime-conformance
ojas-test-memory-conformance
ojas-test-tool-conformance
ojas-test-review-conformance
ojas-test-evidence-conformance
```

---

## 6. Entry-point namespace baseline

```text
ojas.backend.runtime
ojas.backend.memory
ojas.backend.tool
ojas.backend.review
ojas.backend.evidence

ojas.control_plane.ledger
ojas.control_plane.registry
ojas.control_plane.review
ojas.control_plane.policy
ojas.control_plane.evidence
ojas.control_plane.secrets

ojas.test.conformance.runtime
ojas.test.conformance.memory
ojas.test.conformance.tool
ojas.test.conformance.review
ojas.test.conformance.evidence
```

---

## 7. Python version floor

Ojas framework implementation baseline:

```text
Python 3.11+
```

Rationale:

- stable modern `importlib.metadata`
- strong typing support
- exception groups
- `tomllib`
- modern async ecosystem
- better tooling compatibility for a 2026 framework baseline

---

## 8. Activation ladder

```text
discoverable → requested → trusted → variant-eligible → active
```

| State | Meaning |
|---|---|
| Discoverable | metadata found; implementation not loaded |
| Requested | manifest selected the module/backend |
| Trusted | registry/signature/source approved |
| Variant-eligible | certified or permitted for selected variant/risk |
| Active | implementation participated in deployment/run |

Rules:

- discovery is not activation
- installation is not trust
- manifest selection is not certification
- certification is not runtime participation
- active does not create authority by itself

---

## 9. Metadata-only discovery

Ojas must discover modules without executing module implementation code.

Rule:

> **Ojas MUST NOT call `entry_point.load()` during discovery.**

Discovery may inspect package metadata and descriptor files only.

Implementation loading occurs only after:

1. manifest selection
2. descriptor validation
3. registry trust verification
4. variant eligibility check
5. certification check where required

---

## 10. Async-first contract model

All Ojas SPIs are async-first.

Rules:

- no parallel sync SPI surface
- sync-only backend frameworks adapt internally
- sync wrappers must not change governance semantics
- long-running sync calls should use safe offloading where appropriate
- cancellation must be supported where possible

---

## 11. Kernel/service access pattern

Ojas does not use a hidden Spring-style dependency-injection container.

Instead:

- the kernel exposes explicit service surfaces
- backend modules receive controlled kernel references at activation/run time
- per-run dependencies are resolved through run context
- service access is auditable and scoped

This is closer to Python service-locator/explicit lifecycle patterns than Java `@Autowired`.

---

## 12. Pluggy usage boundary

Ojas may use pluggy only for cross-cutting event subscription such as ledger consumers.

Do not use pluggy for primary backend SPIs.

Backend SPIs require richer async protocol surfaces.

---

## 13. Regulated process isolation

For Regulated deployments:

> Backend modules must run in process isolation from the Ojas kernel.

Open and Standard may allow in-process execution subject to policy.

The transport mechanism is deferred to a separate micro-spec.

---

## 14. Implementation principles

1. Manifest is activation source of truth.
2. Descriptor is module identity and claim record.
3. Registry is trust authority for modules.
4. Certification is variant/risk eligibility.
5. Starters are dependency bundles only.
6. SPIs are async `typing.Protocol` contracts.
7. Artifacts are Pydantic models.
8. Conformance is pytest-based.
9. Discovery is metadata-only.
10. Activation is ledgered.
