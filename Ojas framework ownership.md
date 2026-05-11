# Ojas Framework — Ownership Specification

**Status:** Final
**Version:** 1.0 (derived from Framework v0.3.2c)
**Scope:** Defines what the **Ojas Framework** owns. Companion document `OJAS_RUNTIME_OWNERSHIP.md` defines what the **Ojas Runtime** owns.

---

## 0. Purpose and Positioning

This document specifies, in depth, every concern the Ojas Framework owns. It is the authoritative ownership reference for the Framework layer of Ojas.

**The one-line rule:**

> **The Framework defines mechanism contracts. The Runtime implements those contracts and binds them to agents, IAM, tools, storage, admin operations, and deployments.**

The Framework owns *contracts, invariants, lifecycles, composition rules, audit requirements, identity field structure, policy-artifact governance, and conformance assertions.* It does not execute, store, bind to IAM, or implement any specific policy semantics.

**Public positioning:**

> The Ojas Framework is a policy-agnostic, runtime-agnostic specification for the execution mechanics of governed AI agents. It defines how policies register, how hooks fire, how composition works, how identity binds, how audit persists, and what termination authorities exist. It does not specify which policies should exist, how policies are stored, or how agents execute.

---

## 1. Versioning and Release Discipline

### 1.1 What the Framework owns

| Aspect | Detail |
|---|---|
| Spec version | The Framework's own version number (currently v0.3.2c, advancing to v1.0.0 at first final-lock) |
| Conformance contract version | The version of the conformance assertion suite (§17 of the spec) |
| Versioning scheme | Strict semver; minor versions are backward-compatible additions; major versions may break |
| Release cadence | Slow — every change requires the assessment → review → resolution discipline |
| Breaking change policy | Requires ADR + cross-spec impact analysis; no silent breaking changes |
| Lock discipline | Candidate-lock → contradiction review → final-lock (no premature lock) |

### 1.2 What the Framework does NOT own

- The version of any Runtime implementation
- The version of any specific PolicyType implementation
- The version of any deployment configuration
- Release cadence of conformant Runtimes

### 1.3 Independent versioning invariant

```
Ojas Framework version != Ojas Runtime version.
A Runtime may support multiple Framework spec versions.
A Framework version may have multiple conformant Runtime implementations.
```

A Runtime declares which Framework version(s) it conforms to. The Framework does not declare which Runtimes implement it.

---

## 2. Core Concepts Owned by the Framework

### 2.1 Terminology vocabulary

The Framework owns the canonical terminology used across the Ojas architecture:

| Term | Framework definition |
|---|---|
| **PolicyType** | The reusable implementation contract declared once per `(policyTypeId, version)` |
| **PolicyInstance** | A tenant-scoped configured registration of a PolicyType |
| **PolicyConfiguration** | A versioned, tenant-scoped configuration value bound to a PolicyInstance |
| **Hook** | A synchronous, inbound decision point in runtime execution |
| **Event** | An outbound, asynchronous broadcast that something has happened (past-tense naming) |
| **Listener** | A subscriber to events; observes, never decides |
| **Registry** | A tenant-scoped store holding artifacts with lifecycle state |
| **Lifecycle** | Seven-state machine for policy artifacts; five-state machine for configurations |
| **Namespace** | Three-tier authority hierarchy: `ojas.core.*`, `ojas.stdlib.*`, `vendor.*` |
| **runId** | Unique identifier for a single agent invocation |
| **auditRecordId** | Stable identifier for an audit record, used as idempotency key |
| **AgentIdentity** | First-class identity field on every boundary artifact and hook context |

### 2.2 Architectural invariants (frozen)

The Framework enforces these immutable rules across all conformant deployments:

1. **Path C** — Agents propose, evidence proves, external authority decides. Ojas emits `EnvelopeDecision`; it never emits `ApprovalDecision`.
2. **ALL-DENY Composition** — If any policy returns DENY at a hook, the operation is rejected. No priority or weight can override a DENY.
3. **Listener Isolation** — Listeners are strictly observational. Return values are discarded by the framework.
4. **Audit as Infrastructure** — Audit trails are framework-injected, not policy-optional. Audit cannot be silently lost.
5. **Tenant Isolation** — Cross-tenant data access, registry visibility, and execution bleed are prohibited at the framework level.
6. **AgentIdentity is First-Class and Framework-Bound** — `AgentIdentity` is present on every artifact crossing the boundary; the runtime binds it from authenticated context, not from agent self-declaration.
7. **Termination is a Framework Primitive** — Termination does not require agent cooperation. Policies may *request* deny effects (including termination); the framework decides whether to apply them.

These invariants are non-negotiable. A Runtime that violates any of them is non-conformant.

---

## 3. Namespace and Registrar Tiers

### 3.1 What the Framework owns

| Namespace | Authority | Registrar |
|---|---|---|
| `ojas.core.*` | Framework-frozen | Framework maintainer |
| `ojas.stdlib.*` | Distribution-owner | Standard-library distribution maintainer |
| `vendor.*` | Deployment-owner | Adopter-defined |

`ojas.*` is reserved; third parties cannot register policies, hooks, listeners, or reference data under any `ojas.*` prefix.

### 3.2 Namespace ownership rules

- The Framework defines the **tier structure** and the **registrar authority model**.
- The Framework specifies that namespace tiers govern *registration authority*, not data visibility.
- The Framework does **not** specify *who* a deployment's `vendor.*` registrar is — that is a Runtime/deployment concern.
- The Framework does **not** define how a tier-2 (`ojas.stdlib.*`) registrar is appointed — that is distribution-owner policy.

### 3.3 What the Framework does NOT own

