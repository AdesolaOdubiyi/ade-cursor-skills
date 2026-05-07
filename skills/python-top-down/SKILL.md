---
name: python-top-down
description: >-
  Enforce Python top-down module structure: public API and high-level logic
  first, private helpers and low-level implementation at the bottom.
  Use when writing, reviewing, or refactoring any Python module — especially
  when the user mentions top-down structure, module ordering, readability,
  or asks why a function is hard to find. Also use proactively when generating
  new Python modules with more than 3 functions.
---
# Python Top-Down Module Structure
A reader should be able to understand a module from top to bottom without jumping around.
High-level intent first. Implementation details at the bottom.
---
## The Rule
**Public API -> orchestrators -> private helpers -> low-level utilities**
A reader's first question is always: "What does this module DO?" Answer that immediately.
Never make them scroll past private helpers and regex patterns to find the public function.
---
## Canonical Ordering
```
module docstring
imports
logging
-----------------------------------------
PUBLIC TYPES <- enums, dataclasses, TypedDicts, exceptions
that callers import and use
-----------------------------------------
CONSTANTS & CONFIG <- named constants, signal patterns, registries,
lookup tables - supporting data for the public API
-----------------------------------------
PUBLIC API <- functions/classes callers actually use
highest-level first (entry points before helpers)
-----------------------------------------
PRIVATE IMPLEMENTATION <- _prefixed helpers, in dependency order
(caller before callee, top to bottom)
-----------------------------------------
LOW-LEVEL UTILITIES <- pure functions, regex wrappers, data parsers
that have no caller context of their own
```
---
## What "Top-Down" Actually Means
It does NOT mean: alphabetical, or longest-first, or "put constants at the bottom."
It means: **a reader can read the file like a book**. Each function they encounter
references only things they have already seen OR things below that they will read next.
```python
# WRONG - reader hits classify_document before knowing what _SECTION_TARGETS is
_SECTION_TARGETS = [...] # private data
_extract_sections(...) # private helper
_find_section(...) # private helper
classify_section_text(...) # public
classify_document(...) # public <- buried at the bottom
# RIGHT - reader sees the public contract first
class SectionState(Enum): ... # public type
DISPLAY_MESSAGES = {...} # public constant
classify_document(...) # public entry point - reader knows what the module does
classify_section_text(...) # public secondary function
_to_quality(...) # private: used by classify_document
_SECTION_TARGETS = [...] # private data: used by classify_document
_extract_sections(...) # private: used by classify_document
_find_section(...) # private: used by classify_document
```
---
## Key Distinctions
### Constants/config go BEFORE the public API
Signal patterns, registries, lookup tables - these are *configuration* for the
public API, not implementation details. A reader benefits from seeing them before
the function that uses them.
```python
# RIGHT - reader sees the failure signals before the function that checks them
_LLM_FAILURE_PATTERNS = [re.compile(...), ...]
def classify_section_text(text, *, is_ai_fillable):
    for pat in _LLM_FAILURE_PATTERNS:
        ...
```
### Private helpers go AFTER the public API
`_prefixed` functions are implementation - the "how", not the "what".
```python
# RIGHT
def classify_document(docx_path):
    sections = _extract_sections(docx_path) # called here
    ...
def _extract_sections(doc_path): # defined after its caller
    ...
```
### Nested functions stay inside their owner
If a helper is only used inside one function and is short (< 10 lines), a nested
`def` is fine. It stays with its owner rather than polluting the module's private
section.
---
## Classes
Same rule inside a class: public methods first, private methods at the bottom.
```python
class SecurityAgent:
    def analyze(self): # public - first
        ...
    def process(self): # public - second
        ...
    def _build_prompt(self): # private - after all public methods
        ...
    def _parse_response(self): # private - bottom
        ...
```
---
## Review Checklist
When reviewing or writing a Python module, verify in order:
- [ ] Module docstring explains WHAT the module does, not HOW
- [ ] Public types (`class`, `@dataclass`, exceptions) are near the top
- [ ] Constants and registries appear before the public API that uses them
- [ ] The first non-private function a reader hits is the highest-level entry point
- [ ] Private (`_`) functions appear AFTER all public functions
- [ ] Within the private section, callers appear before callees
- [ ] No public function is buried below its own private helpers
---
## Common Violations
| Violation | Symptom | Fix |
|-----------|---------|-----|
| Entry point at the bottom | Reader must scroll past 200 lines of helpers to find `main()` or `process()` | Move public API up |
| Private helpers before public API | `_parse_response` defined before `analyze()` | Reorder - private goes below |
| Dead code from copied private helpers | `_boilerplate_parts`, `_is_template_line` defined but never called | Delete it |
| Public type buried mid-file | `@dataclass class Result` appears between two private helpers | Move to top with other public types |
---
## What NOT to Do
Do not reorder for its own sake if the module is under ~80 lines and already readable.
Ordering is a `Nitpick` unless it materially increases cognitive load or makes the
public API hard to find.
