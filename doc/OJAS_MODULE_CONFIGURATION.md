# OJAS Module Configuration

**Status:** Draft implementation spec  
**Scope:** Configuration model, configuration scope precedence, validation, and secrets.

---

## 1. Purpose

This document defines how Ojas modules receive operational configuration.

Configuration supplies runtime values.

Configuration does not activate or authorize modules.

---

## 2. Core distinction

| Surface | Meaning |
|---|---|
| Manifest | selects and governs |
| Descriptor | declares module identity/capability |
| Registry | verifies trust/certification |
| Configuration | supplies runtime values |
| Secret backend | resolves sensitive values |

Rule:

> Configuration cannot create authority.

---

## 3. Configuration scopes

Ojas supports four configuration scopes.

```text
module-default
  ↓
tenant-default
  ↓
environment
  ↓
run-instance
```

Higher layers override lower layers only for fields permitted by the module config schema.

| Scope | Meaning |
|---|---|
| Module default | safe defaults shipped with module |
| Tenant default | tenant-wide operational settings |
| Environment | dev/qa/staging/prod overlay |
| Run instance | rare per-run override |

---

## 4. Resolution order

```text
merged_config =
  module_default
  overridden by tenant_default
  overridden by environment
  overridden by allowed run_instance overrides
```

Validation happens after merge.

Invalid configuration fails activation.

---

## 5. Pydantic settings model

Each module should define a typed configuration model.

Example:

```python
from pydantic import Field, SecretStr
from pydantic_settings import BaseSettings

class ZepMemoryConfig(BaseSettings):
    api_url: str = Field(description="Zep API endpoint")
    api_key_ref: str = Field(description="Secret reference for Zep API key")
    namespace: str = Field(default="ojas-default")
    timeout_seconds: int = Field(default=30, ge=1, le=300)
    retry_attempts: int = Field(default=3, ge=0, le=10)
```

Raw secrets should not be stored in config.

Use secret references.

---

## 6. Configuration schema

Each module descriptor references configuration schema.

```yaml
configSchema:
  ref: "./config-schema.json"
  secretFields:
    - apiKeyRef
```

Rules:

- schema must be generated or validated from Pydantic model
- required fields must be explicit
- secret fields must not contain raw values
- run-instance overrides must be limited

---

## 7. Secrets handling

Secrets are resolved through control-plane secrets service.

Rules:

- secret values never appear in ledger
- secret references may be ledgered
- secret access is scope-bound
- missing required secret fails activation
- secret rotation must be supported where required

Example:

```yaml
modules:
  ojas-memory-zep:
    apiUrl: "https://api.zep.example"
    apiKeyRef: "secret://tenant-a/zep-api-key"
```

---

## 8. Manifest vs configuration

The manifest may select:

```yaml
memory:
  backend: zep
```

Configuration may provide:

```yaml
modules:
  ojas-memory-zep:
    namespace: tenant-a-prod
    timeoutSeconds: 30
```

Configuration may not change:

- selected backend
- variant
- allowed tools
- memory authority rules
- certification status
- trust decision

---

## 9. Validation failures

Activation fails if:

- required config missing
- config type invalid
- secret reference invalid
- unsupported override used
- config violates variant policy
- config references uncertified endpoint where certification required

---

## 10. Ledger events

Configuration resolution should emit:

| Event | Meaning |
|---|---|
| `MODULE_CONFIG_RESOLUTION_STARTED` | config merge began |
| `MODULE_CONFIG_VALIDATED` | config accepted |
| `MODULE_CONFIG_INVALID` | validation failed |
| `MODULE_SECRET_REF_RESOLVED` | secret reference resolved, not value |
| `MODULE_CONFIG_ACTIVATION_BLOCKED` | config failure blocked activation |

---

## 11. Configuration invariants

1. Configuration does not activate modules.
2. Configuration does not authorize behavior.
3. Manifest governs; config supplies values.
4. Secrets are references, not raw values.
5. Invalid required config fails activation.
6. Run-instance overrides are exceptional and restricted.