- The list of `ojas.stdlib.*` policies (owned by the standard-library distribution maintainer)
- The list of `vendor.*` policies (owned by the deployment)
- The specific implementation of any policy in any namespace

---

## 4. PolicyType Contract

### 4.1 Schema owned by the Framework

```yaml
policyTypeId: <namespaced-id>
version: <semver>
schemaRef: <url-or-path-to-config-schema>

enforcementPoints:
  - component: <runtime-component-id>
    hook: <hook-name>
    decisionFamily: ALLOW_DENY | REWRITE | CAP | RECORD
    priority: <integer-or-null>

errorCodes:
  - code: <namespaced-error-code>
    severity: REJECT | ESCALATE | WARN

evidenceContract:
  recordsPerInvocation: [...]
  redactionRules: [...]

failureMode:
  onTimeout: DENY | WARN | ESCALATE
  onError: DENY | WARN | ESCALATE
  onMalformedDecision: DENY

denyEffects:
  - REJECT_OPERATION
  - REQUEST_TERMINATE_RUN
  - REQUEST_SUSPEND_AGENT
  - REQUEST_QUARANTINE_RUN

owaspMapping:
  - category: ASI01..ASI10
    coverage: PRIMARY | SECONDARY
```

### 4.2 decisionFamily and allowed returns

The Framework owns the mapping between `decisionFamily` and allowed return decisions:

| `decisionFamily` | Allowed returns | Semantic meaning |
|---|---|---|
| `ALLOW_DENY` | `ALLOW`, `DENY` | Pure gate |
| `REWRITE` | `ALLOW`, `DENY`, `REWRITE` | May modify the operation |
| `CAP` | `ALLOW`, `DENY`, `CAP` | May impose a quantitative cap |
| `RECORD` | `RECORD` | Observation only |

### 4.3 Runtime method contract

Every PolicyType implementation provides three methods (signatures owned by Framework):

- `declare()` — returns the PolicyType schema metadata
- `evaluate(hook, context)` — returns a decision object
- `validate(value)` — validates configuration data against `schemaRef`

### 4.4 What the Framework does NOT own

- Specific PolicyType implementations
- The semantics of "ALLOW" or "DENY" beyond the runtime decision (business semantics are policy-specific)
- The programming language or runtime in which a PolicyType is implemented
- The contents of `evidenceContract` payloads

---

## 5. PolicyInstance Contract

### 5.1 Schema owned by the Framework

```yaml
policyInstanceId: <tenant-scoped-id>
policyTypeId: <namespaced-id>
typeVersion: <semver>
activeConfigurationId: <config-id>

applicability:
  environments: []
  domainPackIds: []
  agentIds: []
  runTags: []
  resourceSelectors: []
  condition: null
  globalApplicability: false
  allowOverlap: false
  overlapJustification: null
```

### 5.2 Applicability semantics owned by the Framework

- **Matching rule**: every non-empty selector field must match the corresponding context value
- **Default applicability**: empty selectors match everything (subject to regulated-profile collision rule)
- **Regulated-profile collision rule**: when multiple instances of the same PolicyType coexist, every instance must declare explicit applicability OR opt into overlap with `globalApplicability: true` + `allowOverlap: true` + non-empty `overlapJustification`
- **Multiple matching instances**: all matching instances evaluate under ALL-DENY composition

### 5.3 Activation gate owned by the Framework

A PolicyInstance can transition to `ACTIVE` only if (the **activation gate**):

- `activeConfigurationId` is non-null
- The referenced PolicyConfiguration is in state `ACTIVE` with matching `policyInstanceId`
- Priority and applicability-collision checks pass against the active policy set at activation time

Activation attempts violating the gate are rejected with `ojas.framework.activation-gate-violation` or `ojas.framework.applicability-collision`.

### 5.4 Uniqueness rules

- `policyInstanceId` is unique within a tenant
- A PolicyInstance can be registered without `activeConfigurationId` (DRAFT state); it must have one to activate

### 5.5 What the Framework does NOT own

- Storage of PolicyInstances (Runtime concern)
- Query interfaces for the PolicyInstance registry (Runtime concern)
- The deployment-profile classification (Runtime concern)

---

## 6. PolicyConfiguration Contract

### 6.1 Schema owned by the Framework

```yaml
policyConfigurationId: <tenant-scoped-id>
policyInstanceId: <policy-instance-ref>
version: <semver>
state: DRAFT | CANDIDATE | ACTIVE | SUPERSEDED | RETIRED
configurationDigest: <hash>
configurationData: <value-validated-against-PolicyType.schemaRef>
effectiveFrom: <timestamp>
```

### 6.2 Five-state lifecycle owned by the Framework

| State | Meaning | Allowed transitions |
|---|---|---|
| `DRAFT` | Authored but not yet validated | → CANDIDATE, → RETIRED |
| `CANDIDATE` | Validated against `schemaRef`; pending activation | → ACTIVE, → DRAFT, → RETIRED |
| `ACTIVE` | In use by a PolicyInstance | → SUPERSEDED |
| `SUPERSEDED` | Replaced by newer ACTIVE configuration | → RETIRED |
| `RETIRED` | Removable per retention policy | (terminal) |

### 6.3 Invariants owned by the Framework

- Only one PolicyConfiguration per PolicyInstance may be `ACTIVE` at a time
- `DRAFT` and `CANDIDATE` configurations are not evaluated by hooks
- ACTIVE → SUPERSEDED transition is triggered by activation of another configuration; not a direct admin action
- `configurationDigest` must match the canonical hash of `configurationData`

### 6.4 Bootstrap and activation sequence (owned by Framework)

The five-step bootstrap that resolves the PolicyInstance ↔ PolicyConfiguration circular dependency:

