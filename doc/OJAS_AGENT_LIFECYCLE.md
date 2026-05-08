# OJAS Agent Lifecycle

**Status:** Core governance draft  
**Scope:** Agent lifecycle states, version transitions, scope/memory status, suspension, monitoring, retirement.

---

## 1. Purpose

This document defines the lifecycle of an Ojas agent.

It focuses on governance semantics, not implementation mechanics.

An Ojas agent is not merely code that runs. It is a governed lifecycle entity whose scope, memory, tools, context, authority, and audit state change through controlled phases.

---

## 2. Developer-facing lifecycle

Simple view:

```text
Design → Deploy → Run → Review/Learn → Monitor → Retire
```

| Stage | Meaning |
|---|---|
| Design | Agent intent and constraints are drafted |
| Deploy | Agent is registered, validated, scoped, and activated |
| Run | Agent operates under a run-bound scope and governed context |
| Review/Learn | Outputs and candidate memories may be reviewed and promoted |
| Monitor | Ojas watches drift, misuse, stale authority, and revocation signals |
| Retire | Agent is decommissioned while memory/evidence follow retention policy |

---

## 3. Reference lifecycle states

```text
DESIGNED
  ↓
REGISTERED
  ↓
VALIDATED
  ↓
DEPLOYED
  ↓
ACTIVE
  ↓
RUNNING
  ↓
CLOSED
  ↓
MONITORED
  ↓
RETIRED
```

Exceptional transitions:

```text
ACTIVE → SUSPENDED
RUNNING → SUSPENDED
MONITORED → SUSPENDED
SUSPENDED → ACTIVE
SUSPENDED → RETIRED
```

Version transition:

```text
ACTIVE(v1) → VERSION_TRANSITION → ACTIVE(v2)
```

---

## 4. Phase 1 — Design

The agent exists as a design artifact.

### Scope status

```text
UNBOUND
```

### Memory status

```text
NO_RUNTIME_MEMORY
```

### Rule

> A design artifact has no runtime authority, no tenant access, no tool access, and no memory access.

---

## 5. Phase 2 — Registration

The agent is registered with Ojas.

Registration declares:

- agent identity
- intended variant
- purpose
- scope template
- tool policy
- memory policy
- handoff policy
- review requirements
- risk class

### Scope status

```text
DECLARED_SCOPE
```

### Memory status

```text
DECLARED_MEMORY_POLICY
```

### Rule

> Registration records intent. It does not grant runtime authority.

---

## 6. Phase 3 — Validation

Ojas validates the registered agent definition.

Validation includes:

- identity uniqueness
- variant consistency
- scope template validity
- tool policy validity
- memory policy validity
- handoff policy validity
- review path presence
- audit/evidence requirements
- conflict with existing authority

### Scope status

```text
VALIDATED_SCOPE_TEMPLATE
```

### Memory status

```text
VALIDATED_MEMORY_POLICY
```

### Rule

> A valid agent definition is a deployable contract, not a runtime grant.

---

## 7. Phase 4 — Deployment / Activation

The agent is deployed into an environment.

Examples:

```text
dev
qa
staging
production
```

### Scope status

```text
ENVIRONMENT_BOUND_SCOPE
```

### Memory status

```text
MEMORY_BACKEND_BOUND
```

### Rule

> Deployment activates the governance envelope, not unlimited agent access.

---

## 8. Phase 5 — Version Transition

Agent version changes are governed lifecycle events.

### Required rules

1. In-flight runs complete under the version they started with.
2. New runs use the newly activated version.
3. Memory inheritance across versions requires explicit migration policy.
4. Semantic memory authority is not automatically inherited across incompatible versions.
5. Tool, scope, context, and review policies must be revalidated for the new version.
6. Rollback must restore the old version's scope, memory compatibility, and authority assumptions.

### Scope status

```text
VERSION_BOUND_SCOPE
```

### Memory status

```text
MIGRATION_PENDING | MIGRATION_APPROVED | MIGRATION_REJECTED
```

### Rule

> A new agent version is not automatically authorized to inherit the scope, memory authority, or operating assumptions of the old version.

---

## 9. Phase 6 — Run Initialization

A caller invokes the agent.

Ojas creates a run-bound governance context:

- run ID
- trace/audit identity
- tenant identity
- caller identity
- agent version
- variant
- scope snapshot
- applicable policies
- memory access constraints
- tool constraints
- review requirements

