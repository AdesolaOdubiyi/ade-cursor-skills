# Example: AIHawk ADE context strategy

**Project type:** Partial-automation job application filler (browser tier)

## Canonical files

| Layer | File | Update when |
|-------|------|-------------|
| Semantic / product | `CONTEXT.md` | Scope or invariants change |
| Semantic / ops | `docs/STATUS.md` | Every build (checkmarks) |
| Semantic / decisions | `docs/DECISIONS.md` | Irreversible choices only |
| Operator | `docs/RUNBOOK.md` | CLI or env vars change |
| Episodic | `docs/build-log/YYYY-MM-DD.md` | Each build session |
| Episodic | `data_folder/batch_results.jsonl` | Each batch run |
| Handoff | `docs/HANDOFF.md` | End of multi-session work |

## Consolidation pattern (this project)

1. After smokes or batch runs, append one line to build-log — do **not** paste smoke output into CONTEXT.
2. Promote repeated MANUAL_REVIEW fields from `unmapped_questions.example.yaml` → profile + `field_mapping.yaml` → remove from "unmapped" narrative in STATUS.
3. Replace STATUS rows like "Batch CLI not built" with `[x]` when `src/cli/apply_batch.py` ships — don't leave both old and new bullets.
4. Location prefs: single source in `candidate_profile.yaml` → `location_preference_ranked`; CONTEXT mentions ranked fallback, not per-employer hacks.

## Ranked location prefs (user-specific, stable)

1. New York City  
2. Northern NJ (Weehawken, Jersey City, Newark)  
3. Boston  
4. Texas  
5. Atlanta  
6. California  

Rationale: summer 2026 internships; NYC long-term preference; East Coast over West Coast for full-time life.

## Role essays

Flag `MANUAL_REVIEW` — no LLM auto-fill unless explicitly in `common_answers`.
