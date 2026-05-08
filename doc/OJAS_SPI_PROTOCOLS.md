# OJAS SPI Protocols

**Status:** Draft implementation spec — **v0.1 sketch-level**
**Scope:** Async `typing.Protocol` contracts for backend modules and control-plane services.

> **Depth-of-spec note.** This document defines the **v0.1 contract shape** for Ojas SPIs. It identifies the protocol surfaces, the conceptual method signatures, and the binding contract requirements that backend authors and control-plane implementers must satisfy.
>
> **Full method-level contracts** — including per-method preconditions, postconditions, idempotency expectations, exception models, ledger obligations, and conformance test references — are planned for **v0.2**. Until v0.2 lands, treat the conformance test packages as the binding interpretation of any contract requirement that is summarized here in bullet form.
>
> Backend implementers may begin work against v0.1; v0.2 will deepen the contracts without changing the protocol shape.

---

## 1. Purpose

This document defines the high-level SPI protocol surfaces for Ojas modules.

The concrete Pydantic models referenced here are defined in `OJAS_ARTIFACTS.md`.

All SPIs are async-first.

---

## 2. SPI design rules

1. All SPI methods are async.
2. No parallel sync SPI is defined.
3. Implementations must use Ojas artifacts for inputs/outputs.
4. Normal business/tool failures return typed result artifacts.
5. Contract violations raise typed Ojas exceptions.
6. Every SPI implementation must emit or support ledger events required by conformance.
7. Every SPI implementation must declare governance coverage.
8. Every SPI implementation must pass pytest conformance tests before certification.

---

## 3. Runtime Backend SPI

Package:

```text
ojas-spi-runtime
```

Conceptual protocol:

```python
from typing import Protocol

class RuntimeBackend(Protocol):
    backend_id: str

    async def initialize(self, run_context: RunContext) -> RuntimeSession: ...
    async def execute_step(self, session: RuntimeSession, context: ContextPackage) -> StepResult: ...
    async def request_tool_call(self, session: RuntimeSession, call: ToolCallRequest) -> ToolCallToken: ...
    async def submit_tool_result(self, session: RuntimeSession, token: ToolCallToken, result: ToolResult) -> None: ...
    async def close_session(self, session: RuntimeSession) -> SessionClosure: ...
    async def report_lifecycle_state(self, session: RuntimeSession) -> LifecycleState: ...
    async def governance_coverage(self) -> GovernanceCoverage: ...
```

### Contract requirements

- All tool calls must pass through `request_tool_call`.
- The backend must not execute unbrokered tools.
- Lifecycle state must agree with Ojas run state.
- Backend state persisted outside the session must be visible to Ojas through declared accessors.
- Direct memory writes are not allowed unless routed through Memory SPI and Ojas admission.
- Late operations after run closure must be rejected.

### Failure model

| Failure | Expected behavior |
|---|---|
| invalid context package | raise contract violation |
| backend unavailable | return/raise infrastructure failure |
| tool bypass attempted | raise governance violation |
| lifecycle mismatch | raise lifecycle violation |
| unsupported feature | return unsupported capability result |

---

## 4. Tool SPI

Package:

```text
ojas-spi-tool
```

Conceptual protocol:

```python
class ToolImplementation(Protocol):
    tool_id: str
    declared_schema: ToolSchema
    declared_effects: list[Effect]

    async def invoke(self, scope: ToolScope, args: dict) -> ToolResult: ...
    async def declared_idempotent(self) -> bool: ...
    async def declared_destructive(self) -> bool: ...
    async def health_check(self) -> ToolHealth: ...
```

### Contract requirements

- Tool must not execute side effects outside `invoke`.
- Arguments must conform to declared schema before side effects.
- Tool must declare effects: read, write, destructive, external.
- Expected tool/business failures return `ToolResult`.
- Unexpected security/contract failures raise typed exceptions.
- Tool output must carry source-trust metadata.

---

## 5. Memory SPI

Package:

```text
ojas-spi-memory
```

Conceptual protocol:

```python
class MemoryBackend(Protocol):
    backend_id: str

    async def query_candidates(self, scope: MemoryScope, query: MemoryQuery) -> list[CandidateMemory]: ...
    async def write_candidate(self, scope: MemoryScope, candidate: CandidateMemory) -> WriteToken: ...
    async def commit_admission(self, token: WriteToken, decision: AdmissionDecision) -> None: ...
    async def revoke(self, scope: MemoryScope, memory_id: str, reason: RevocationReason) -> None: ...
    async def list_revocations_since(self, scope: MemoryScope, epoch: int) -> list[Revocation]: ...
    async def health_check(self) -> MemoryHealth: ...
```

### Contract requirements

- `query_candidates` returns candidates, not context permission.
- Candidates must preserve stored authority labels.
- `write_candidate` must not create authority.
- Durability/authority effect requires `commit_admission`.
- Revocations must be queryable for replay and filtering.
- Backend must not strip scope, authority, revocation, or expiry metadata.

---

## 6. Review SPI

Package:

```text
ojas-spi-review
```

Conceptual protocol:

```python
class ReviewBackend(Protocol):
    backend_id: str

    async def submit_for_review(self, artifact: GovernedArtifact, scope: ReviewScope) -> ReviewToken: ...
    async def poll_decision(self, token: ReviewToken) -> ReviewDecision | None: ...
    async def get_decision_evidence(self, token: ReviewToken) -> ReviewEvidence: ...
    async def cancel_review(self, token: ReviewToken, reason: str) -> ReviewCancellation: ...
```

