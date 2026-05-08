# OJAS Review, Promotion, Revocation, and Suspension

**Status:** Core governance draft  
**Scope:** Review requests, approval decisions, semantic promotion, revocation, emergency freeze, and recovery.

---

## 1. Purpose

This document defines the Ojas governance path for turning candidate outputs or memories into approved authority, and for removing or freezing authority when risk is detected.

It focuses on core governance semantics, not implementation mechanics.

---

## 2. Core principle

> **The agent may propose, but it cannot approve its own authority.**

Review, promotion, and revocation are external governance paths.

---

## 3. Review triggers

Ojas should route an artifact to review when it is:

- authority-affecting
- high-risk
- destructive
- cross-tenant-sensitive
- privacy-sensitive
- policy-changing
- semantic-memory candidate
- scope-expansion request
- tool-action proposal requiring approval
- conflict with existing authority
- generated from untrusted source
- Open-origin artifact seeking stricter use

---

## 4. Reviewable artifacts

| Artifact | Review purpose |
|---|---|
| Proposal | decide whether action/output may proceed |
| Candidate Lesson | decide whether learning is valid |
| Semantic Promotion Request | decide whether memory becomes authority |
| Scope Expansion Request | decide whether agent receives more scope |
| Tool Approval Request | decide whether risky/destructive tool may run |
| Revocation Request | decide whether authority must be withdrawn |
| Exception Request | decide whether policy exception is allowed |
| Evidence Pack | support review decision |

---

## 5. Review decision types

```text
APPROVE
REJECT
REQUEST_MORE_EVIDENCE
DOWNGRADE
QUARANTINE
ESCALATE
APPROVE_WITH_SCOPE_LIMIT
APPROVE_WITH_EXPIRY
REVOKE
```

---

## 6. Review decision requirements

A review decision must include:

- reviewer identity or external authority identity
- reviewer scope
- decision type
- decision time
- evidence references
- reason
- effective scope
- expiry/review date where applicable
- downstream action
- audit reference

Rule:

> A review decision without scope and evidence cannot create durable authority.

---

## 7. Semantic promotion

Semantic promotion turns candidate knowledge into authority-bearing Semantic Memory.

Required inputs:

- candidate memory
- source evidence
- source trust
- conflict analysis
- reviewer/external authority
- effective scope
- variant eligibility
- expiry or review date
- revocation path

Promotion output:

```text
SEMANTIC_APPROVED
```

Rule:

> Semantic promotion is an authority-creating event and must be more auditable than ordinary storage.

---

## 8. Promotion boundaries

Prohibited:

```text
agent promotes its own memory
tool output becomes semantic memory directly
reviewer feedback becomes rule directly
Open self-promoted memory becomes Regulated authority directly
simulation result becomes production authority directly
graph inference becomes semantic fact directly
```

Allowed:

```text
candidate lesson routed to review
reviewer-approved promotion
external authoritative source imported through admission
existing semantic memory revised through controlled update
semantic memory revoked or expired
```

---

## 9. Revocation

Revocation removes or disables authority.

Revocation may apply to:

- semantic memory
- identity memory
- procedural memory
- package/backend memory
- review decision
- proposal approval
- tool authorization
- scope grant

Revocation reasons:

```text
source invalidated
policy changed
reviewer error
memory poisoning detected
cross-tenant risk
privacy deletion request
expiry reached
backend certification revoked
legal/compliance hold
```

---

## 10. Revocation effects

| Target | Effect |
|---|---|
| Semantic Memory | excluded from future authority/context |
| Candidate Lesson | rejected or quarantined |
| Episodic Memory | retained as evidence but not authority |
| Tool Authorization | future calls denied |
| Scope Grant | removed from future scope snapshots |
| Package/Backend Memory | backend no longer eligible |
| Review Decision | downstream authority may be invalidated |
| Evidence | retained, annotated, or legally restricted |

Rule:

> Revocation must not erase audit history unless a valid legal deletion path applies.

---

## 11. Emergency freeze

Emergency freeze suspends agent action or learning.

Triggers:

- cross-tenant access attempt
- memory poisoning
- unauthorized tool call
- repeated policy violations
- severe privacy incident
- stale critical authority
- reviewer compromise
- backend/capability revoked
- lifecycle inconsistency
- audit/ledger failure

Freeze state:

```text
SUSPENDED
```

Memory state:

```text
READ_ONLY_OR_QUARANTINED
```

Rule:

> A frozen agent cannot continue acting, learning, promoting, or expanding scope.

---

## 12. Freeze levels

| Level | Meaning |
|---|---|
| `RUN_FREEZE` | current run blocked |
| `AGENT_FREEZE` | agent blocked from new runs |
| `MEMORY_FREEZE` | memory reads/writes restricted |
| `TOOL_FREEZE` | tool class disabled |
| `TENANT_FREEZE` | tenant-level isolation action |
| `BACKEND_FREEZE` | backend/capability disabled |
| `GLOBAL_EMERGENCY_FREEZE` | platform-wide severe event |