### Scope status

```text
RUN_BOUND_SCOPE
```

### Memory status

```text
RUN_MEMORY_CONTEXT_INITIALIZED
```

### Rule

> Every run receives a fresh scope snapshot. The agent cannot widen it during execution.

---

## 10. Phase 7 — Context Assembly

Before execution, Ojas assembles the context the agent is allowed to see.

### Top-level invariant

> **Memory retrieval does not equal context permission. Ojas must approve what enters the agent context.**

### Scope status

```text
CONTEXT_FILTERED_SCOPE
```

### Memory status

```text
READ_FILTERED_MEMORY
```

### Rule

> Context assembly is the moment where retrieved material becomes allowed or blocked model input.

---

## 11. Phase 8 — Execution

The agent performs work under Ojas governance.

### Scope status

```text
ACTIVE_EXECUTION_SCOPE
```

### Memory status

```text
ACTIVE_WORKING_MEMORY
```

### Ojas-governed operations

- tool calls
- memory reads
- memory writes
- handoffs
- context updates
- output generation
- evidence capture

### Rule

> The framework or runtime may execute logic, but Ojas governs every boundary-crossing operation.

---

## 12. Phase 9 — Tool Invocation

The agent requests a tool.

Ojas checks:

- tool allowed
- action allowed
- tenant allowed
- caller allowed
- arguments valid
- risk class
- approval requirement
- result handling policy

### Scope status

```text
TOOL_SCOPE_CHECK
```

### Memory status

```text
TOOL_OUTPUT_CLASSIFIED
```

### Rule

> Tool output is evidence or candidate context first. It becomes memory only through admission.

---

## 13. Phase 10 — Memory Write Attempt

The agent or runtime attempts to write memory.

Possible outcomes:

```text
ADMITTED_WORKING_MEMORY
ADMITTED_SESSION_MEMORY
ADMITTED_EPISODIC_MEMORY
CANDIDATE_LESSON
PENDING_REVIEW
QUARANTINED
REJECTED
```

### Scope status

```text
MEMORY_WRITE_SCOPE_CHECK
```

### Memory status

```text
ADMISSION_PENDING | ADMITTED | QUARANTINED | REJECTED
```

### Rule

> The agent may propose memory. Ojas admits, downgrades, quarantines, or rejects it.

Decision logic belongs to the memory authority model.

---

## 14. Phase 11 — Output / Proposal Generation

The agent returns an output.

Ojas classifies and wraps it.

Possible output classifications:

- answer
- recommendation
- proposal
- review request
- evidence report
- scope expansion request
- memory promotion request
- tool action proposal

### Scope status

```text
OUTPUT_SCOPE_VALIDATED
```

### Memory status

```text
NO_AUTHORITY_CREATED_BY_OUTPUT
```

### Rule

> Agent output is a proposal unless a separate approved authority path says otherwise.

---

## 15. Phase 12 — Review / Approval

High-risk proposals and authority-affecting outputs are reviewed.

### Scope status

```text
REVIEW_SCOPE
```

### Memory status

```text
PENDING_REVIEW
```

### Rule

> Human or external authority may approve promotion or action. The agent cannot approve its own authority.

---

## 16. Phase 13 — Semantic Promotion

Approved knowledge may be promoted into authority-bearing memory.

Promotion requires:

- source evidence
- scope
- reviewer/external authority
- effective date
- expiry/review date
- revocation path
- audit record

### Scope status

```text
AUTHORITY_SCOPE_BOUND
```

### Memory status

```text
SEMANTIC_MEMORY_APPROVED
```

### Rule

> Semantic promotion is an authority-creating event and must be fully auditable.

---

## 17. Phase 14 — Run Closure

The run ends.

### Scope status

```text
RUN_SCOPE_CLOSED
```

### Memory closure behavior

| Memory type | Closure behavior |
|---|---|
| Working | discarded |
| Session | retained until TTL |
| Episodic | retained if admitted |
| Candidate Lesson | pending/reviewed |
| Semantic | unaffected unless promotion occurred |
| Procedural | updated only by governed stats pipeline |

### Rule

> Run closure prevents late mutation.

---

## 18. Phase 15 — Monitoring / Drift / Revocation

Ojas monitors deployed agents.

Signals:

- scope violations
- tool misuse
- memory conflict
- stale authority
- backend drift
- policy mismatch
- unexpected outputs
- review backlog
- revocation triggers