### Contract requirements

- Review decisions must carry reviewer identity.
- Reviewer scope must be recorded.
- Evidence must be retrievable.
- Review status must not be inferred from ticket existence.
- Lack of response does not create approval.

---

## 7. Evidence SPI

Package:

```text
ojas-spi-evidence
```

Conceptual protocol:

```python
class EvidenceBackend(Protocol):
    backend_id: str

    async def store(self, evidence: Evidence) -> EvidenceRef: ...
    async def retrieve(self, ref: EvidenceRef) -> Evidence: ...
    async def link(self, parent_ref: EvidenceRef, child_ref: EvidenceRef) -> None: ...
    async def export_pack(self, refs: list[EvidenceRef]) -> EvidencePack: ...
```

### Contract requirements

- Evidence storage must be append-only for Standard/Regulated.
- Retrieval must return exact stored content where possible.
- Mutation-capable stores are non-conformant unless wrapped append-only.
- Evidence links must be auditable.
- Export packs must preserve source references.

---

# Control-plane SPIs

Control-plane SPIs live under:

```text
ojas-spi-control-plane
```

with sub-services:

```text
ledger
registry
review
policy
evidence
secrets
```

---

## 8. Control-plane Ledger SPI

Conceptual protocol:

```python
class ControlPlaneLedger(Protocol):
    async def append_event(self, event: LedgerEvent) -> LedgerEventRef: ...
    async def get_run_events(self, run_id: str) -> list[LedgerEvent]: ...
    async def close_run(self, run_id: str, closure: RunClosure) -> None: ...
    async def verify_integrity(self, run_id: str) -> LedgerIntegrityResult: ...
```

### Contract requirements

- Append-only semantics.
- Events must be ordered per run.
- Run closure must prevent late mutation.
- Integrity verification required for Regulated.

---

## 9. Control-plane Registry SPI

Conceptual protocol:

```python
class ControlPlaneRegistry(Protocol):
    async def verify_module_trust(self, descriptor: ModuleDescriptor) -> TrustDecision: ...
    async def resolve_certification(self, module_id: str, variant: Variant) -> CertificationDecision: ...
    async def check_revocation(self, module_id: str) -> RevocationStatus: ...
    async def record_activation(self, activation: ModuleActivationRecord) -> None: ...
```

### Contract requirements

- Registry is the trust authority for modules.
- Module descriptor claims are not self-authorizing.
- Revoked modules must not activate.
- Certification is variant/risk-specific.

---

## 10. Control-plane Policy SPI

Conceptual protocol:

```python
class ControlPlanePolicy(Protocol):
    async def evaluate_scope(self, request: ScopeDecisionRequest) -> PolicyDecision: ...
    async def evaluate_tool(self, request: ToolPolicyRequest) -> PolicyDecision: ...
    async def evaluate_memory(self, request: MemoryPolicyRequest) -> PolicyDecision: ...
    async def evaluate_context(self, request: ContextPolicyRequest) -> PolicyDecision: ...
```

### Contract requirements

- Policy decisions must identify policy version.
- Deny decisions must be explicit.
- Policy output must be ledgerable.
- Policy does not rewrite memory or scope by itself.

---

## 11. Control-plane Review SPI

Conceptual protocol:

```python
class ControlPlaneReview(Protocol):
    async def submit(self, artifact: GovernedArtifact, scope: ReviewScope) -> ReviewToken: ...
    async def decision(self, token: ReviewToken) -> ReviewDecision | None: ...
    async def evidence(self, token: ReviewToken) -> ReviewEvidence: ...
```

This is the enterprise control-plane review surface. It may wrap ServiceNow, Jira, or an enterprise workflow system.

---

## 12. Control-plane Evidence SPI

Conceptual protocol:

```python
class ControlPlaneEvidence(Protocol):
    async def store(self, evidence: Evidence) -> EvidenceRef: ...
    async def retrieve(self, ref: EvidenceRef) -> Evidence: ...
    async def export_pack(self, request: EvidenceExportRequest) -> EvidencePack: ...
```

---

## 13. Control-plane Secrets SPI

Conceptual protocol:

```python
class ControlPlaneSecrets(Protocol):
    async def resolve(self, ref: SecretRef, scope: SecretScope) -> SecretValue: ...
    async def check_access(self, ref: SecretRef, scope: SecretScope) -> SecretAccessDecision: ...
```

### Contract requirements

- Secret values must never be written to ledger.
- Secret references may be ledgered.
- Secret access is scope-bound.
- Secret resolution failure must fail activation/run where required.

---

## 14. Conformance mapping

Each SPI method must map to conformance tests.

Examples:

| Protocol | Method | Conformance expectation |
|---|---|---|
| RuntimeBackend | `request_tool_call` | all tools route through Ojas |
| RuntimeBackend | `report_lifecycle_state` | state agrees with Ojas lifecycle |
| MemoryBackend | `query_candidates` | returns candidates with authority labels |
| MemoryBackend | `commit_admission` | no authority before admission |
| ToolImplementation | `invoke` | validates args before side effects |
| ReviewBackend | `poll_decision` | no decision inferred from missing response |
| EvidenceBackend | `store` | append-only behavior |
| ControlPlaneLedger | `append_event` | ordered, immutable event append |
| ControlPlaneRegistry | `verify_module_trust` | descriptor claim not self-authorizing |
