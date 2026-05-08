# OJAS Artifacts

**Status:** Draft implementation spec — **v0.1 inventory-level**
**Scope:** Shared artifact/model inventory used by Ojas SPIs.

> **Depth-of-spec note.** This document defines the **v0.1 artifact inventory** — the families of Pydantic models that must exist and the surfaces they cross. It does **not** yet provide full model field definitions, validation rules, JSON Schema bindings, or content-hash conventions.
>
> **Full Pydantic model definitions** — fields, types, constraints, frozen-vs-mutable discipline, discriminated unions, hash conventions, and schema versioning — are planned for **v0.2**. Until v0.2 lands, this inventory is the binding list of artifact families; field-level shape may evolve.
>
> SPI implementers should treat the artifact names listed here as stable; field-level contracts will be tightened in v0.2 without removing artifacts from the inventory.

---

## 1. Purpose

Ojas SPIs exchange structured artifacts.

All artifacts should be represented as Pydantic v2 models with JSON Schema export.

---

## 2. Core identity artifacts

| Artifact | Purpose |
|---|---|
| `AgentId` | stable agent identity |
| `AgentVersion` | version identifier |
| `TenantId` | tenant identity |
| `UserId` | user/caller identity |
| `RunId` | Ojas run identity |
| `TraceId` | trace identity |
| `LedgerEventId` | ledger event identity |
| `ModuleId` | module identity |
| `BackendId` | backend identity |
| `ToolId` | tool identity |
| `MemoryId` | memory identity |
| `EvidenceRef` | evidence reference |

---

## 3. Runtime artifacts

| Artifact | Purpose |
|---|---|
| `RunContext` | immutable run context |
| `RuntimeSession` | backend session handle |
| `ContextPackage` | Ojas-approved context passed to runtime |
| `StepResult` | runtime step output |
| `SessionClosure` | run/session closure artifact |
| `LifecycleState` | runtime lifecycle state |
| `GovernanceCoverage` | backend coverage declaration |

---

## 4. Scope artifacts

| Artifact | Purpose |
|---|---|
| `ScopeSnapshot` | immutable run scope |
| `ToolScope` | tool invocation scope |
| `MemoryScope` | memory operation scope |
| `ReviewScope` | reviewer decision scope |
| `SecretScope` | secret access scope |
| `ScopeDecisionRequest` | policy request for scope |
| `ScopeExpansionRequest` | agent request for broader scope |

---

## 5. Tool artifacts

| Artifact | Purpose |
|---|---|
| `ToolSchema` | argument schema |
| `ToolCallRequest` | requested tool call |
| `ToolCallToken` | brokered tool-call token |
| `ToolResult` | result from tool |
| `ToolHealth` | tool health state |
| `Effect` | read/write/destructive/external effect classification |
| `ToolPolicyRequest` | policy check for tool |

---

## 6. Memory artifacts

| Artifact | Purpose |
|---|---|
| `MemoryQuery` | memory candidate query |
| `CandidateMemory` | candidate memory returned from backend |
| `WriteToken` | pending memory-write token |
| `AdmissionDecision` | admit/quarantine/reject/promote decision |
| `RevocationReason` | reason for revocation |
| `Revocation` | revocation record |
| `MemoryHealth` | backend health state |
| `MemoryPolicyRequest` | policy check for memory |

---

## 7. Context artifacts

| Artifact | Purpose |
|---|---|
| `ContextCandidate` | material considered for context |
| `ContextDecision` | include/exclude/redact decision |
| `ContextPackage` | final model-visible context package |
| `ContextPolicyRequest` | policy check for context inclusion |

---

## 8. Review artifacts

| Artifact | Purpose |
|---|---|
| `GovernedArtifact` | proposal/reviewable item |
| `ReviewToken` | review tracking token |
| `ReviewDecision` | reviewer/external decision |
| `ReviewEvidence` | evidence supporting decision |
| `ReviewCancellation` | cancellation artifact |

---

## 9. Evidence artifacts

| Artifact | Purpose |
|---|---|
| `Evidence` | evidence record |
| `EvidenceRef` | reference to evidence |
| `EvidencePack` | grouped exportable evidence |
| `EvidenceExportRequest` | request to export pack |

---

## 10. Ledger artifacts

| Artifact | Purpose |
|---|---|
| `LedgerEvent` | append-only event |
| `LedgerEventRef` | event reference |
| `RunClosure` | run closure record |
| `LedgerIntegrityResult` | integrity verification result |
| `ModuleActivationRecord` | record that module became active |

---

## 11. Registry/certification artifacts

| Artifact | Purpose |
|---|---|
| `ModuleDescriptor` | parsed module descriptor |
| `TrustDecision` | registry trust decision |
| `CertificationDecision` | variant/risk eligibility decision |
| `RevocationStatus` | module revocation state |
| `PackageAuditRecord` | package-level audit result |

---

## 12. Policy artifacts

| Artifact | Purpose |
|---|---|
| `PolicyDecision` | allow/deny/escalate decision |
| `ToolPolicyRequest` | tool policy request |
| `MemoryPolicyRequest` | memory policy request |
| `ContextPolicyRequest` | context policy request |
| `ScopeDecisionRequest` | scope policy request |

---

## 13. Secrets artifacts

| Artifact | Purpose |
|---|---|
| `SecretRef` | reference to secret |
| `SecretScope` | scope for secret access |
| `SecretValue` | resolved secret value, never ledgered |
| `SecretAccessDecision` | allow/deny secret access |

---

## 14. Model rules

All artifacts should follow these rules:

1. Pydantic v2 model.
2. JSON Schema exportable.
3. Explicit version field for long-lived artifacts.
4. Immutable where applicable.
5. Tenant/scope fields where governance-relevant.
6. No raw secrets in ledgerable artifacts.
7. Clear authority labels for authority-affecting artifacts.
8. Clear source trust fields for memory/context/tool-output artifacts.