1. Register PolicyInstance in `DRAFT` with `activeConfigurationId: null`
2. Register PolicyConfiguration referencing the DRAFT PolicyInstance
3. Validate the configuration; transition `DRAFT → CANDIDATE`
4. Promote `CANDIDATE → ACTIVE`
5. Bind PolicyInstance's `activeConfigurationId` to the ACTIVE PolicyConfiguration
6. Promote PolicyInstance through `DRAFT → TESTING → SIMULATION → ACTIVE`

### 6.5 What the Framework does NOT own

- Configuration storage backend (Runtime concern)
- Configuration management UI (Runtime concern)
- GitOps integration (Runtime concern)

---

## 7. Hook Registry

### 7.1 Hook contract owned by the Framework

```yaml
hookId: <namespaced-id>
version: <semver>
inputSchema: <schema>
outputSchema: <schema>
category: EVALUATION | LIFECYCLE | CROSS_CUTTING | CONDITIONAL
```

### 7.2 Hook categories owned by the Framework

| Category | Purpose | Decisional? |
|---|---|---|
| `EVALUATION` | Gate runtime actions | Yes (pre); No (post) |
| `LIFECYCLE` | Policy reacts to its own state changes | No (RECORD only) |
| `CROSS_CUTTING` | Fire across components, not at specific actions | Yes (pre); No (post) |
| `CONDITIONAL` | Fire under specific runtime conditions | Yes (pre); No (post) |

All decisional hooks have `pre-` and `post-` versions. `pre-` is decisional; `post-` is observational only.

### 7.3 Core hook set owned by the Framework

The Framework defines a minimal core hook set every conformant Runtime must provide:

- `pre-tool-call`, `post-tool-call`
- `pre-file-read`, `post-file-read`
- `pre-mutation`, `post-mutation`
- `pre-emission`, `post-emission`
- `run.onStart`, `run.onEnd`, `run.onError`
- `policy.onRegister`, `policy.onActivate`, `policy.onDeactivate`, `policy.onRetire`
- `escalation.onRequest`
- `error.onStructuredError`

### 7.4 What the Framework does NOT own

- Implementation of hook firing (Runtime concern)
- The exhaustive hook catalog beyond core (Runtime/distribution extensions)
- Network protocols for hook dispatch (Runtime concern)

---

## 8. Hook Evaluation and Composition

### 8.1 Evaluation flow owned by the Framework

The Framework owns the canonical 10-step evaluation flow at every `pre-` hook:

1. Runtime hits a `pre-` hook
2. Framework collects all `ACTIVE` PolicyInstances whose PolicyType declares that hook AND whose applicability matches
3. Order policies by `priority` ascending; ties broken by `policyTypeId` then `policyInstanceId`
4. Evaluate ALL matching policies against the **original** hook context (resolving configuration per §9)
5. Compose results: ALL-DENY first, then REWRITE patch composition, then CAP
6. If final decision is DENY, abort operation; apply `denyEffects`
7. If REWRITE patches exist and no DENY, apply patches in priority order
8. If CAP, apply most-restrictive cap
9. Close audit event; durably persist; emit composition and evaluation events
10. Fire `post-` hook (observational)

### 8.2 Composition rules owned by the Framework

**ALL-DENY (evaluate-all):**
- All matching policies evaluate against the original hook context
- Implementations may NOT short-circuit on DENY (conformance requirement)
- If any policy returns DENY → final decision = DENY

**REWRITE patch model:**
- REWRITE policies return declarative patches against the original context
- Framework applies patches in priority order only if no DENY
- Patch ops: `add`, `replace`, `remove`, `redact`, `constrain`
- Cross-priority transformed-path conflicts: regulated profile DENYs; non-regulated profile uses deployment-configured resolution (DENY / SKIP_LATER / APPLY_AGAINST_ORIGINAL)
- Same-priority conflicts: regulated profile DENYs; non-regulated profile is deployment-configurable

**CAP composition:**
- Most-restrictive wins after all REWRITE patches applied

### 8.3 Determinism invariant owned by the Framework

For any given input patch set and deployment configuration, the framework MUST produce the same final operation across implementations and across runs. Non-deterministic patch composition is a conformance failure.

---

## 9. Configuration Resolution (Run Stability)

### 9.1 First-applicability snapshot rule owned by the Framework

PolicyConfiguration resolution is **run-stable per `(runId, policyInstanceId)`**. The first time a PolicyInstance is determined applicable within a run, the Framework snapshots its `ACTIVE` PolicyConfiguration; subsequent evaluations of the same PolicyInstance within the same run use that snapshot.

### 9.2 Snapshot semantics owned by the Framework

- Snapshot key: `(runId, policyInstanceId)`
- Snapshot pins `policyConfigurationId`, `configurationDigest`, `configurationData`
- Snapshot creation is recorded in audit
- If snapshotted configuration is retired during the run, run continues using snapshot; next run fails per missing-configuration rule

### 9.3 Mid-run refresh policy owned by the Framework

- Default: refresh disabled (run-stable behavior)
- Opt-in: deployment may enable mid-run refresh in non-regulated profiles only
- Regulated profiles: refresh rejected at deployment with `ojas.framework.mid-run-refresh-denied`
- Refresh transitions audited via `framework.safeguardTriggered` with subtype `configuration-refresh`

### 9.4 What the Framework does NOT own

- Pre-snapshot optimization strategies (Runtime implementation choice)
- Snapshot storage (Runtime concern)

---

## 10. Lifecycle State Machine

### 10.1 Seven-state lifecycle owned by the Framework (policy artifacts)

```
DRAFT → TESTING → SIMULATION → ACTIVE → DEPRECATED → DISABLED → RETIRED
```

Applied to: PolicyTypes, PolicyInstances, Hooks, Listeners.

### 10.2 Authorized transitions owned by the Framework

