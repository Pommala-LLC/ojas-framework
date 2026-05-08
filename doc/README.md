# Ojas Framework Implementation Mechanics Pack

**Status:** Draft implementation-spec pack  
**Scope:** Python-native modular framework mechanics for Ojas.

This pack intentionally excludes core agent-governance semantics such as memory authority, scope invariants, agent lifecycle, context governance, review, promotion, revocation, and suspension. Those belong in the **Ojas Core Agent Governance** pack.

## Purpose

This pack defines **how Ojas is implemented as a Python-native governed module framework**.

It covers:

- framework design baseline
- Python-native module discovery and activation
- SPI protocol surfaces
- shared artifact/schema model
- module descriptor schema
- module configuration
- conformance testing
- module security and supply-chain trust

## Canonical framework statement

> **Ojas is a manifest-driven, Python-native governed module framework.**
>
> It adopts Spring Boot's packaging discipline: modules, starters, SPIs, and dependency bundles.
>
> It rejects Spring Boot's activation discipline: classpath presence and silent auto-configuration.
>
> Ojas activation is explicit and governed through manifest selection, metadata-only discovery, registry trust, descriptor validation, variant eligibility, certification, and ledgered activation.

## Documents

| Document | Purpose |
|---|---|
| `OJAS_FRAMEWORK_DESIGN.md` | Freezes the Python-native framework architecture |
| `OJAS_SPI_PROTOCOLS.md` | Defines async `typing.Protocol` SPI surfaces |
| `OJAS_ARTIFACTS.md` | Defines shared Pydantic artifact/model inventory |
| `OJAS_MODULE_DESCRIPTOR_SCHEMA.md` | Defines `ojas-module-descriptor.yaml` structure |
| `OJAS_MODULE_CONFIGURATION.md` | Defines module configuration and secret handling |
| `OJAS_TEST_CONFORMANCE.md` | Defines pytest-based conformance testing |
| `OJAS_MODULE_SECURITY.md` | Defines metadata-only discovery, registry trust, signing, isolation, and monkey-patching rules |

## See also

Core agent governance semantics — authority invariants, agent lifecycle, scope model, memory authority, context assembly, review/promotion/revocation — are defined in the companion pack:

> **`OJAS_Core_Agent_Governance_Docs`**

Specifically:

| Concern | Companion document |
|---|---|
| Master authority invariants and labels | `OJAS_FOUNDATION_AUTHORITY_INVARIANTS.md` |
| Agent lifecycle states and audit events | `OJAS_AGENT_LIFECYCLE.md` |
| Scope types, lifecycle, and invariants | `OJAS_SCOPE_MODEL.md` |
| Core memory types and admission rules | `OJAS_MEMORY_AUTHORITY.md` |
| Runtime context assembly and module-trust filtering | `OJAS_CONTEXT_ASSEMBLY.md` |
| Review, promotion, revocation, freeze, recovery | `OJAS_REVIEW_PROMOTION_REVOCATION.md` |

This pack assumes those governance semantics are defined externally. Where this pack uses concepts like *authority label*, *governed artifact*, *admission decision*, or *scope snapshot*, the operational definitions live in the core pack.
