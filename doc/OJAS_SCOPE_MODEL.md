# OJAS Scope Model

**Status:** Core governance draft  
**Scope:** Agent, tenant, user, memory, tool, context, handoff, environment, variant, package/backend, and time scope.

---

## 1. Master scope invariant

> **Scope is assigned by Ojas and policy. Scope is never widened by the agent.**

The agent may request operations inside scope. It may not expand its own authority, tenant, data, tool, memory, handoff, or context scope.

---

## 2. Scope is multidimensional

Ojas scope is not a single flag.

| Scope type | Meaning |
|---|---|
| Tenant scope | Which tenant data can be accessed |
| User scope | Which user/session context applies |
| Domain scope | Which business domain applies |
| Data scope | Which records, documents, fields, and datasets are accessible |
| Tool scope | Which tools/actions may be invoked |
| Memory scope | Which memories may be read, written, promoted, or revoked |
| Context scope | Which material may enter the model context |
| Handoff scope | Which agents/workflows may be called |
| Environment scope | dev, qa, staging, production |
| Variant scope | Open, Standard, Regulated |
| Package/backend scope | Which backends/modules are eligible at a conceptual level |
| Time scope | effective dates, TTLs, expiry windows |
| Purpose scope | why data/memory/tool is being used |
| Review scope | who may review and decide |
| Evidence scope | what evidence may be included/exported |

---

## 3. Scope lifecycle

```text
UNBOUND
  ↓
DECLARED
  ↓
VALIDATED
  ↓
ENVIRONMENT_BOUND
  ↓
RUN_BOUND
  ↓
ACTIVE
  ↓
CLOSED
  ↓
ARCHIVED
```

| State | Meaning |
|---|---|
| `UNBOUND` | no runtime scope exists |
| `DECLARED` | intended scope is stated |
| `VALIDATED` | scope template is accepted |
| `ENVIRONMENT_BOUND` | scope tied to environment/deployment |
| `RUN_BOUND` | immutable run-specific scope snapshot created |
| `ACTIVE` | scope is enforced during execution |
| `CLOSED` | no new operations allowed under that scope |
| `ARCHIVED` | retained for audit/replay |

---

## 4. Scope binding layers

```text
Declared scope
  ↓
Deployment scope
  ↓
Run scope
  ↓
Operation scope
  ↓
Artifact scope
```

### 4.1 Declared scope

The agent definition says what scope it expects.

### 4.2 Deployment scope

The environment narrows what is allowed.

### 4.3 Run scope

A run gets a fixed snapshot.

### 4.4 Operation scope

Each tool call, memory read, memory write, handoff, or context assembly action is checked.

### 4.5 Artifact scope

Outputs, evidence, and memories receive scope metadata.

---

## 5. Run scope snapshot

A run scope snapshot should include:

```json
{
  "tenantId": "tenant-a",
  "agentId": "invoice-review-agent",
  "agentVersion": "1.2.0",
  "variant": "standard",
  "environment": "production",
  "domainScopes": ["invoicing"],
  "allowedActions": ["read_invoice", "draft_response"],
  "blockedActions": ["release_payment", "delete_invoice"],
  "memoryReadScopes": ["tenant-a:invoicing:approved"],
  "memoryWriteScopes": ["working", "session", "episodic"],
  "semanticPromotion": "reviewer_required",
  "handoffScopes": ["compliance-review-agent"]
}
```

The snapshot is immutable for the run.

---

## 6. Scope enforcement points

| Operation | Scope check |
|---|---|
| Context assembly | tenant, memory, authority, source, revocation, variant |
| Tool call | tenant, user, tool, action, arguments, approval |
| Memory read | tenant, type, authority, source, expiry, revocation |
| Memory write | tenant, type, source trust, target authority, privacy/security |
| Handoff | source agent, target agent, tenant, domain, handoff policy |
| Output generation | scope boundaries, data leakage, authority claims |
| Evidence export | tenant, destination, redaction, compliance boundary |
| Review | reviewer role, tenant, domain, authority level |
| Promotion | reviewer authority, source authority, effective scope |
| Revocation | revocation authority, affected scope, downstream effect |

---

## 7. Scope widening is prohibited

Prohibited examples:

```text
agent changes its own tenant
agent adds a tool
agent removes a blocked tool
agent requests broader memory retrieval and grants it to itself
agent promotes Open memory into Regulated authority
agent activates a broader handoff target
agent expands purpose from support to billing
agent treats retrieved data as in-scope without Ojas approval
```

Allowed examples:

```text
agent requests escalation
agent emits scope expansion request
agent asks for review
agent proposes manifest update
agent proposes a new tool requirement
```

### Rule

> A request for scope is not scope.

---

## 8. Scope expansion request

If an agent needs more scope, it produces an artifact.

```json
{
  "artifactType": "SCOPE_EXPANSION_REQUEST",
  "agentId": "invoice-review-agent",
  "requestedScope": {
    "tools": ["payment_status_lookup"]
  },
  "reason": "Required to answer customer billing status",
  "evidenceRefs": ["ev_123"],
  "authority": "NO_SELF_AUTHORITY",
  "requiresReview": true
}
```

The request must be approved externally.

---

## 9. Scope and memory

| Memory operation | Required scope |
|---|---|
| Read working memory | same run |
| Read session memory | same tenant/session and allowed agent |
| Read episodic memory | tenant/domain/source/retention allowed |
| Read semantic memory | authority label valid for tenant/domain/variant |
| Write candidate lesson | write permission and source trust |
| Promote semantic memory | reviewer/external authority |
| Revoke memory | revocation authority |
| Share memory | explicit shared/team/org scope |

---

## 10. Scope and context

Context inclusion is a scope decision.

A retrieved item may be excluded because:

- wrong tenant
- wrong domain
- expired
- revoked
- wrong variant origin
- insufficient authority
- untrusted source
- PII not permitted
- review pending
- purpose mismatch

Rule:

> Retrieval is not context permission.

---

## 11. Scope and package/backend availability

At a conceptual level, Ojas must distinguish:

```text
available
requested
trusted
eligible
active
```

A backend/module/package being available does not mean:

- it was requested
- it is trusted
- it is eligible for the current variant/risk class
- it is active in this run
- it produced authority

### Rule

> Backend availability is not execution authority.

---

## 12. Scope closure

When a run closes:

- tool scope closes
- handoff scope closes
- run-bound memory write scope closes
- context assembly scope closes
- late backend/tool/memory events are rejected
- run scope snapshot is archived

Rule:

> No operation may use a closed run scope.

---

## 13. Scope violations

| Violation | Ojas response |
|---|---|
| cross-tenant memory read | deny, alert, possible suspension |
| blocked tool call | deny and record |
| unapproved semantic promotion | quarantine/reject |
| backend not eligible for variant | deny activation/use |
| late memory write after closure | reject and log lifecycle violation |
| unmanifested handoff | deny and escalate |
| purpose mismatch | exclude context and record |
| unauthorized context inclusion | block and record |

---

## 14. Scope invariants

1. Scope is never widened by the agent.
2. Run scope is immutable once created.
3. Context inclusion requires scope approval.
4. Memory authority is scoped, never global by default.
5. Tool permission is scoped by tenant, action, risk, and variant.
6. Backend availability is not execution authority.
7. Review authority is scoped.
8. Closed, suspended, or retired scope cannot be reused.