```
DRAFT → TESTING              (author promotes; conformance checks pass)
TESTING → SIMULATION         (registrar approves; referenced hooks must be ACTIVE)
SIMULATION → ACTIVE          (registrar approves after simulation evidence)
ACTIVE → DEPRECATED          (registrar marks; successor recorded)
ACTIVE → DISABLED            (ops action OR conformance failure; reason recorded)
DISABLED → ACTIVE            (ops action; conformance re-check required)
DEPRECATED → RETIRED         (migration period elapsed; usage zero)
ANY → RETIRED                (emergency removal; elevated authority)
```

**Important:** `ACTIVE → SIMULATION` is NOT a valid transition. `ACTIVE → DISABLED` may be triggered by conformance failure (error rate, timeout rate, malformed-decision rate, version drift) as defined in §19; it is not solely an ops action.

### 10.3 Per-type gate semantics owned by the Framework

| Type | `ACTIVE → DEPRECATED` gate | `→ RETIRED` gate |
|---|---|---|
| PolicyType | Successor recorded | Migration elapsed; PolicyInstance count = 0 |
| PolicyInstance | Supersession recorded | No active configurations remain |
| PolicyConfiguration | N/A (uses 5-state lifecycle) | (5-state lifecycle) |
| Hook | No successor required | No PolicyType references in `ACTIVE`/`DEPRECATED` |
| Listener | No successor required | Pending events dropped |

### 10.4 Semantic use of states owned by the Framework

- `DEPRECATED` = superseded but functional (successor exists)
- `DISABLED` = removed from runtime evaluation (conformance failure, ops action, remediation)
- These are NOT interchangeable

### 10.5 What the Framework does NOT own

- Agent lifecycle state machine (Runtime owns)
- Run lifecycle state machine (Runtime owns)
- Domain pack lifecycle state machine (Runtime owns)
- Deployment lifecycle state machine (Runtime owns)

---

## 11. Events (Outbound, Asynchronous)

### 11.1 Event guarantees owned by the Framework

- Events fire after action completes (post-decision)
- Framework **durably publishes** events before listener notification
- Per-listener delivery guarantee selected at subscription: `AT_LEAST_ONCE` or `BEST_EFFORT`
- `EXACTLY_ONCE` is not a framework guarantee
- `AT_LEAST_ONCE` is **mandatory** for `audit.*`, `conformance.*`, and `framework.safeguardTriggered` subscriptions
- Event schemas are stable across minor versions
- Every event includes `runId`, `tenantId`, `timestamp`, `eventType`, plus applicable: `hookId`, `policyInstanceIds`, `auditRecordId`, `contextFingerprint`

### 11.2 Event catalog owned by the Framework

| Category | Events |
|---|---|
| Registry — PolicyType | `policytype.registered`, `policytype.versionRegistered`, `policytype.successorRecorded`, `policytype.activated`, `policytype.deprecated`, `policytype.disabled`, `policytype.retired` |
| Registry — PolicyInstance | `policyinstance.registered`, `policyinstance.activated`, `policyinstance.deprecated`, `policyinstance.superseded`, `policyinstance.disabled`, `policyinstance.retired` |
| Registry — PolicyConfiguration | `policyconfiguration.drafted`, `policyconfiguration.promotedToCandidate`, `policyconfiguration.activated`, `policyconfiguration.superseded`, `policyconfiguration.retired` |
| Registry — Hook / Listener | `hook.{registered,activated,deprecated,retired}`, `listener.{registered,activated,deprecated,retired}` |
| Evaluation | `evaluation.{denied,rewritten,capped,errored,slow,malformed}` |
| Composition | `composition.conflictDetected`, `composition.patchConflictDetected`, `composition.transformedPathConflictDetected`, `composition.completed` |
| Audit | `audit.recorded`, `audit.persistenceFailed`, `audit.compositionDivergence` |
| Conformance | `conformance.violation`, `conformance.driftDetected` |
| Framework Safeguard | `framework.safeguardTriggered` |
| Run Lifecycle | `run.terminated`, `run.quarantined` |
| Agent Lifecycle | `agent.suspended`, `agent.disabled` |

### 11.3 Naming discipline owned by the Framework

- All event names use past-tense form
- `*.on*` is reserved for hooks, never events
- PolicyType versions are immutable; framework does not emit `*.versionUpdated`
- Run-scoped lifecycle (`run.*`) is distinct from agent-definition lifecycle (`agent.*`)

### 11.4 What the Framework does NOT own

- Event transport implementation (queue, stream, etc. — Runtime concern)
- Event listener execution infrastructure (Runtime concern)
- Listener retry, backoff, dead-letter handling (Runtime concern)
- Event storage and replay (Runtime concern)

---

## 12. Listeners (Observational Only)

### 12.1 Listener invariants owned by the Framework

- Listeners can never affect runtime decisions
- Return values are discarded by the framework
- Listener exceptions are logged via runtime safeguard channel but never propagate
- No architectural path exists by which a listener can prevent, modify, or delay a runtime action

### 12.2 Listener contract owned by the Framework

```yaml
listenerId: <namespaced-id>
version: <semver>

subscriptions:
  - eventType: <event-type>
    filter: <expression>

delivery:
  mode: ASYNCHRONOUS | BATCHED | FIRE_AND_FORGET
  endpoint: <url-or-queue>
  retryPolicy: <retry-spec>
  deliveryGuarantee: AT_LEAST_ONCE | BEST_EFFORT

failureMode: CONTINUE_ON_LISTENER_ERROR
```

### 12.3 Delivery guarantee constraints owned by the Framework

