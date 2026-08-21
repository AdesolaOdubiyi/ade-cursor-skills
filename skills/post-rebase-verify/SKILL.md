---
name: post-rebase-verify
description: Run after any rebase that required manual conflict resolution. Checks
  that intentional additions survived --ours/--theirs resolution and that structured
  files (package manifests, configs, interfaces) are consistent. Catches the class
  of silent-drop failures that only surface when a reviewer asks "where's the TODO?"
  or a build fails. Trigger on "post-rebase", "after rebase", "verify rebase", "check
  rebase", or whenever you just resolved rebase conflicts.
allowed-tools: Bash Read Grep Glob
---

# Post-Rebase Verification

Run this immediately after any rebase that required manual conflict resolution,
before pushing or requesting review.

---

## Before you start: identify your intentional additions

List (mentally or literally) what you explicitly added in this branch that is
not auto-generated or boilerplate. Common examples:

- TODO / FIXME comments
- Named constants (replacing a magic number)
- Explanatory comment blocks
- Annotations or decorators you added
- Specific config values or environment variables
- Field-level description strings

Keep this list in mind — Steps 2 and 3 verify these survived.

---

## Step 1: Confirm the diff matches your mental model

```bash
git diff origin/main...HEAD --stat
# or: git diff origin/master...HEAD --stat
```

For every file you intentionally changed, confirm:
- It appears in the list
- Its line delta (+N/-M) is in the right ballpark

**Red flag:** A file you know you changed shows `+0/-0` or far fewer lines than
expected — your change was silently dropped during `--ours` or `--theirs`.
Stop here. Do not push until you restore it.

---

## Step 2: Grep for intentional additions

For each item from your pre-flight list, grep the relevant file:

```bash
grep -n "TODO\|your-constant-name\|specific-annotation" path/to/file
```

If anything is missing: restore it in a fixup commit before pushing.

---

## Step 3: Check interface/implementation parity

If your branch changes both a contract (interface, type definition, abstract class,
API schema) and its implementation, verify every method/field you added to the
contract has a matching implementation.

```bash
grep -c "YourNewMethod" path/to/contract-file
grep -n "YourNewMethod" path/to/impl-file
```

Also verify the reverse: no implementation is referencing a type or symbol that
got dropped from the contract during conflict resolution.

---

## Step 4: Check structured files for duplicates or missing entries

Any file with enforced structure — package manifests (`package.json`, `pyproject.toml`,
`pom.xml`, `go.mod`), lock files, deploy configs, dependency injection registries —
can silently accumulate duplicate entries or lose entries when both sides of a
conflict add to the same block.

```bash
# Check for duplicate dependency entries (replace with what you added)
grep -c "your-package-name" package.json
# Should be 1. If 2+, remove the duplicate.

# Confirm the entry exists at all
grep "your-package-name" package.json
```

For DI registries and similar files, visually scan the registration block to
confirm all entries you added are present.

---

## Step 5: Confirm no conflict markers remain

```bash
grep -rn "<<<<<<\|=======\|>>>>>>>" src/
```

Should return nothing. Any result means a conflict was staged but not resolved.

---

## Step 6: Final diff review

```bash
git diff origin/main...HEAD -- path/to/file/you/care/about
```

Skim the actual lines. Ask: does this show everything I intended to add, and
nothing I didn't? If no on either side, fix it before pushing.

---

## When to run

- After any `git rebase` where you resolved ≥1 conflict manually
- After any sequence of `git checkout --ours` or `--theirs` calls
- Before force-pushing a rebased branch
- Any time a reviewer asks about something you expected to be in the PR

## Time cost

2 minutes. The cost of skipping it is typically 30–60 minutes debugging a
build failure or re-explaining a code decision to a reviewer.
