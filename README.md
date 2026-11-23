# Runtime contract

Bohrium Playground Period 4 submission — challenge `product-arena-04-4b82ad21` (track: `academic-writing`).

This repository holds the complete skill package for the challenge: the skill
definition, its executable entry point and all supporting scripts. No precomputed
answers, private data or credentials are included.

## Skill

- Name: `intro-related-work-builder`
- Summary: Draft a five-move introduction and a theme-grouped related-work section from supplied background notes and a reference library, keeping every sentence traceable and marking missing gap or contribution material instead of inventing it. Use when a task asks for an introduction, literature review or research-gap narrative with correct citation placement.

## Contents

| File |
|------|
| `SKILL.md` |
| `scripts/run_skill.py` |
| `README.md` |

## How to run

```bash
python scripts/run_skill.py --case /input/case.json --output /app/output/result.json
```

The entry point is `SKILL.md`; input/output contracts, error handling and a minimal
usage example are documented there.

## Notes

- Pure Python standard library; no network access required.
- The result JSON keeps the input case id, the main results, per-item evidence,
  a run status and an uncertainty section.
