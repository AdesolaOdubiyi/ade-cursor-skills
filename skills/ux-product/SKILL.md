---
name: ux-product
description: Activate when designing, building, or reviewing any user-facing feature, interaction, screen, or flow. Forces product thinking before and during implementation.
---
# UX / Product Thinking
## Step 1 — Understand user and goal
- Who is using this?
- What are they trying to accomplish?
- What is the primary action on this screen?
## Step 2 — Validate flow quality
- Is the flow understandable without docs?
- Is state clarity obvious (loading, empty, error, success)?
- Is there avoidable friction?
## Step 3 — Validate output quality
- Can users understand result quickly?
- Is next action obvious?
- Is trust/uncertainty communicated clearly?

## After implementation (merge gate)
Before merging UI changes, run **`frontend-pre-merge-reviewer`** to verify the code matches the decisions above (states, flow, edge cases) and to check framework-specific failure modes.
