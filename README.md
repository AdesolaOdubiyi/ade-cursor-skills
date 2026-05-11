# ade-cursor-skills

Reusable rules and skills for software engineering workflows — primarily backend, but covering the full stack. Drop them into any project to encode repeatable engineering processes so you stop re-explaining the same standards to an agent every session.

See `how-to-use.md` for workflow guidance and sample prompts.

## Credits

Several skills are adapted from [Matt Pocock's skills catalog](https://github.com/mattpocock/skills): `improve-codebase-architecture`, `grill-with-docs`, `write-a-skill`, and `caveman`.

## Install Into Any Project

Jump down to the end of the prompt for the "skills-to-skip" list, edit it, then paste this prompt into your agent:

> You are about to install a set of reusable rules and skills into this project.
>
> First, identify which agent you are (Cursor, Claude Code, Codex, Windsurf, etc.).
> Then use web browse to look up the correct file paths and formats for rules and skills
> for your specific agent — for example, Cursor uses `.cursor/rules/` and `.cursor/skills/`,
> while Claude Code uses `CLAUDE.md` for rules. Use the conventions native to your agent.
>
> Then fetch and clone `https://github.com/adesolaodubiyi/ade-cursor-skills` into a temp
> directory and install:
> - All files under `rules/` into the correct rules location for your agent, converting
>   format if needed (e.g. `.mdc` files to inline prose for agents that use `CLAUDE.md` or `AGENTS.md`)
> - All folders under `skills/` into the correct skills location for your agent
>
> Before installing, ask: should these be installed at the workspace level or project level only?
>
> Skip any skills listed below. Delete the temp clone when done.
>
> **Skills to skip:** *(e.g. `security-auditor`, `tdd`)*
>
> After installing, list every skill and rule file created and confirm the paths used.

Remove any skills you don't need after the agent runs.

## Rules

| Rule | Purpose |
|---|---|
| `baseline-coding-standard.mdc` | Naming, function design, error handling, comments, and module organization |
| `engineering-behavior.mdc` | Agent collaboration behavior and scope discipline |
| `python-backend.mdc` | Python typing, logging, secrets, and boundary standards |

## Skills

| Skill | Purpose |
|---|---|
| `architecture-and-api` | Module structure and API design |
| `backend-pre-merge-reviewer` | Pre-merge backend code review |
| `caveman` | Ultra-brief communication mode |
| `clean-code-reviewer` | Clean code standards review |
| `debugging-systematic` | Structured bug diagnosis |
| `design-to-build` | Design → phased TDD build → review gates with CONTEXT + KNOWLEDGEBASE + architecture plan |
| `frontend-pre-merge-reviewer` | UI merge review with ux-product alignment and framework-aware failure modes |
| `grill-with-docs` | Stress-test plans against domain docs |
| `improve-codebase-architecture` | Find refactoring opportunities |
| `python-top-down` | Enforce top-down Python module ordering |
| `python-type-discipline` | Python typing standards |
| `readme-writer` | README generation |
| `security-auditor` | Adversarial multi-pass security audit |
| `systems-design` | Systems design with trade-off analysis and pushback protocol |
| `tdd` | Test-driven development loop |
| `technical-docs-writer` | Technical documentation |
| `ux-product` | Product and UX thinking |
| `write-a-skill` | Author new skills |
