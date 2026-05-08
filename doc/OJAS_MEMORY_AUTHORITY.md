# OJAS Memory Authority Model

**Status:** Core governance draft  
**Scope:** Core memory types, extended memory classes, memory authority states, admission, promotion, revocation, and projections.

---

## 1. Load-bearing memory principle

> **Storage, retrieval, graph linkage, vector relevance, or framework memory does not create authority. Authority requires Ojas admission, scope, source trust, review, promotion, and revocation.**

Short form:

> **Memory storage is not memory authority.**

---

## 2. Core memory types

| # | Memory type | Purpose | Authority | Writer |
|---:|---|---|---|---|
| 1 | Working Memory | current-run scratchpad | none | runtime/agent during run |
| 2 | Session Memory | short-lived continuity | none, privacy-governed | runtime/session |
| 3 | Episodic Memory | append-only events/outcomes/corrections | evidence, no direct authority | runtime/audit pipeline |
| 4 | Semantic Memory | approved stable knowledge | medium authority | review-gated process |
| 5 | Procedural Memory | governed strategy/outcome statistics | limited structural authority | governed stats pipeline |
| 6 | Identity Memory | agent identity, scope, tools, constraints | high authority | manifest/deployment authority |

---

## 3. Working Memory

Working Memory is the current-run scratchpad.

Examples:

- intermediate calculations
- temporary plan
- active tool result being processed
- transient chain state
- temporary notes

Scope:

```text
run-scoped
```

Rule:

> Working Memory can help execution, but it has no durable authority and is discarded unless admitted elsewhere.

---

## 4. Session Memory

Session Memory supports short-term continuity.

Examples:

- current user task
- temporary preference
- incomplete form state
- recent conversation context

Scope:

```text
tenant + session + user/context
```

Rule:

> Session Memory may improve continuity, but it cannot become durable business knowledge without admission and promotion.

Privacy requirements:

- purpose limitation
- retention/TTL
- deletion/erasure path where applicable
- tenant/user scope
- no broad sharing unless explicitly approved

---

## 5. Episodic Memory

Episodic Memory records what happened.

Examples:

- tool call happened
- reviewer rejected proposal
- user corrected answer
- run failed
- policy denied action
- memory was revoked

Rule:

> Episodic Memory is evidence of past events. It does not decide future policy.

---

## 6. Semantic Memory

Semantic Memory stores approved stable knowledge.

Examples:

- approved rule
- domain fact
- policy interpretation
- frozen architecture decision
- reviewer-approved lesson

Required controls:

- source trust
- scope
- reviewer/external authority
- effective date
- expiry/review date
- versioning
- revocation path
- audit evidence

Rule:

> Semantic Memory cannot be self-authored by the agent in governed variants.

---

## 7. Procedural Memory

Procedural Memory stores governed strategy statistics.

Examples:

- retrieval strategy success rates
- tool sequence effectiveness
- escalation path performance
- failure/retry outcome statistics

Rule:

> Procedural Memory may prefer among allowed actions. It cannot authorize new actions, tools, policies, or scopes.

---

## 8. Identity Memory

Identity Memory defines the agent.

Includes:

- agent ID
- role
- purpose
- variant
- allowed tools
- blocked tools
- handoff policy
- memory policy
- scope constraints
- review requirements
- risk class

Rule:

> The agent cannot rewrite its own identity, scope, tools, or governance constraints at runtime.

---

## 9. Extended memory model

Extended memory classes specialize core memory types.

They are not independent authority systems.

### Rule

> Every extended memory class must map back to a core memory type and inherit that type’s admission, authority, scope, review, retention, and revocation rules.

---

## 10. Consolidated extended memory classes

To avoid taxonomy explosion, Ojas uses a reduced extended memory set.

