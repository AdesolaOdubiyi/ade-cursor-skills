---
name: grilling
description: |
  One-question-at-a-time design grilling. Ask a single targeted question, wait for the answer,
  push back like a senior engineer if the answer introduces risk or ambiguity, then continue.
  Use inside design sessions to stress-test decisions before locking them.
---

# Grilling

Ask exactly one question per turn. Never ask two at once.

For each question:
- Provide your recommended answer (concrete, not "it depends")
- Give the reasoning behind your recommendation in one sentence
- Wait for the user's answer before continuing

After each answer:
- Acknowledge what was decided (one sentence)
- Identify any risk or ambiguity introduced by the answer
- Push back if the answer introduces scope creep, unquantified risk, or a decision that can't be reversed without pain — push back once, clearly, then accept the user's call
- Move to the next question

Continue until all open questions for the current decision are resolved.

Do not move to implementation, architecture lock, or document creation while questions remain open.