- `AT_LEAST_ONCE` is default when omitted
- `audit.*`, `conformance.*`, `framework.safeguardTriggered` subscriptions MUST use `AT_LEAST_ONCE`
- `BEST_EFFORT` rejected for these categories at subscription time with `ojas.framework.delivery-guarantee-required`

### 12.4 What the Framework does NOT own

- Queue infrastructure (Runtime concern)
- Retry mechanism implementation (Runtime concern)
- Dead-letter handling (Runtime concern)
- Listener invocation transport (Runtime concern)
- Listener execution sandboxing (Runtime concern)

---

## 13. Registries (Contract Layer)

### 13.1 Registry contracts owned by the Framework

The Framework defines three registries; the Framework owns their **contracts and lifecycle rules**, not their storage:

**PolicyType Registry:**
- Stores PolicyType declarations
- `ojas.core.*` and `ojas.stdlib.*` PolicyTypes: runtime-global, immutable per version
- `vendor.*` PolicyTypes: tenant-scoped
- Lookup by `(policyTypeId, version)`

**PolicyInstance Registry:**
- Tenant-scoped
- Multiple instances of same PolicyType permitted subject to applicability rules
- Lookup by `policyInstanceId`; listing by `policyTypeId` returns all instances for current tenant

**PolicyConfiguration Registry:**
- Tenant-scoped
- A PolicyInstance may have many PolicyConfigurations in history; at most one ACTIVE
- Lookup by `policyConfigurationId`; listing by `policyInstanceId` returns configuration history

### 13.2 Registration requirements owned by the Framework

The Framework specifies the rules for what can be registered, when, and under what conditions (see PolicyType, PolicyInstance, PolicyConfiguration sections above).

### 13.3 What the Framework does NOT own

- Physical storage backend (Runtime concern)
- Indexing strategy (Runtime concern)
- API implementation (Runtime concern)
- Query interfaces (Runtime concern)
- Backup and recovery (Runtime concern)

---

## 14. Shared Reference Data

### 14.1 Reference data mechanism owned by the Framework

The Framework provides a `Shared Reference Data` mechanism for policies to reference shared vocabularies (Sensitivity Classifications, Risk Classes, etc.) without each policy duplicating definitions.

```yaml
refDataId: <namespaced-id>
schemaRef: <schema-location>
data: <structured-data>
lifecycle:
  state: ACTIVE
  version: <semver>
```

PolicyTypes reference shared vocabularies via `refDataId`. The Framework ensures existence and versioning; it does not interpret the data.

### 14.2 What the Framework does NOT own

- Specific reference data vocabularies (PII fields, risk classes, etc. — these are domain/distribution concerns)
- Storage of reference data (Runtime concern)
- Management UI for reference data (Runtime concern)

---

## 15. Domain Pack Policy Requirements (Dependency Inversion Contract)

### 15.1 Contract owned by the Framework

Domain packs declare `requiredPolicies` referencing PolicyTypes. The Framework owns the **enforcement contract** for these requirements:

```yaml
requiredPolicies:
  - policyTypeId: <namespaced-id>
    minVersion: <semver>
    requiredApplicability:
      domainPackIds: []
      environments: []
```

### 15.2 Enforcement rule owned by the Framework

At domain pack load time, for each `requiredPolicies` entry, the runtime verifies:
- Tenant has at least one `ACTIVE` PolicyInstance of that PolicyType
- Instance is at or above `minVersion`
- Instance has a referenced `ACTIVE` PolicyConfiguration
- Instance applicability matches `requiredApplicability` (when declared)

Otherwise, the pack refuses to load.

### 15.3 Applicability matching rule owned by the Framework

When `requiredApplicability` is declared, the satisfying PolicyInstance's `applicability` must be at least as constrained as the requirement. A random ACTIVE instance with global applicability does NOT satisfy a requirement with specific applicability scope.

### 15.4 What the Framework does NOT own

- The domain pack mechanism itself (Runtime owns: pack definition, loading, agent registration)
- Specific domain packs (deployment concern)
- Domain pack lifecycle (Runtime owns)

---

## 16. Audit Infrastructure

### 16.1 Audit contract owned by the Framework

- Every hook firing produces an audit event
- Audit is framework-injected, not policy-optional
- Each audit record carries `auditRecordId` as idempotency key
- Audit events written to durable sink **before** listener notification
- Listeners receive a copy; they are not the durability mechanism

### 16.2 Audit failure behavior owned by the Framework

**For `pre-` hooks:**
- If audit cannot be opened or persisted, operation **fails closed** (DENY)
- Emit `audit.persistenceFailed` via safeguard channel

**For `post-` and observational hooks:**
- Audit failure recorded through runtime safeguard channel
- Remediation may include run quarantine depending on profile

### 16.3 Audit flow owned by the Framework

1. Hook entry: open audit event with `auditRecordId`, `runId`, `tenantId`, `hookId`, `AgentIdentity`, timestamp, context fingerprint
2. Policy evaluation: record each policy's decision and evidence
3. Composition: record final composed decision
4. Hook exit: close audit event; durably persist
5. Listener notification: emit `audit.recorded` event

### 16.4 Safeguard channel contract owned by the Framework

When primary audit sink is unavailable, `audit.persistenceFailed` and `framework.safeguardTriggered` are routed through a separate runtime safeguard channel:

- Safeguard emissions are best-effort until audit durability restored
- Loss of safeguard-channel events in regulated profiles is a fatal runtime health condition
- Loss in non-regulated profiles is recorded for post-hoc analysis

### 16.5 Tenant isolation invariant owned by the Framework

All runtime data — registry entries, hook evaluations, events, audit records, conformance metrics — strictly segregated by `tenantId`. Cross-tenant leakage prohibited at framework level.

### 16.6 What the Framework does NOT own

