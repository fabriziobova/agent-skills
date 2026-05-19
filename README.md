# agent-skills

A curated collection of **GitHub Copilot agent skills** — self-contained, prompt-driven extensions that teach Copilot how to handle specialized domains.

## Repository Structure

```
skills/
└── <skill-name>/
    ├── SKILL.md          # Skill definition: frontmatter + system prompt
    ├── evals/
    │   └── evals.json    # Evaluation test cases for the skill
    └── references/       # Supporting reference documents loaded at runtime
        └── *.md
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [`tosca`](skills/tosca/) | Expert skill for writing, editing, and validating **TOSCA 2.0** files (OASIS standard for cloud topology and orchestration). Exclusively targets TOSCA 2.0 — never produces TOSCA 1.x syntax. |

## How a Skill Works

Each skill lives in `skills/<name>/` and consists of:

- **`SKILL.md`** — The core skill file. The YAML frontmatter (`name`, `description`) is used by the Copilot CLI to register and trigger the skill. The Markdown body is the system prompt injected when the skill is invoked.
- **`evals/evals.json`** — A list of prompt/expected-output pairs used to measure skill quality. Run evaluations via the `skill-creator` meta-skill.
- **`references/`** — Supplementary Markdown files that the skill's system prompt references. These are loaded on demand to provide domain knowledge without bloating the core prompt.

## Adding a New Skill

1. Create a new directory under `skills/<your-skill-name>/`.
2. Add a `SKILL.md` with the required frontmatter:
   ```yaml
   ---
   name: <skill-name>
   description: >
     <One or two sentences that describe when to trigger this skill.
     Be explicit about trigger keywords so the router fires correctly.>
   ---
   ```
3. Write the system prompt body in Markdown below the frontmatter.
4. Add at least 3 evaluation cases in `evals/evals.json`.
5. Optionally add reference files under `references/`.

## Installing Skills

### Personal skill (available in all your projects)

Copy the skill directory into `~/.copilot/skills/`:

```bash
# Clone once, then symlink or copy individual skills
git clone https://github.com/fabriziobova/agent-skills.git

# Install a specific skill (e.g. tosca)
cp -r agent-skills/skills/tosca ~/.copilot/skills/tosca
```

### Project skill (available only in a specific repository)

Copy the skill directory into `.github/skills/` inside your project:

```bash
mkdir -p .github/skills
cp -r /path/to/agent-skills/skills/tosca .github/skills/tosca
```

### Using `gh skill` (GitHub CLI)

If you have the `gh` CLI with the `skill` extension:

```bash
gh skill install fabriziobova/agent-skills/skills/tosca
```

### Reload skills in a running session

If Copilot CLI is already open, reload without restarting:

```
/skills reload
```

Verify the skill is active:

```
/skills info tosca
```

---

## Running Evaluations

Use the built-in `skill-creator` skill to run evals and benchmark a skill:

```
@copilot run evals for the tosca skill
```

or

```
@copilot optimize the tosca skill description for better trigger accuracy
```
