# [Project Name] — Domain Language

Read this file at the start of every session. It defines the canonical terms for this project.
Use these terms exactly. Consistent language prevents AI drift across sessions.

---

## Language

<!-- Define each term that is specific to this project or feature.
     General programming concepts (hooks, middleware, handlers) do not belong here.
     Only terms that could be confused or used inconsistently across sessions. -->

**[Term]**:
[One sentence definition of what it IS, not what it does.]
_Avoid_: [synonym1], [synonym2]

**[Term]**:
[One sentence definition.]
_Avoid_: [synonym]

---

## Relationships

<!-- Express how domain terms relate to each other. Use cardinality where obvious. -->

- A **[Term]** contains one or more **[Term]**
- A **[Term]** belongs to exactly one **[Term]**

---

## Example Dialogue

<!-- A short conversation demonstrating how the terms interact naturally.
     This helps AI agents understand boundaries between related concepts. -->

> **Dev:** "When a **[Term]** is [action], does a **[Term]** get created immediately?"
> **Domain expert:** "No — a **[Term]** is only created after **[Term]** is confirmed."

---

## Flagged Ambiguities

<!-- Call out any term that was used inconsistently and how it was resolved. -->

- "[word]" was used to mean both **[Term A]** and **[Term B]** — resolved: these are distinct.