- Audit storage backend (Runtime concern)
- Retention policy (Runtime concern)
- Query interface (Runtime concern)
- Dashboards and visualization (Runtime concern)
- Operational recovery procedures (Runtime concern)

---

## 17. AgentIdentity (Boundary Contract Field)

### 17.1 Schema owned by the Framework

```yaml
AgentIdentity:
  agentId: <stable-id-of-the-agent>
  identityKind: SERVICE_ACCOUNT | DELEGATED | JIT
  delegatedFrom: <originating-principal-id>
  credentialRef: <credential-handle>
  credentialExpiry: <timestamp>
  scope: [<permission>]
  attestationRef: <signed-attestation-ref>
```

### 17.2 Framework-bound invariant owned by the Framework

- `AgentIdentity` is framework-bound, not agent-supplied
- Runtime binds identity from deployment metadata, credential context, or upstream authenticated source **before** any policy evaluates
- Agent code may not declare, modify, or override `AgentIdentity`
- Attempts to inject identity fields are rejected at boundary

### 17.3 Framework guarantees

- `AgentIdentity` present on every artifact crossing the boundary
- `AgentIdentity` present on every hook context
- `AgentIdentity` bound by framework, not agent

### 17.4 Framework non-guarantees

- Cryptographic verification of `attestationRef` (identity provider responsibility)
- Validity or revocation status of `credentialRef` (identity provider responsibility)
- Interpretation of `scope` permissions (identity provider responsibility)

### 17.5 Anonymous identity restriction owned by the Framework

- LOCAL and explicitly non-regulated profiles: anonymous identity (`agentId: "anonymous"`, `identityKind: SERVICE_ACCOUNT`) may be supplied; recorded in audit
- Production and regulated profiles: missing identity **fails closed** with `ojas.framework.identity-required`
- Anonymous identity in regulated profiles permitted only when explicitly enabled in deployment policy

### 17.6 What the Framework does NOT own

- Identity provider implementation
- IAM integration mechanics (Okta, Auth0, Entra ID, AWS IAM — Runtime concern)
- Credential management
- Attestation cryptographic verification
- Service account lifecycle (Runtime concern)

---

## 18. Termination Primitive

### 18.1 Termination authorities owned by the Framework

Termination may be invoked by five authorities:

1. **Ops action** — explicit terminate command from authorized operator
2. **Policy request** — policy returning DENY with `REQUEST_TERMINATE_RUN` in `denyEffects`; framework decides whether to honor
3. **Conformance failure** — autonomous transition on conformance threshold violation
4. **Budget exhaustion** — runtime or economic budget exceeded
5. **Framework safeguard** — invariant violation (tenant isolation breach, audit sink failure on pre-hook)

### 18.2 Termination semantics owned by the Framework

- Termination does NOT require agent cooperation
- Framework directly stops the run: releases locks, preserves audit state, cleans sandbox
- Run-level termination emits `run.terminated`
- If agent state also affected, `agent.suspended` or `agent.disabled` emitted with same `auditRecordId`
- Termination is **idempotent** on `runId`

### 18.3 Policy termination request semantics owned by the Framework

Policies may *request* termination via `denyEffects`. Framework decides whether to honor based on:
- Policy's namespace tier (`ojas.core.*` typically honored; `vendor.*` may require additional authority)
- Operator policy
- Current run state

**Invariant:** termination is a framework primitive, not a policy side effect.

### 18.4 Termination audit and safeguard emission owned by the Framework

Every termination emits an audit record and `framework.safeguardTriggered` event tagged with the invoking authority.

### 18.5 What the Framework does NOT own

- Termination UI (Runtime concern)
- Operator console (Runtime concern)
- CLI tooling for termination (Runtime concern)
- Termination workflow approval (Runtime concern)

### 18.6 Terminology: "termination primitive" vs "kill switch"

**Canonical specification term:** `termination primitive`. The Framework uses this term throughout for precision — termination is a *primitive* (a foundational mechanism), and the five authorities (ops, policy, conformance, budget, safeguard) are *invocations* of that primitive.

**Operator-facing alias:** `kill switch` / `emergency stop`. Runtime distributions and public-facing documentation may use "kill switch" or "emergency stop" as accessible aliases when describing operator UI, dashboards, CLI commands, or marketing material. These are not different mechanisms — they are the same `termination primitive` rendered in operator vocabulary.

Specification language MUST use `termination primitive`. Operator-facing surfaces MAY use the alias.

---

## 19. Runtime Conformance Monitoring

### 19.1 Monitored signals owned by the Framework

| Signal | What it measures |
|---|---|
| Policy evaluation timeout rate | Policy exceeded declared time budget |
| Policy evaluation error rate | Policy threw exceptions |
| Malformed decision rate | Policy returned decision outside `decisionFamily` allowed returns |
| Version drift | Implementation digest, schema digest, or PolicyConfiguration digest mismatch |
| Hook firing failure | Runtime cannot fire a hook |
| Listener delivery failure rate | Events not reaching listeners |
| Composition conflict rate | Multiple policies disagreeing at same hook |

### 19.2 Version drift definition owned by the Framework

Drift means the **loaded artifact** at runtime no longer matches the **registered digest** for the active version. Three classes:
- Implementation binary digest mismatch
- Schema digest mismatch
- PolicyConfiguration `configurationDigest` mismatch

**Ownership split:** The Framework owns the **definition** of what drift is and the **rule** that drift detection triggers `ACTIVE → DISABLED`. The Runtime owns the actual digest computation, the comparison against stored registry digests, and the implementation of the mismatch-detection mechanism. The Framework specifies what counts as drift; the Runtime detects it.

### 19.3 Autonomous lifecycle transitions owned by the Framework

