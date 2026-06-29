Test Failure Decision Policy

Automated tests are treated as executable specifications and must never be modified solely to make the test suite pass.

When a test run fails, determine the reason before making any changes.

Missing Tests

If implementation is complete but automated tests are missing or insufficient:

* Create comprehensive tests.
* Follow the project’s existing testing framework, conventions, and structure.
* Cover expected behaviour, edge cases, error handling, and regression scenarios.
* Do not reduce test coverage to simplify implementation.

Existing Test Failures

If an existing automated test fails:

1. Identify precisely why the failure occurred.
2. Determine whether the failure is caused by:
    * an implementation defect,
    * an intentional behavioural change,
    * an outdated test,
    * or an incorrect test.

The default assumption is that the test is correct.

Unexpected Behaviour Changes

If code changes alter behaviour such that existing tests fail, do not automatically update, delete, weaken, or rewrite the failing tests.

Instead:

* Stop implementation.
* Present the failing tests.
* Explain exactly what behaviour changed.
* Explain why the implementation now differs from the existing specification.
* Ask the user whether:
    * the behavioural change was intentional and the tests should be updated, or
    * the behavioural change is unintended and the implementation should be corrected.

Do not proceed until the user confirms the intended behaviour.

Permitted Test Updates

Tests may be updated without user confirmation only when:

* they are objectively incorrect,
* they contain typographical or syntactic errors,
* they are flaky due to nondeterministic behaviour,
* they rely on deprecated APIs that do not affect intended behaviour,
* or they fail because of unrelated infrastructure or environment issues.

Clearly explain why such changes are safe.

Prohibited Behaviour

Never:

* modify tests simply to obtain a passing build,
* remove assertions without justification,
* weaken assertions to accommodate incorrect behaviour,
* delete failing tests without explicit approval,
* reduce coverage to avoid failures,
* ignore failing tests.

Principle

Tests define the expected behaviour of the system. A failing existing test represents a potential contract violation until proven otherwise. Preserve that contract unless the user explicitly confirms that the specification itself has changed.