### Scope status

```text
MONITORED_SCOPE
```

### Memory status

```text
ACTIVE | STALE | REVOKED | EXPIRED | QUARANTINED | DOWNGRADED
```

### Rule

> Memory authority must remain revocable and explainable.

---

## 19. Phase 16 — Suspension / Emergency Freeze

Ojas may freeze an agent.

Triggers:

- cross-tenant access attempt
- tool bypass
- memory poisoning
- stale policy detected
- review queue failure
- certification invalidated
- lifecycle inconsistency
- severe privacy/security incident

### Scope status

```text
SUSPENDED
```

### Memory status

```text
READ_ONLY_OR_QUARANTINED
```

### Rule

> A frozen agent cannot continue learning or acting until restored by authorized governance.

---

## 20. Phase 17 — Retirement

The agent is decommissioned.

### Scope status

```text
RETIRED
```

### Memory handling

Memory may be:

- retained for audit
- expired
- anonymized
- revoked
- archived
- deleted where legally required

### Rule

> Retiring an agent does not automatically delete audit evidence or memory authority. Retention policy governs.

---

## 21. Runtime lifecycle authority

External runtime state is subordinate to Ojas lifecycle state.

If an underlying framework believes a run is active but Ojas has closed, suspended, or retired it, Ojas treats that as a lifecycle inconsistency.

### Rule

> Ojas lifecycle state is authoritative for governed operations.

---

## 22. Required lifecycle audit events

Each phase must produce audit events.

| Phase | Entry event | Exit event | Exceptional event |
|---|---|---|---|
| Registration | `AGENT_REGISTER_REQUESTED` | `AGENT_REGISTERED` | `REGISTRATION_REJECTED` |
| Validation | `AGENT_VALIDATION_STARTED` | `AGENT_VALIDATED` | `AGENT_INVALID` |
| Deployment | `AGENT_DEPLOYMENT_STARTED` | `AGENT_DEPLOYED` | `DEPLOYMENT_BLOCKED` |
| Version transition | `VERSION_TRANSITION_STARTED` | `VERSION_ACTIVATED` | `VERSION_MIGRATION_REJECTED` |
| Run initialization | `RUN_INITIALIZATION_STARTED` | `RUN_INITIALIZED` | `RUN_SCOPE_DENIED` |
| Context assembly | `CONTEXT_ASSEMBLY_STARTED` | `CONTEXT_ASSEMBLED` | `CONTEXT_ITEM_REJECTED` |
| Execution | `EXECUTION_STARTED` | `EXECUTION_COMPLETED` | `LIFECYCLE_CONFLICT` |
| Tool invocation | `TOOL_REQUESTED` | `TOOL_COMPLETED` | `TOOL_DENIED` |
| Memory write | `MEMORY_WRITE_REQUESTED` | `MEMORY_WRITE_DECIDED` | `MEMORY_QUARANTINED` |
| Output | `OUTPUT_GENERATED` | `PROPOSAL_WRAPPED` | `OUTPUT_SCOPE_VIOLATION` |
| Review | `REVIEW_REQUESTED` | `REVIEW_DECIDED` | `REVIEW_SLA_BREACH` |
| Promotion | `PROMOTION_REQUESTED` | `PROMOTION_APPROVED` | `PROMOTION_REJECTED` |
| Closure | `RUN_CLOSURE_STARTED` | `RUN_CLOSED` | `LATE_MUTATION_ATTEMPT` |
| Monitoring | `MONITORING_SIGNAL_RECEIVED` | `MONITORING_DECISION_RECORDED` | `DRIFT_DETECTED` |
| Suspension | `SUSPENSION_TRIGGERED` | `AGENT_SUSPENDED` | `FREEZE_FAILURE` |
| Retirement | `RETIREMENT_STARTED` | `AGENT_RETIRED` | `RETENTION_POLICY_FAILURE` |

---

## 23. Lifecycle invariants

1. Scope is never widened by the agent.
2. Memory retrieval does not equal context permission.
3. Agent output is a proposal unless externally approved.
4. Ojas lifecycle is authoritative for governed operations.
5. In-flight runs complete on the agent version they started with.
6. Memory inheritance across versions requires explicit migration policy.
7. Semantic promotion is an authority event.
8. Run closure prevents late mutation.
9. Suspension blocks acting and learning.
10. Retirement follows retention and revocation policy.