| Extended class | Base type | Purpose |
|---|---|---|
| Scoped Knowledge Memory | Semantic or Episodic depending item | project/domain/team/org knowledge with explicit scope tier |
| Shared Memory | Semantic/Episodic | cross-user or cross-agent shared context |
| Evidence Memory | Episodic | evidence used for proposals, reviews, audits |
| Tool Result Memory | Episodic | tool output stored as evidence |
| Compliance Memory | Episodic or Semantic | compliance evidence or approved mappings |
| Feedback Memory | Episodic → Candidate Lesson | user/reviewer corrections before learning |
| Simulation Memory | Episodic/Test | dry-run/test/red-team outputs |
| Policy Shadow Memory | Episodic | policy decision traces and explanations |
| User Preference Memory | Session or Semantic | user-specific preferences, privacy-governed |
| Package / Backend Memory | Semantic / Identity-adjacent | compatibility, certification, eligibility knowledge |

---

## 11. Scoped Knowledge Memory

Scoped Knowledge Memory consolidates project, domain-pack, team, and organization memory.

### Scope tiers

```text
project
domain-pack
team
organization
tenant
```

Mapping examples:

| Item | Core type |
|---|---|
| project decision | Semantic |
| project discussion log | Episodic |
| domain rule | Semantic |
| organization policy | Semantic with stronger admission |
| working note | Working until admitted |

### Organization-wide rule

> Organization-wide memory requires organization-authorized source, explicit versioning, reviewer approval, effective date, expiry/review date, and revocation path.

---

## 12. Shared Memory

Shared Memory is cross-user or cross-agent memory.

Risk:

- cross-user leakage
- cross-agent authority pollution
- role mismatch
- stale shared assumptions

Rule:

> Shared Memory requires explicit shared scope and cannot be inferred from collaboration alone.

---

## 13. Evidence Memory

Evidence Memory stores material used to support proposals, reviews, approvals, audits, or exports.

Base type:

```text
Episodic
```

Rule:

> Evidence supports decisions; it does not itself become the decision.

---

## 14. Tool Result Memory

Tool Result Memory stores outputs from tools.

Source trust labels:

```text
TOOL_ENTERPRISE_SYSTEM
TOOL_EXTERNAL_PUBLIC
TOOL_UNTRUSTED
TOOL_SENSITIVE
TOOL_GENERATED
```

Rule:

> Tool output is evidence first, not authority.

---

## 15. Compliance Memory

Compliance Memory stores compliance-related evidence and approved mappings.

Examples:

- control mapping
- audit artifact reference
- risk assessment output
- model inventory metadata
- review attestation
- regulatory framework mapping

Rule:

> Ojas may produce compliance memory as runtime evidence, but it does not become the enterprise GRC system of record.

---

## 16. Feedback Memory

Feedback Memory stores corrections, ratings, rejections, approvals, and reviewer/user comments.

Transition:

```text
Episodic → Candidate Lesson → Semantic or Procedural after review
```

Rule:

> Feedback is not automatically learning.

---

## 17. Simulation Memory

Simulation Memory stores test, dry-run, red-team, and regression outputs.

Rule:

> Simulation Memory may inform validation, but production agents cannot treat simulation output as production authority.

---

## 18. Policy Shadow Memory

Policy Shadow Memory records how policy was applied.

Examples:

- allow/deny decision
- matched policy
- policy version
- guardrail decision
- explanation

Rule:

> Policy Shadow Memory records how policy was applied; it does not rewrite policy.

---

## 19. User Preference Memory

User Preference Memory stores user-specific preferences.

Required privacy controls:

- purpose limitation
- consent or permitted basis where required
- retention policy
- deletion/erasure path
- data residency tag where required
- subject-access support
- sharing prohibition unless explicitly approved

Rule:

> User Preference Memory is privacy-governed and purpose-limited. It cannot become broad enterprise authority.

---

## 20. Package / Backend Memory

Package/Backend Memory stores governance knowledge about external capabilities at a conceptual level.

Examples:

- backend eligible for Standard
- backend not eligible for Regulated
- certification expired
- package audit decision
- compatibility restriction
- vulnerability discovered
- module revoked

Rule:

> Package/Backend Memory influences what may run, but must itself be reviewed, versioned, and revocable.

---

## 21. Supporting projections

Supporting projections are not authority.

