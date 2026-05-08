# OJAS Test and Conformance

**Status:** Draft implementation spec  
**Scope:** pytest-based conformance testing for Ojas modules and SPIs.

---

## 1. Purpose

Conformance tests make certification concrete.

A module is not certified because it claims compliance. It must pass the applicable Ojas conformance test suites.

---

## 2. Test package baseline

```text
ojas-test
ojas-test-runtime-conformance
ojas-test-memory-conformance
ojas-test-tool-conformance
ojas-test-review-conformance
ojas-test-evidence-conformance
```

Control-plane conformance may be added under:

```text
ojas-test-control-plane-conformance
```

or split later by sub-service if needed.

---

## 3. Pytest model

Ojas uses pytest as the conformance test runner.

A backend package provides fixtures.

Ojas conformance packages provide tests.

Example:

```python
import pytest
from ojas_backend_langgraph import LangGraphRuntimeBackend

@pytest.fixture
def runtime_backend():
    return LangGraphRuntimeBackend(test_mode=True)
```

The conformance suite imports tests that run against this fixture.

---

## 4. Conformance test principles

1. Every SPI method maps to at least one conformance test.
2. Every governance requirement maps to one or more tests.
3. Passing tests is required for certification candidate status.
4. Test suite changes may trigger recertification.
5. Tests must cover success and failure cases.
6. Tests must assert ledger events where required.
7. Tests must simulate bypass attempts where possible.

---

## 5. Runtime conformance examples

Required tests:

- backend initializes from valid run context
- backend rejects invalid context package
- all tool calls route through Ojas request method
- lifecycle state matches Ojas state
- close session prevents late operations
- unsupported feature returns typed unsupported result
- governance coverage descriptor present

---

## 6. Memory conformance examples

Required tests:

- candidate query returns authority labels
- candidate query preserves scope metadata
- write candidate does not create authority
- admission commit is required for durability/authority
- revocation is stored and queryable
- revoked memory does not appear as active candidate

---

## 7. Tool conformance examples

Required tests:

- argument schema validation before side effect
- destructive tool declares destructive effect
- expected business failure returns ToolResult
- unexpected contract violation raises typed exception
- output carries source trust
- no constructor-time side effects

---

## 8. Review conformance examples

Required tests:

- submit returns review token
- decision includes reviewer identity
- decision includes reviewer scope
- missing decision does not imply approval
- decision evidence retrievable
- cancellation recorded

---

## 9. Evidence conformance examples

Required tests:

- store returns reference
- retrieve returns same content
- append-only behavior preserved
- link creates auditable relationship
- export pack preserves evidence refs

---

## 10. Control-plane conformance examples

Ledger:

- append-only event ordering
- run closure prevents late events
- integrity verification result

Registry:

- descriptor claim not self-authorizing
- revoked module blocked
- variant eligibility enforced

Policy:

- allow/deny decisions include policy version
- deny is explicit
- policy output ledgerable

Secrets:

- secret value not ledgered
- secret reference ledgered
- scope-bound access enforced

---

## 11. Certification relationship

Conformance result states:

```text
not_tested
failed
passed
passed_with_warnings
expired
```

Certification may require:

- passed conformance tests
- package audit
- descriptor validation
- registry trust
- vulnerability review
- variant-specific review

Conformance is necessary but not always sufficient for Regulated certification.

---

## 12. Conformance versioning

Open item for future micro-spec:

- how long certified modules have to re-pass after test-suite update
- which test-suite changes are breaking
- whether failed recertification causes immediate suspension or grace period
- how emergency security tests are applied

---

## 13. Invariants

1. Certification requires test evidence.
2. SPI claims must be testable.
3. Bypass risks must be tested where possible.
4. Test failures block certification.
5. Conformance tests are versioned.
6. Passing Open tests does not imply Regulated certification.
