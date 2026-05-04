# ade-cursor-skills
Reusable Cursor rules and skills for software engineering workflows.

## Credits

Several skills in this set are adapted from [Matt Pocock's skills catalog](https://github.com/mattpocock/skills), including `tdd`, `improve-codebase-architecture`, `grill-with-docs`, `write-a-skill`, and `caveman`.

## Install Into Any Project

Use this prompt with a Cursor agent:

> Clone the skills repository at `https://github.com/adesolaodubiyi/ade-cursor-skills` into a temp directory, then copy the contents of the `skills/` folder into this project's `.cursor/skills/` directory (create it if it doesn't exist). Skip any skills listed below. Delete the temp clone when done.
>
> **Skills to skip:** *(list any you don't want, e.g. `security-auditor`, `tdd`)*
>
> After copying, list every skill folder that was created so I can confirm.

## Setting Up Agent Skills

To install the standard Cursor skill set into this project, give the following prompt to a Cursor agent:

> Clone `https://github.com/adesolaodubiyi/ade-cursor-skills` into a temp directory,
> copy the `skills/` folder contents into `.cursor/skills/` (create it if needed),
> skip any skills not relevant to this project, then delete the temp clone and list
> what was installed.

After the agent runs, delete any skills you don't need from `.cursor/skills/`.

### Available skills
| Skill | Purpose |
|---|---|
| `architecture-and-api` | Module structure and API design |
| `backend-pre-merge-reviewer` | Pre-merge backend code review |
| `caveman` | Ultra-brief communication mode |
| `clean-code-reviewer` | Clean code standards review |
| `debugging-systematic` | Structured bug diagnosis |
| `grill-with-docs` | Stress-test plans against domain docs |
| `improve-codebase-architecture` | Find refactoring opportunities |
| `python-type-discipline` | Python typing standards |
| `readme-writer` | README generation |
| `security-auditor` | Adversarial multi-pass security audit |
| `systems-design` | Systems design guidance |
| `tdd` | Test-driven development loop |
| `technical-docs-writer` | Technical documentation |
| `ux-product` | Product and UX thinking |
| `write-a-skill` | Author new skills |