---

## 13. Recovery from freeze

Recovery requires:

- cause analysis
- affected scope identification
- memory review
- revocation/downgrade decisions
- tool/backend eligibility review
- reviewer/external authority approval
- audit record
- restored scope snapshot

Rule:

> Unfreezing is itself a governed decision.

---

## 14. Downgrade

Downgrade reduces authority without full revocation.

Examples:

```text
SEMANTIC_APPROVED → STALE
SEMANTIC_APPROVED → EPISODIC_EVIDENCE_ONLY
CANDIDATE_LESSON → QUARANTINED
STANDARD_AUTHORITY → OPEN_CONTEXT_ONLY
```

Rule:

> Downgrade must be visible to context assembly and future runs.

---

## 15. Review backlog and stale authority

If review queues back up, Ojas must not silently promote authority.

Possible actions:

- keep as pending
- downgrade candidate
- expire pending item
- escalate
- block dependent run
- mark system stale
- alert operator

Rule:

> Lack of review does not create approval.

---

## 16. Audit events

The full audit-event vocabulary for review, promotion, revocation, freeze, and recovery flows.

### 16.1 Review events

| Event | Meaning |
|---|---|
| `REVIEW_REQUESTED` | artifact routed to review |
| `REVIEW_ASSIGNED` | reviewer/external authority assigned |
| `REVIEW_DECIDED` | decision recorded |
| `REVIEW_REJECTED` | reviewer rejected the artifact |
| `REVIEW_DOWNGRADED` | reviewer reduced authority claim |
| `REVIEW_QUARANTINED` | reviewer quarantined the artifact |
| `REVIEW_ESCALATED` | reviewer escalated to higher authority |
| `REVIEW_CANCELLED` | review cancelled before decision |
| `REVIEW_SLA_BREACH` | review delay exceeded SLA |

### 16.2 Promotion events

| Event | Meaning |
|---|---|
| `PROMOTION_REQUESTED` | semantic promotion requested |
| `PROMOTION_APPROVED` | authority created |
| `PROMOTION_REJECTED` | authority denied |
| `PROMOTION_DOWNGRADED` | promotion accepted at a lower authority than requested |
| `PROMOTION_EXPIRED` | promotion request expired without decision |

### 16.3 Revocation events

| Event | Meaning |
|---|---|
| `REVOCATION_REQUESTED` | revocation requested |
| `REVOCATION_APPROVED` | revocation decision recorded |
| `REVOCATION_REJECTED` | revocation request denied |
| `MEMORY_REVOKED` | memory authority revoked |
| `TOOL_AUTHORIZATION_REVOKED` | tool authorization revoked |
| `SCOPE_GRANT_REVOKED` | scope grant revoked |
| `MODULE_AUTHORITY_REVOKED` | module/backend authority revoked |
| `REVIEW_DECISION_REVOKED` | downstream effects of a review decision revoked |
| `AUTHORITY_DOWNGRADED` | authority reduced without full revocation |
| `AUTHORITY_EXPIRED` | authority lapsed by time |
| `AUTHORITY_STALE_MARKED` | authority marked stale pending revalidation |

### 16.4 Freeze and recovery events

| Event | Meaning |
|---|---|
| `FREEZE_TRIGGERED` | freeze condition detected |
| `FREEZE_APPLIED` | freeze enforcement active |
| `RUN_FREEZE_APPLIED` | current run blocked |
| `AGENT_SUSPENDED` | agent blocked from new runs |
| `MEMORY_FREEZE_APPLIED` | memory reads/writes restricted |
| `TOOL_FREEZE_APPLIED` | tool class disabled |
| `TENANT_FREEZE_APPLIED` | tenant-level isolation action |
| `BACKEND_FREEZE_APPLIED` | backend/capability disabled |
| `GLOBAL_EMERGENCY_FREEZE_APPLIED` | platform-wide severe event |
| `FREEZE_FAILURE` | freeze enforcement failed |
| `UNFREEZE_REQUESTED` | recovery requested |
| `RECOVERY_REVIEW_REQUIRED` | recovery requires review or external authority |
| `UNFREEZE_APPROVED` | recovery decision recorded |
| `UNFREEZE_REJECTED` | recovery denied |
| `AGENT_RESTORED` | agent returned to active |
| `RECOVERY_SCOPE_RESTORED` | restored scope snapshot recorded |

---

## 17. Invariants

1. The agent cannot approve its own authority.
2. Review decisions require scope and evidence.
3. Semantic promotion is an auditable authority event.
4. Revocation removes future authority but preserves audit evidence.
5. Freeze blocks acting and learning.
6. Unfreezing is governed.
7. Lack of review does not create approval.
8. Downgraded authority must be visible to context assembly.
9. Every freeze, revocation, downgrade, and recovery event must be ledgered.
10. Recovery from freeze requires reviewer or external authority approval.
