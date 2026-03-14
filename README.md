# Coding Agent Skills

A curated collection of skills for AI coding agents — autonomous workflows, domain expertise,
and specialized knowledge packs that make your agent dramatically more effective.

---

## Installing skills

Skills are installed via [skills.sh](https://skills.sh), which automatically detects your
coding agent and installs to the right location.

```bash
npx skills add <owner>/<skill-name>
```

No pre-installation required — runs directly via `npx`. Supports 38+ agents including Claude
Code, Cursor, Cline, Windsurf, GitHub Copilot, Continue, Codex, Warp, and more.

See the [skills.sh CLI docs](https://skills.sh/docs/cli) for the full command reference
(`list`, `remove`, `update`, `find`, etc.).

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

## Skill format

All skills use the `SKILL.md` standard:

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
  One or two sentences. Be specific about triggers.
---
```

Keep `SKILL.md` under 500 lines. Move detailed reference material to `references/` and link to
it — agents load reference files only when they need them.

---

## Contributing

1. Create a directory: `skill-name/`
2. Add `SKILL.md` with frontmatter and instructions
3. Add `references/` docs for anything too detailed for `SKILL.md`
4. Update this README with a summary entry
5. Open a PR
