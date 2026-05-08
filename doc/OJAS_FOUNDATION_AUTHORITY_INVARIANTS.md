# OJAS Foundation — Authority Invariants

**Status:** Core governance draft  
**Applies to:** All Ojas agents, memories, scopes, tools, contexts, proposals, reviews, and runtime decisions.

---

## 1. Purpose

This document defines the master authority principles of Ojas.

It is not a framework implementation document. It does not define Python package structure, SPI signatures, module discovery, descriptor schemas, or conformance-test mechanics.

It defines what Ojas means by **authority**.

---

## 2. Master Authority Invariant

> **Capability, retrieval, storage, observation, execution, traceability, and framework output do not create authority. Authority requires Ojas admission, scope, label, review, promotion, certification, and audit.**

Expanded form:

> **Nothing becomes authority merely because a framework produced it, a package stored it, a retriever found it, a trace observed it, a tool returned it, or an agent generated it. Ojas must admit, scope, label, review, promote, certify, and record it.**

---

## 3. Why this invariant exists

Agent systems fail when they confuse capability with authority.

Examples:

| Mistaken assumption | Ojas correction |
|---|---|
| The agent generated it, so it is true | Generation is not authority |
| The memory backend stored it, so it is trusted | Storage is not authority |
| The retriever found it, so the agent may see it | Retrieval is not context permission |
| The tool returned it, so it is authoritative | Tool output must be source-trust classified |
| The trace recorded it, so it was allowed | Observation is not approval |
| The framework supports it, so Ojas permits it | Capability is not scope |
| The module is available, so it can run | Availability is not certification |
| The user said it, so it is policy | User input is not enterprise authority |

---

## 4. Non-authority states

These states may be useful, but they are non-authoritative by default.

| State | Meaning |
|---|---|
| Raw observation | Unclassified input or event |
| Working state | Temporary run-local execution state |
| Session context | Short-lived continuity context |
| Tool result | Evidence candidate from a tool |
| Retrieved memory | Candidate material found by retrieval |
| Graph relation | Inferred or stored relationship |
| Vector similarity | Relevance signal |
| Trace event | Observation of runtime behavior |
| Draft output | Agent-generated proposal |
| Candidate lesson | Possible learning pending admission/review |
| Open-origin memory | Relaxed-variant memory that cannot be inherited as stricter authority |

---

## 5. Authority-creating paths

Only governed paths can create or recognize authority.

| Path | Required controls |
|---|---|
| Memory admission | source trust, type, tenant, scope, security/privacy checks |
| Context inclusion | scope, authority label, revocation, expiry, source trust |
| Tool authorization | allowed tool, allowed action, argument validation, risk/approval |
| Semantic promotion | evidence, reviewer/external authority, effective scope, versioning |
| Identity activation | manifest/deployment authority, scope constraints, variant rules |
| Proposal approval | review decision, evidence pack, authority boundary, audit |
| Revocation | revocation authority, affected scope, downstream invalidation |
| Certification | approved capability for a scope/variant/risk class |

---

## 6. Derived invariants

### 6.1 Scope invariant

> **Scope is assigned by Ojas and policy. Scope is never widened by the agent.**

### 6.2 Memory invariant

> **Memory storage is not memory authority.**

### 6.3 Context invariant

> **Memory retrieval does not equal context permission. Ojas must approve what enters the agent context.**

### 6.4 Proposal invariant

> **Agent output is a proposal unless a separate approved authority path grants final authority.**

### 6.5 Projection invariant

> **Graph links, vector embeddings, indexes, and search caches inherit authority from source records; they do not create authority.**

### 6.6 Review invariant

> **The agent cannot approve its own authority.**

### 6.7 Promotion invariant

> **Semantic promotion is an authority-creating event and must be fully auditable.**

### 6.8 Revocation invariant

> **Authority must remain revocable, explainable, and scoped.**

---

## 7. Authority labels

Ojas should distinguish authority levels explicitly.

Suggested core labels:

| Label | Meaning |
|---|---|
| `NO_AUTHORITY` | May support execution but cannot guide future authority |
| `NON_AUTHORITY_EVIDENCE` | Evidence record only |
| `CANDIDATE_AUTHORITY` | May become authority after review |
| `SELF_PROMOTED_OPEN` | Open-only relaxed memory; not stricter authority |
| `REVIEW_PENDING` | Awaiting approval |
| `MEDIUM_REVIEWER_APPROVED` | Approved semantic/domain authority |
| `HIGH_IDENTITY_AUTHORITY` | Manifest/deployment/agent identity authority |
| `REVOKED_AUTHORITY` | Previously active but no longer usable |
| `EXPIRED_AUTHORITY` | Time-bound authority no longer active |
| `QUARANTINED` | Unsafe or uncertain; cannot be used |

---

## 8. Authority and variants

| Variant | Authority posture |
|---|---|
| Open | permissive experimentation; relaxed artifacts labeled clearly |
| Standard | governance required for durable memory, tools, and review-triggering outputs |
| Regulated | strict admission, review, evidence, revocation, and certification required |

Variant rule:

> **A looser-origin artifact cannot be consumed as stricter authority without governed promotion.**

---

## 9. Authority violations

| Violation | Example |
|---|---|
| Storage-as-authority | Memory stored directly as semantic truth |
| Retrieval-as-permission | Retrieved material injected into prompt without filtering |
| Tool-output-as-truth | External result treated as policy |
| Trace-as-approval | Logged event treated as authorized event |
| Open-to-Regulated inheritance | Open self-promoted memory used in Regulated decision |
| Agent self-scope expansion | Agent adds tools or tenant access |
| Agent self-promotion | Agent approves its own semantic memory |
| Projection-as-truth | Graph/vector inference treated as approved fact |
| Feedback-as-learning | Reviewer comment becomes rule without admission |
| Simulation-as-production | Test result used as production authority |

---

## 10. Canonical short forms

Use these consistently:

> **Capability is not authority.**

> **Retrieval is not context permission.**

> **Storage is not semantic authority.**

> **Observation is not approval.**

> **The agent may propose; Ojas governs authority.**
