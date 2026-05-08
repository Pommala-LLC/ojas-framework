# OJAS Module Security and Supply-Chain Trust

**Status:** Draft implementation spec  
**Scope:** Module discovery, trust, signing, registry approval, monkey-patching, isolation, and activation safety.

---

## 1. Purpose

This document defines implementation security rules for Ojas modules.

It is separate from core agent-governance semantics.

---

## 2. Master module security rule

> **Installed is not trusted. Trusted is not certified. Certified is not active. Active is not authority.**

---

## 3. Metadata-only discovery

Ojas discovers modules by metadata.

Rule:

> **Ojas MUST NOT call `entry_point.load()` during discovery.**

Discovery uses metadata such as:

- distribution metadata
- entry-point group and name
- descriptor file reference
- publisher identity where available
- package version
- package hash where available

Implementation code loads only after the activation gate passes.

---

## 4. Activation gate

Implementation code may load only after:

1. manifest selection
2. descriptor discovery
3. descriptor validation
4. registry trust verification
5. signature verification where required
6. variant eligibility check
7. certification check where required

---

## 5. Descriptor location

Canonical descriptor file:

```text
ojas-module-descriptor.yaml
```

The descriptor must be discoverable without importing the module implementation.

---

## 6. Registry trust

The registry records:

- module identity
- publisher identity
- descriptor hash
- package hash
- supported versions
- certification status
- variant eligibility
- revocation status
- audit history
- security advisory status

Descriptor claims are not trusted unless registry verifies them.

---

## 7. Signing

Descriptor signing format is deferred to a micro-spec.

Minimum expected concepts:

- publisher signature
- registry signature
- descriptor hash
- package hash
- certification evidence reference
- revocation marker

Unsigned modules are not Regulated-eligible.

---

## 8. No-monkey-patching pledge

Modules accepted into the Ojas registry must not:

- monkey-patch the Ojas kernel
- monkey-patch other Ojas modules
- modify protected framework surfaces
- alter governance services at runtime
- bypass Ojas tool/memory/context paths
- replace ledger behavior
- suppress governance events

Detected monkey-patching is a registry violation.

Possible responses:

- module suspension
- run suspension
- certification review
- registry review
- revocation
- emergency freeze

---

## 9. Process isolation

For Regulated deployments:

> Backend modules must run in process isolation from the Ojas kernel.

Open and Standard may allow in-process execution subject to policy.

The transport mechanism is deferred to a micro-spec.

Possible future transports:

- gRPC
- Unix sockets
- multiprocessing queues
- ZeroMQ
- local sidecar protocol

---

## 10. Import side effects

Python imports may execute code.

Therefore:

- avoid implementation imports during discovery
- descriptor reading must not require implementation import
- loading happens only after activation gate
- import-time side effects are prohibited for certified modules
- side effects detected at import/load may trigger certification failure

---

## 11. Module revocation

A revoked module must not activate.

Revocation triggers:

- security vulnerability
- conformance failure
- expired certification
- policy violation
- monkey-patching
- unsupported package drift
- malicious behavior
- registry decision

Revocation must be visible to activation and run startup.

---

## 12. Activation failure behavior

Activation fails if:

- descriptor missing
- descriptor invalid
- registry trust fails
- signature invalid where required
- variant eligibility missing
- certification required but absent
- module revoked
- package version out of supported range
- required config invalid
- required secret unavailable
- backend prerequisite missing

No silent fallback.

---

## 13. Partial activation policy

All manifest-required modules must activate successfully.

If a required module fails:

```text
deployment/run startup fails
```

Open may allow optional modules only if marked optional and not required for governance-critical behavior.

Non-optional even in Open:

- scope enforcement
- tool brokering where tools are used
- memory admission where memory is written
- ledger/audit where required by variant
- review path where review is required

---

## 14. Security events

| Event | Meaning |
|---|---|
| `MODULE_DISCOVERED` | metadata found |
| `MODULE_DESCRIPTOR_VALIDATED` | descriptor schema valid |
| `MODULE_TRUST_VERIFIED` | registry trust accepted |
| `MODULE_SIGNATURE_VERIFIED` | signature accepted |
| `MODULE_VARIANT_ELIGIBLE` | certification permits variant |
| `MODULE_LOAD_STARTED` | implementation loading began |
| `MODULE_ACTIVATED` | module active |
| `MODULE_ACTIVATION_FAILED` | activation failed |
| `MODULE_MONKEY_PATCH_DETECTED` | monkey-patching detected |
| `MODULE_REVOKED` | module revoked |
| `MODULE_DRIFT_DETECTED` | package/version drift detected |

---

## 15. Security invariants

1. Discovery is metadata-only.
2. Loading is gated.
3. Installation is not trust.
4. Descriptor is not self-authorizing.
5. Registry trust is required.
6. Certification is variant-specific.
7. Revocation blocks activation.
8. Regulated requires isolation.
9. Monkey-patching is prohibited.
10. No silent fallback.
