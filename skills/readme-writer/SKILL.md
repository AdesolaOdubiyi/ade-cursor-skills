---
name: readme-writer
description: |
  Write or rewrite a README for any project.
  Activate when user says: write a readme, create a readme, update the readme,
  document this project, write docs for this, readme for this.
---

# README Writer

## What a README Is
Reference material for engineers. Not a pitch. Not a tutorial. Not a defense of your choices.
The reader is a competent engineer. Write accordingly.

---

## Banned AI Writing Patterns
These patterns are prohibited. If you catch yourself doing any of them, rewrite the sentence.

**Contradiction sentences** — stating what something is NOT
- ❌ "URLs come only from the environment, not from source strings"
- ✅ "URLs are read from environment variables"
- ❌ "This is real batch logic, not a toy program"
- ✅ Delete it. The code speaks for itself.

**Self-defense / justification** — explaining why a choice is valid
- ❌ "This keeps the emphasis on data engineering practice and avoids shipping copy-paste operational recipes"
- ✅ State what it does. Never justify it unless the decision is non-trivial.

**Notability claims** — asserting the project is good, serious, or substantial
- ❌ "The project does something substantive offline"
- ❌ "This is a durable, production-grade pipeline"
- ✅ Describe what it does. Let the reader decide.

**Teacher voice** — explaining things a competent engineer already knows
- ❌ "Offline mode means the client reads from disk instead of opening HTTP connections, so the pipeline runs without a network"
- ✅ "Offline mode reads from `data/sample/offline_vendor.json`"

**Glue word openers** — Furthermore, Moreover, Additionally, It is worth noting, It is important to note
- These never appear in a README. Delete them on sight.

**Parallel list padding** — three-part sentences where two parts add nothing
- ❌ "It presents the features, emphasizes the strengths, and explains the benefits"
- ✅ Pick the one thing that matters and say it

**Hedging qualifiers** — "when configured", "if applicable", "where relevant", "in most cases"
- Only include a qualifier if omitting it would cause a real failure

**Defensive parentheticals** — "(no network)", "(real logic, not a stub)", "(never committed)"
- If it needs a parenthetical, rewrite the sentence or cut the point entirely

**Vague superlatives** — robust, seamless, comprehensive, powerful, elegant, clean
- These words are banned. Use specifics or nothing.

**Uniform sentence rhythm** — every sentence the same length, same structure
- Vary length deliberately. Short sentences land harder. Use them.

**Overclaiming specificity** — fake-precise numbers or outcomes with no source
- ❌ "Reduces processing time by 40%"
- ✅ State the mechanism, not a made-up metric

---

## README Structure by Project Type

Determine the project type first. Use the appropriate structure. Do not add sections that aren't in the template unless there is a clear reason the reader needs them.

### Library / Tool
```
# Name
One sentence: what it does.

## Install
[command]

## Usage
[minimal working example]

## API
[only what isn't obvious from usage]

## Configuration
[table: key | default | description]
```

### Service / Backend
```
# Name
One sentence: what it does and what it serves.

## Stack
[language, framework, DB — no fluff]

## Run Locally
[exact commands, in order]

## Environment Variables
[table: var | required | description]

## Architecture
[only if non-trivial — module map or brief prose]
```

### Data Pipeline / Research
```
# Name
What the pipeline does. Inputs and outputs in one sentence.

## Run
[exact commands]

## Inputs
[format, location, how to supply them]

## Outputs
[what files, where, what they contain]

## Configuration
[config file fields that affect behavior]

## Schema Notes
[only if schema is non-obvious or has traps]
```

### Portfolio / Personal Project
```
# Name
What problem it solves. One sentence.

## Stack
[tech choices — no justification unless the choice is genuinely surprising]

## Run
[exact commands]

## Notes
[non-trivial decisions only — one sentence each]
```

---

## Writing Rules

**Tense and voice**
- Present tense. Active voice.
- "The client reads from disk" not "disk is read by the client"

**Explanations**
- Explain a decision only if a reasonable engineer would make the wrong assumption without it
- One sentence per explanation. Justification if necessary.
- Never explain what the code already shows

**Commands and paths**
- Always use exact commands. Never "something like" or "for example, you might run"
- Always use exact file paths. Never "the config file" when you mean `config.yaml`
- If there are multiple ways to run something, list all of them without ranking them

**Tables over prose**
- Environment variables → table
- Configuration options → table
- File layout → table or tree
- Anything with more than two key/value pairs → table

**Length**
- If a section can be deleted without losing information the reader needs, delete it
- If a sentence can be cut in half without losing meaning, cut it
- No introductory sentences that restate the section header

**The final check**
Before finishing, read the README and ask:
1. Does any sentence defend, justify, or congratulate the code? → Delete it
2. Does any sentence explain something the reader already knows? → Delete it
3. Is every command exact and runnable? → Fix any that aren't
4. Is every file path specific? → Fix any that use generic names
5. Would a new engineer know exactly how to run this in under 2 minutes? → If not, fix the gap