| Projection | Purpose | Authority |
|---|---|---|
| Graph Memory | relationships among memories/evidence/tools/policies | inherited |
| Vector / Embedding Memory | similarity search | inherited |
| Index / Search Cache | retrieval acceleration | inherited |
| Semantic cache | performance optimization | inherited and expiry-bound |

Rules:

> Retrieval score is not authority.  
> Graph inference does not become authority automatically.  
> Cache hit is not authority.

---

## 22. Memory authority states

```text
RAW_OBSERVATION
CLASSIFIED
ADMISSION_PENDING
ADMITTED_NON_AUTHORITY
CANDIDATE_LESSON
PENDING_REVIEW
SEMANTIC_APPROVED
SELF_PROMOTED_OPEN
QUARANTINED
STALE
REVOKED
EXPIRED
ARCHIVED
```

| State | Meaning |
|---|---|
| `RAW_OBSERVATION` | observed but not classified |
| `CLASSIFIED` | source/type/scope classified |
| `ADMISSION_PENDING` | waiting admission decision |
| `ADMITTED_NON_AUTHORITY` | stored but not authoritative |
| `CANDIDATE_LESSON` | possible future authority |
| `PENDING_REVIEW` | awaiting reviewer/external approval |
| `SEMANTIC_APPROVED` | approved authority |
| `SELF_PROMOTED_OPEN` | Open-only, non-Regulated authority |
| `QUARANTINED` | unsafe or uncertain |
| `STALE` | requires revalidation |
| `REVOKED` | must not be used |
| `EXPIRED` | inactive due to time |
| `ARCHIVED` | retained for audit only |

---

## 23. Admission decision factors

Ojas considers:

- source trust
- tenant scope
- memory type
- extended class
- data classification
- PII/security risk
- poisoning indicators
- conflict with existing authority
- variant
- package/backend origin at conceptual level
- reviewer requirement
- expiry policy
- revocation compatibility

---

## 24. Decision outcomes

| Outcome | When used |
|---|---|
| `ADMIT_WORKING` | current run only |
| `ADMIT_SESSION` | session continuity |
| `ADMIT_EPISODIC` | event/evidence record |
| `CREATE_CANDIDATE_LESSON` | possible future learning |
| `ROUTE_TO_REVIEW` | authority-affecting or risky |
| `QUARANTINE` | unsafe/uncertain |
| `REJECT` | disallowed, poisoned, out of scope, or invalid |
| `PROMOTE_SEMANTIC` | reviewer/external authority approved |
| `DOWNGRADE` | conflict, stale source, variant mismatch |
| `REVOKE` | no longer valid or unsafe |
| `EXPIRE` | TTL/effective date ended |

---

## 25. Allowed learning path

```text
Working observation
  ↓
Classified observation
  ↓
Session or Episodic Memory
  ↓
Candidate Lesson
  ↓
Review / validation
  ↓
Semantic Memory
```

Prohibited shortcut:

```text
Agent output
  ↓
Semantic Memory
```

---

## 26. Variant behavior

| Memory area | Open | Standard | Regulated |
|---|---|---|---|
| Working | allowed | allowed | allowed |
| Session | allowed | TTL/privacy | strict TTL/privacy |
| Episodic | allowed | admission required | strict admission |
| Candidate Lesson | allowed | review path | formal review |
| Semantic | self-promotion allowed only with label | reviewer required for risky memory | reviewer/external authority required |
| Procedural | experimental | governed stats | governed stats + validation |
| Identity | manifest-owned | manifest-owned | strict manifest/deployment-owned |
| Projections | allowed | filtered | filtered + audited |

---

## 27. Memory invariants

1. Memory storage is not authority.
2. Retrieval is not context permission.
3. Vector relevance is not authority.
4. Graph inference is not authority.
5. Feedback is not learning until admitted.
6. Tool output is evidence first.
7. Simulation output is not production authority.
8. Semantic promotion is an authority event.
9. Identity Memory is not runtime mutable.
10. Extended memory classes inherit core memory authority rules.
