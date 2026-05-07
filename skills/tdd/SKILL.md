---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests verify intent, not implementation. The question is not "does the code do what the code does?" — that is tautologically true and catches nothing. The question is "does the system do what the business needs?"

**Good tests** exercise real code paths through public interfaces, describe capability ("user can checkout with valid cart"), and survive internal refactors. If renaming an internal function breaks a test, that test was checking implementation.

**The coverage trap**: 90% coverage built of AI-generated tests is often 90% "works as implemented" and 0% "works as designed." Coverage is a lagging signal, not a quality signal.

**The AI entropy problem**: AI writes code → AI writes tests → two divergent layers, zero convergence. The AI tests validate the AI code's behavior, not your intent. You now have two layers of "works as implemented" and zero layers of "works as designed."

**The fix — human-AI split**: You write test names and intent. AI fills setup boilerplate. The name is the spec: `test_user_creation_fails_when_email_already_exists_and_returns_409`. That sentence is a requirement. An AI-generated name like `test_user_creation` is a description of the code. You own the convergence step; AI owns the typing.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

---

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** Tests written in bulk check imagined behavior. Tests written vertically respond to what you just learned.

```
WRONG: RED: test1,2,3,4,5  →  GREEN: impl1,2,3,4,5
RIGHT: RED→GREEN: test1→impl1,  test2→impl2,  test3→impl3 ...
```

---

## Workflow

### 0. Declare and Confirm (always first)

Before writing any test, declare what you are testing and why — at the capability level, not the implementation level. Get the user to confirm:

> "I'm going to write tests for **[capability or contract being verified]**.
> Key properties being asserted: [2–3 bullets].
> Does this match what you expect?"

Do not proceed without confirmation. This is the human convergence step. Skip it and you risk testing what was built, not what was intended.

### 1. Plan

Use the project's domain glossary for test names. Identify which behaviors matter most — you cannot test everything. Write test names yourself before touching implementation. Confirm the list with the user.

Consult [deep-modules.md](deep-modules.md) and [interface-design.md](interface-design.md) before finalizing interfaces.

### 2. Red → Green Loop

```
RED:   Write one test for one behavior → fails
GREEN: Write minimal code to pass → passes
Repeat for each remaining behavior.
```

One test at a time. Only enough code to pass the current test. No speculative features.

### 3. Refactor

After all green: extract duplication, deepen modules, apply SOLID where natural. Run tests after each refactor step. **Never refactor while RED.**

See [refactoring.md](refactoring.md).

---

## Beyond Pure Functions: Side Effects, Convergence, LLM Output

The loop above is ideal for pure functions. Three additional patterns apply when the system involves side effects, stage pipelines, or non-deterministic components (LLM calls, network I/O).

### Stage contract tests

When a function's contract IS its side effect (writes files, populates a temp dir, updates DB state), test the produced artifact — not whether the internal write calls were made.

```python
# Bad: tests the call chain
assert mock_github_client.get_file.call_count == 3

# Good: tests what the next stage actually receives
assert (temp_dir / "get_ticket" / "get_issues.py").exists()
assert len(list(temp_dir.rglob("*.py"))) == expected_file_count
```

Mocking the side effect away makes you test your mock, not your function.

### Convergence tests

When two code paths must produce equivalent intermediate state (e.g. ZIP upload and GitHub upload both routed through the same chunker), run both real paths and assert equivalence. Mocks defeat this test's purpose — both mocked paths will always "pass" regardless of whether the real paths actually converge.

```python
zip_chunks = parse_codebase(extract_zip_to(fixture_zip, tmp_path / "zip"))
gh_chunks  = parse_codebase(fixture_github_dir)   # same codebase, different delivery

assert {c.file_path for c in zip_chunks} == {c.file_path for c in gh_chunks}
assert len(zip_chunks) == len(gh_chunks)
```

### Property assertions for LLM and non-deterministic output

Never assert exact LLM output text. Assert semantic properties. Run N=3–5 real trials on a fixed input fixture — one passing trial proves nothing for a non-deterministic component.

```python
# Bad: one mocked call, exact output assertion
mock_llm.return_value = '{"auth_mechanism": "HMAC"}'
assert result == {"auth_mechanism": "HMAC"}

# Good: N real calls, property assertion on every run
for _ in range(5):
    result = extract_evidence(fixture_chunks)    # real LLM call, fixed input
    assert "auth_mechanism" in result            # property: key always present
    assert result["auth_mechanism"] is not None  # property: never null for this input
```

---

## Checklist Per Cycle

```
[ ] Declare + confirm step done before any test is written
[ ] Test name is human-written: it is the spec, not a description of the code
[ ] Test verifies intent (what is needed), not implementation (what the code does)
[ ] Test uses public interface only; would survive an internal refactor
[ ] For side-effect functions: artifact tested, not call chain mocked
[ ] For convergence points: both real paths exercised, no mock on the convergence
[ ] For LLM/non-deterministic output: N=3 property assertions, not 1 exact-output assertion
[ ] Code is minimal for this test; no speculative features added
