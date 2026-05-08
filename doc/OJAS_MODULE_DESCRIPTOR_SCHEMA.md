# OJAS Module Descriptor Schema

**Status:** Draft implementation spec  
**Scope:** `ojas-module-descriptor.yaml` structure and validation requirements.

---

## 1. Purpose

Every Ojas module must ship a descriptor.

The descriptor is a machine-readable declaration of module identity, SPI implementation, package compatibility, governance coverage, risks, certification, and signatures.

---

## 2. Canonical filename

```text
ojas-module-descriptor.yaml
```

The descriptor must be discoverable through package/distribution metadata without importing implementation code.

---

## 3. Descriptor is not self-authorizing

A descriptor is a claim.

It becomes trusted only after registry verification.

Rule:

> Module descriptor claims do not create trust or certification by themselves.

---

## 4. Required top-level fields

```yaml
descriptorVersion: "1.0"

module:
  id: ojas-backend-langgraph
  type: runtime_backend
  publisher: pommala
  moduleVersion: 1.0.0
  ojasCoreVersionRange: ">=1.0,<2.0"
  ojasSpiVersionRange: ">=1.0,<2.0"
  spisImplemented:
    - ojas-spi-runtime
```

---

## 5. Supported package declaration

```yaml
supportedBackendPackages:
  - name: langgraph
    versionRange: ">=0.2,<0.3"
  - name: langgraph-checkpoint
    versionRange: ">=2.0,<3.0"
```

Rules:

- package names must be exact distribution names
- version ranges must be explicit
- unconstrained versions are not allowed
- package drift outside range blocks activation

---

## 6. Supported variants

```yaml
supportedVariants:
  open: allowed
  standard: allowed
  regulated: certification_required
```

Allowed values:

```text
allowed
certification_required
not_allowed
```

Rule:

> Variant support is declared by the module but decided by registry/certification.

---

## 7. Governance coverage

```yaml
governanceCoverage:
  scope: full
  toolBroker: full_if_wrapped
  memoryAdmission: partial
  auditTrace: strong
  rollback: partial
  drift: partial
  concurrency: partial
  budget: partial
  regression: strong
  contract: strong
  severity: ojas_owned
  evidence: partial
  expiry: partial
```

Coverage vocabulary should be controlled.

Suggested values:

```text
full
strong
partial
restricted
none
not_applicable
ojas_owned
full_if_wrapped
```

---

## 8. Bypass risks

```yaml
bypassRisks:
  - id: BR-001
    description: direct tool calls if native framework tool layer is not wrapped
    severity: high
    mitigation: require Ojas Tool Broker wrapper
```

Rules:

- empty list means explicit claim of no known bypasses
- bypass risk claims are auditable
- unknown bypass risk may block certification

---

## 9. Certification

```yaml
certification:
  status: candidate
  certifiedAt: null
  expiresAt: null
  certifyingAuthority: null
  certificationEvidenceRef: null
```

Allowed statuses:

```text
candidate
certified
revoked
expired
rejected
```

---

## 10. Configuration schema reference

```yaml
configSchema:
  ref: "./config-schema.json"
  secretFields:
    - apiKeyRef
```

Rules:

- config schema must be valid JSON Schema or Pydantic-exported schema
- secret fields must reference secret refs, not raw secret values
- configuration does not activate or authorize modules

---

## 11. Signatures

```yaml
signatures:
  publisherSignature: "<base64-or-ref>"
  registrySignature: null
```

Rules:

- publisher signature asserts package publisher identity
- registry signature asserts trust/certification record
- signature format is deferred to module security spec
- unsigned modules are not Regulated-eligible

---

## 12. Descriptor validation rules

Activation must fail if:

- descriptor missing
- descriptor invalid
- required field missing
- descriptorVersion unsupported
- module ID mismatch
- package version out of range
- variant not allowed
- certification expired/revoked
- registry verification fails
- signature invalid where required

---

## 13. Minimal descriptor example

```yaml
descriptorVersion: "1.0"

module:
  id: ojas-memory-zep
  type: memory_backend
  publisher: pommala
  moduleVersion: "1.0.0"
  ojasCoreVersionRange: ">=1.0,<2.0"
  ojasSpiVersionRange: ">=1.0,<2.0"
  spisImplemented:
    - ojas-spi-memory

supportedBackendPackages:
  - name: zep-cloud
    versionRange: ">=2.0,<3.0"

supportedVariants:
  open: allowed
  standard: allowed
  regulated: certification_required

governanceCoverage:
  scope: strong
  memoryAdmission: full_if_wrapped
  auditTrace: partial
  revocation: strong
  expiry: strong

bypassRisks:
  - id: BR-001
    description: backend context block must not bypass Ojas context assembly
    severity: high
    mitigation: use Ojas Context Assembler only

certification:
  status: candidate
  certifiedAt: null
  expiresAt: null
  certifyingAuthority: null
  certificationEvidenceRef: null

configSchema:
  ref: "./config-schema.json"
  secretFields:
    - apiKeyRef

signatures:
  publisherSignature: "<publisher-signature>"
  registrySignature: null
```
