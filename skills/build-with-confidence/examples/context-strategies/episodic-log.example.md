# Example: episodic build log entry

Append to `docs/build-log/2026-06-28.md` (create file if missing):

```markdown
## Build: batch CLI + ranked locations + essay MANUAL_REVIEW

- Tests: pytest tests/test_link_batch.py tests/test_apply_batch.py tests/test_browser_agents.py -q → 34 passed
- Batch CLI: `python -m src.cli.apply_batch data_folder/application_links.txt`
- Parallel: headless only, max 5 workers; headed = 1 worker (shared chrome profile)
- Smokes: not re-run (RUN_GATE2_SMOKES=0)
- Blockers unchanged: Ashby hCaptcha, Figma custom Qs
```

**Do not** copy this into `CONTEXT.md` — only distilled facts (e.g. "batch CLI exists") belong there.
