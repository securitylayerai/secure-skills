# Coding Agent Skills

A curated collection of skills for AI coding agents — autonomous workflows, domain expertise,
and specialized knowledge packs that make your agent dramatically more effective.

Skills work across all major coding agents. Install once, use everywhere.

---

## Skills

### [autoresearch](./autoresearch/)

Set up and run autonomous AI research loops based on
[Andrej Karpathy's autoresearch](https://github.com/karpathy/autoresearch).

Turns your coding agent into an unsupervised overnight optimizer: it proposes experiments, runs
them, keeps improvements, reverts failures, and loops until you stop it (~12 experiments/hour).

**The skill helps you:**
- Write a high-quality `program.md` strategy document (the human's primary lever)
- Install and configure the original LLM training target (minimize `val_bpb`)
- Adapt the loop to any measurable target: bundle size, test speed, Lighthouse scores, API latency, build time

**Triggers on:** "autoresearch", "autonomous experiments", "overnight optimization", "program.md"

**Includes:**
- [`SKILL.md`](./autoresearch/SKILL.md) — Installation guide, loop mechanics, program.md quickstart
- [`references/program-md-guide.md`](./autoresearch/references/program-md-guide.md) — Full templates, section-by-section writing guide, simplicity criterion calibration, common mistakes
- [`references/adapting-targets.md`](./autoresearch/references/adapting-targets.md) — Step-by-step setup for 5 non-LLM optimization targets

---

## Installing a skill

Skills use a common `SKILL.md` format. Most agents pick them up by copying the skill directory
to the right location. Find your agent below.

### Claude Code

```bash
cp -r autoresearch ~/.claude/skills/
```

Skills are discovered automatically. The agent loads the full skill when it determines it's
relevant based on the `description` field in `SKILL.md`.

- **Project-scoped:** `.claude/skills/<skill-name>/SKILL.md`
- **Global:** `~/.claude/skills/<skill-name>/SKILL.md`

### pi (badlogic/pi)

```bash
cp -r autoresearch ~/.pi/agent/skills/
```

Invoke manually with `/skill:autoresearch` or let the agent auto-detect from the description.

- **Project-scoped:** `.pi/skills/<skill-name>/SKILL.md`
- **Global:** `~/.pi/agent/skills/<skill-name>/SKILL.md`

### Cursor

Cursor uses `.mdc` rule files instead of `SKILL.md`. Adapt the skill by copying its contents
into a new rule file:

```bash
# Create the rules directory if it doesn't exist
mkdir -p .cursor/rules

# Copy skill content as a Cursor rule
cp autoresearch/SKILL.md .cursor/rules/autoresearch.mdc
```

Add YAML frontmatter to the `.mdc` file to control when the rule activates:

```yaml
---
description: "Autoresearch autonomous experiment loop setup and program.md writing"
alwaysApply: false
---
```

- **Project-scoped:** `.cursor/rules/*.mdc`
- **Global:** Cursor Settings → Rules (UI only)

### Windsurf

```bash
mkdir -p .windsurf/rules
cp autoresearch/SKILL.md .windsurf/rules/autoresearch.md
```

- **Project-scoped:** `.windsurf/rules/*.md`
- **Global:** `~/.codeium/windsurf/memories/global_rules.md` (single file, append content)

### GitHub Copilot

```bash
cp autoresearch/SKILL.md .github/instructions/autoresearch.instructions.md
```

Add frontmatter to scope it:
```yaml
---
applyTo: "**"
---
```

Or add key content to `.github/copilot-instructions.md` for repo-wide always-on context.

- **Repo-wide:** `.github/copilot-instructions.md`
- **Scoped:** `.github/instructions/*.instructions.md`

### Cline

```bash
mkdir -p .clinerules
cp autoresearch/SKILL.md .clinerules/autoresearch.md
```

Cline loads all `.md` and `.txt` files in `.clinerules/` automatically at the start of every task.

- **Project-scoped:** `.clinerules/*.md`
- **Global:** `~/Documents/Cline/Rules/` (macOS/Linux), `Documents\Cline\Rules\` (Windows)

### Continue

```bash
mkdir -p .continue/rules
cp autoresearch/SKILL.md .continue/rules/autoresearch.md
```

Add frontmatter to control loading:
```yaml
---
name: autoresearch
alwaysApply: false
description: "Autoresearch autonomous experiment loop setup and program.md writing"
---
```

- **Project-scoped:** `.continue/rules/*.md`
- **Global:** `~/.continue/rules/*.md`

### Aider

Aider doesn't auto-load files — pass the skill explicitly at startup:

```bash
aider --read autoresearch/SKILL.md
```

Or add it to `.aider.conf.yml` to auto-load in a project:

```yaml
read: autoresearch/SKILL.md
```

### Zed

Zed picks up any of these files from the project root automatically:
`.rules`, `CLAUDE.md`, `AGENTS.md`, `AGENT.md`

```bash
cat autoresearch/SKILL.md >> .rules
```

Or use the built-in Rules Library (Agent Panel → `...` → "Rules...") to paste skill content
and set it as a default or `@mention` it on demand.

### Devin

Devin uses playbooks attached per-session via the web UI. Rename the skill file and drag it in:

```bash
cp autoresearch/SKILL.md autoresearch.devin.md
```

Then upload `autoresearch.devin.md` when starting a Devin session at `app.devin.ai`.

---

## Skill format

All skills in this repo use the `SKILL.md` standard compatible with Claude Code and pi:

```
skill-name/
├── SKILL.md          # Required. YAML frontmatter + Markdown instructions.
└── references/       # Optional. Detailed docs loaded only when needed.
    └── topic.md
```

**`SKILL.md` frontmatter:**

```yaml
---
name: skill-name          # lowercase, hyphens only
description: >            # what it does + when to trigger it
  One or two sentences.
  Be specific about triggers.
---
```

Keep `SKILL.md` under 500 lines. Move detailed reference material to `references/` and link to
it from `SKILL.md` — agents load reference files only when they need them.

---

## Contributing

To add a new skill:

1. Create a directory: `skill-name/`
2. Add `SKILL.md` with frontmatter (`name`, `description`) and instructions
3. Add `references/` docs for anything too detailed for `SKILL.md`
4. Update this README with a summary entry
5. Open a PR

**Good skill candidates:** any domain where an agent needs procedural knowledge it doesn't
already have — proprietary APIs, specialized workflows, domain-specific schemas, opinionated
tooling, agentic loops, research frameworks.

**Keep skills lean:** `SKILL.md` should contain only what the agent needs to act. Everything
else belongs in `references/`.