- `ACTIVE → DISABLED` — framework hard limit exceeded OR version drift detected
- No autonomous `ACTIVE → DEPRECATED` (DEPRECATED reserved for supersession)
- No autonomous `ACTIVE → SIMULATION` (revalidation through DISABLED → ACTIVE after manual review)

### 19.4 Tier-configurable malformed-decision thresholds owned by the Framework

| Namespace | Default | Configurable? |
|---|---|---|
| `ojas.core.*` | Zero-tolerance: first malformed → DISABLED | No (framework-frozen) |
| `ojas.stdlib.*` | First malformed → DISABLED | Yes, per PolicyType in `failureMode` |
| `vendor.*` | First malformed → DISABLED | Yes, no framework-imposed minimum |

### 19.5 What the Framework does NOT own

- Metric collection infrastructure (Runtime concern)
- Monitoring dashboards (Runtime concern)
- Alerting integration (Runtime concern)
- Signal aggregation backends (Runtime concern)

---

## 20. Conformance Profiles

### 20.1 Profile-sensitive rules owned by the Framework

The Framework defines rules that vary by profile, but does NOT define what each profile means concretely. Profile-sensitive rules include:

| Framework rule | Profile behavior |
|---|---|
| Anonymous AgentIdentity | LOCAL/non-regulated: permitted; production/regulated: fails closed |
| Applicability collision | Regulated: explicit applicability or overlap opt-in required |
| Patch conflict at same priority | Regulated: DENY; non-regulated: deployment-configurable |
| Transformed-path conflict | Regulated: DENY; non-regulated: deployment-configurable (DENY/SKIP_LATER/APPLY_AGAINST_ORIGINAL) |
| Mid-run configuration refresh | Regulated: rejected at deployment; non-regulated: opt-in |
| Safeguard-channel loss | Regulated: fatal health condition; non-regulated: recorded for analysis |
| Delivery guarantee for audit/conformance | All profiles: AT_LEAST_ONCE mandatory |

### 20.2 Profile vocabulary owned by the Framework

The Framework references profiles by name: `LOCAL`, non-regulated, production, regulated. These are abstract categories defined for rule-applicability purposes. The Framework does **not** define how a deployment maps its environment, SKU, and domain pack to one of these abstract profiles — that is a Runtime responsibility (see Runtime ownership §20).

### 20.3 What the Framework does NOT own

- Profile classification (Runtime owns: mapping environment × SKU × domain pack → profile)
- Profile names beyond the abstract categories above
- The decision of which concrete deployment is "regulated"

---

## 21. OWASP Mapping Metadata

### 21.1 Metadata contract owned by the Framework

PolicyTypes may declare OWASP ASI coverage via `owaspMapping` in their schema:

```yaml
owaspMapping:
  - category: ASI01..ASI10
    coverage: PRIMARY | SECONDARY
```

The Framework stores the metadata and exposes it via the registry API; it does **not** enforce coverage.

### 21.2 What the Framework does NOT own

- Specific OWASP coverage claims (policy-specific)
- Coverage testing (out of framework scope)
- OWASP compliance certification (external concern)

---

## 22. Conformance Criteria (Framework Compliance)

### 22.1 Required capabilities owned by the Framework

The Framework defines 38 required capabilities a Runtime must provide to claim Framework conformance, organized into:

- Registry and lifecycle (6 requirements)
- Hook evaluation (6 requirements)
- Applicability and collision (2 requirements)
- Events and listeners (5 requirements)
- Audit infrastructure (3 requirements)
- Identity (2 requirements)
- Termination primitive (3 requirements)
- Administration (2 requirements)
- Conformance monitoring (4 requirements)
- Tenant isolation (1 requirement)
- Failure modes (1 requirement)
- Deterministic patch composition (3 requirements)

See `OJAS_POLICY_FRAMEWORK_SPEC_v0.3.2c.md` §17.1 for the full list.

### 22.2 Behavioral assertions owned by the Framework

The Framework publishes a behavioral assertion suite (~25 representative assertions) executable against any implementation. Distributions may extend the suite but may not weaken framework-mandated assertions.

### 22.3 What the Framework does NOT own

- A reference implementation (Runtime concern)
- Implementation-specific assertions beyond framework requirements (Runtime concern)
- Integration test suites (Runtime concern)

---

## 23. Administrative Operations

### 23.1 Administrative verbs owned by the Framework

The Framework defines administrative operations as auditable artifacts. Verbs include (but are not limited to):

- `register_policy`
- `register_configuration`
- `transition_state`
- `bind_listener`
- `unbind_listener`
- `update_reference_data`
- `quarantine_run`
- `terminate_run`

### 23.2 Audit requirement owned by the Framework

Every administrative operation produces an immutable audit record with the same durability guarantees as policy-evaluation audit (§16).

### 23.3 Authorization separation owned by the Framework

The Framework mandates that Runtimes expose RBAC for administrative operations distinct from policy-evaluation RBAC.

### 23.4 What the Framework does NOT own

- Which roles exist (Runtime concern)
- How roles bind to enterprise IAM (Runtime concern)
- Wire format of Admin API (Runtime concern)
- Admin console UX (Runtime concern)
- CLI command structure (Runtime concern)
- GitOps integration (Runtime concern)

---

## 24. Distribution

### 24.1 Framework distribution package contents

| Artifact | Description |
|---|---|
| Framework specification | The versioned spec document (`OJAS_POLICY_FRAMEWORK_SPEC_vX.Y.Z.md`) |
| Conformance test suite | Behavioral assertions executable against any Runtime |
| OWASP mapping metadata schema | For policy authors |
| Glossary | Canonical terminology |
| Version history | Change log across all revisions |

### 24.2 Distribution licensing owned by the Framework maintainer

