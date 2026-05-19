# Agent Instructions — agent-skills

This repository contains **GitHub Copilot agent skills**. When working here, follow the conventions below.

## Repository Purpose

Skills are self-contained Markdown files that extend Copilot's behaviour for a specific domain. Each skill has:
- A `SKILL.md` (frontmatter + system prompt)
- An `evals/evals.json` (evaluation test cases)
- An optional `references/` directory with supporting Markdown documents

## Conventions

### SKILL.md Frontmatter

```yaml
---
name: <kebab-case-name>
description: >
  <Trigger description. Must be explicit about keywords and scenarios
  that should cause this skill to fire. 2–4 sentences.>
---
```

- `name` must match the directory name exactly.
- `description` is used by the skill router: be specific about trigger terms.

### Evals Format (`evals/evals.json`)

```json
{
  "skill_name": "<name>",
  "evals": [
    {
      "id": 1,
      "prompt": "<User prompt that should trigger this skill>",
      "expected_output": "<Description of what a correct response contains>",
      "files": []
    }
  ]
}
```

- Minimum 3 eval cases per skill.
- `expected_output` should describe observable properties, not the exact text.
- `files` lists any files that must be present in context for the eval to run.

### Reference Files

- Store domain reference material in `references/*.md`.
- Reference them explicitly in `SKILL.md` using relative paths, e.g. `See references/my-ref.md`.
- Keep reference files factual and stable; put opinionated guidance in `SKILL.md` itself.

## When Editing Skills

- **Never downgrade domain constraints** in a skill (e.g., do not remove the TOSCA 2.0-only restriction from the `tosca` skill).
- **Update evals** whenever the skill's behaviour changes — they are the source of truth for expected output.
- **Keep `SKILL.md` concise**: reference files exist to offload bulky catalogs and examples.
- After any change, run evals with the `skill-creator` skill to verify no regression.

## Adding a Skill

1. `mkdir skills/<name>`
2. Create `SKILL.md`, `evals/evals.json`, and optionally `references/`.
3. Register the skill in `README.md` skills table.
4. Run at least one eval round before merging.