The Framework specification's license and openness model are owned by the Framework specification maintainer / SPHUTA distribution maintainer.

### 24.3 What the Framework distribution does NOT include

- Reference implementation (separate Runtime distribution)
- Standard-library policies (separate distribution)
- Admin tooling (Runtime concern)
- Agent framework integrations (Runtime concern)
- Domain pack examples (Runtime concern)

---

## 25. Forward References (Deferred Concerns)

### 25.1 Concerns the Framework explicitly defers

| Concern | Deferred to |
|---|---|
| Inter-agent communication governance (ASI07) | v0.4 of framework OR `OJAS_INTER_AGENT_GOVERNANCE_SPEC.md` |
| Concrete IAM provider bindings | `OJAS_IDENTITY_BINDING_SPEC.md` |
| Admin console, CLI, API surface | `OJAS_RUNTIME_ADMIN_SPEC.md` |
| Agent framework integration mechanics | Per-framework integration specs |

### 25.2 What the Framework owns about deferred concerns

The Framework explicitly defines:
- That each concern is recognized as relevant
- That each is deliberately not designed in the current Framework version
- Where each is addressed (companion document or future Framework version)
- Why deferral is appropriate (e.g., ASI07 is a substantive design problem requiring dedicated work)

### 25.3 Forward-reference invariant owned by the Framework

Concerns deferred from the Framework MUST be addressed by referenced companion documents before claiming production readiness for the affected use cases.

---

## 26. What the Framework Does NOT Provide (Explicit Exclusions)

### 26.1 Explicit exclusions

The Framework deliberately does NOT provide:

- Specific policy semantics
- Business definitions of "denied" beyond runtime DENY decision
- Particular evidence formats within policies
- Opinion on which policies should exist
- Audit storage backend implementation
- Policy authoring tooling
- Identity provider implementation
- Cryptographic verification of attestations
- Termination UI or operator tooling
- Permission interpretation for `AgentIdentity.scope`
- Network protocols for events or audit records
- Exhaustive hook catalog beyond core minimum
- `EXACTLY_ONCE` event delivery
- Patch semantic-merge resolution beyond same-priority and transformed-path rules
- LLM-specific prompt templates or model version resolution

### 26.2 Rationale

Each exclusion preserves the Framework's policy-agnostic, runtime-agnostic property at a specific surface. Adding any of these would couple the Framework to implementation choices and reduce conformant implementation flexibility.

---

## 27. Framework Maintainer Responsibilities

### 27.1 The Framework maintainer

The Ojas Framework specification is maintained by the **Ojas specification maintainer**. The SPHUTA distribution operates as the reference distribution maintainer.

### 27.2 Maintainer responsibilities

- Author and revise the Framework specification
- Manage the assessment → review → resolution discipline
- Publish the conformance test suite
- Maintain the version history
- Coordinate breaking-change ADRs

### 27.3 What the maintainer does NOT decide

- Specific Runtime implementations
- Standard-library policy contents (separate maintainer authority)
- Deployment configurations
- Enterprise adoption choices

---

## Appendix A — Cross-references to Framework Spec

This ownership document maps to the canonical Framework specification (`OJAS_POLICY_FRAMEWORK_SPEC_v0.3.2c.md`) as follows:

| Ownership section | Framework spec section |
|---|---|
| §2 Core Concepts | Spec §1 |
| §3 Namespace and Registrar Tiers | Spec §2 |
| §4 PolicyType Contract | Spec §3.1, §3.1.1, §3.5, §3.6, §3.7 |
| §5 PolicyInstance Contract | Spec §3.2, §3.2.1 |
| §6 PolicyConfiguration Contract | Spec §3.3, §3.3.1, §3.3.2 |
| §7 Hook Registry | Spec §5 |
| §8 Hook Evaluation and Composition | Spec §7 |
| §9 Configuration Resolution | Spec §7.2.1 |
| §10 Lifecycle State Machine | Spec §6 |
| §11 Events | Spec §8 |
| §12 Listeners | Spec §9 |
| §13 Registries (Contract Layer) | Spec §4 |
| §14 Shared Reference Data | Spec §10 |
| §15 Domain Pack Policy Requirements | Spec §11 |
| §16 Audit Infrastructure | Spec §12 |
| §17 AgentIdentity | Spec §13 |
| §18 Termination Primitive | Spec §14 |
| §19 Runtime Conformance Monitoring | Spec §15 |
| §20 Conformance Profiles | Spec various (profile-sensitive rules) |
| §21 OWASP Mapping Metadata | Spec §16 |
| §22 Conformance Criteria | Spec §17 |
| §23 Administrative Operations | Spec §14.5 |
| §25 Forward References | Spec §18.2 |
| §26 Explicit Exclusions | Spec §18.1 |

---

## Appendix B — Glossary

| Term | Definition |
|---|---|
| **Framework** | The Ojas Policy Framework — the policy execution mechanism specification |
| **Runtime** | An implementation of the Framework that hosts agents and enforces policies |
| **Conformant Runtime** | A Runtime that passes the Framework's conformance assertion suite |
| **Distribution** | A packaged release of a Framework, Runtime, or both with associated tooling |
| **Deployment** | A concrete production instance of a Runtime serving a tenant |
| **Tenant** | An isolated namespace within a Runtime (typically per-organization or per-environment) |
| **Hook** | A synchronous decision point in runtime execution |
| **Event** | An asynchronous past-tense notification |
| **Listener** | An observational subscriber to events |
| **Policy** | A registered governance rule (PolicyType + PolicyInstance + PolicyConfiguration) |
| **Run** | A single invocation of an agent identified by `runId` |
| **Boundary** | The contract surface between Ojas and external authorities |

---

*End of `OJAS_FRAMEWORK_OWNERSHIP.md` — final.*
