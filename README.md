# Claude Code Skills

A curated collection of skills that extend [Claude Code](https://claude.ai/claude-code) with
specialized knowledge, workflows, and domain expertise.

## What are skills?

Skills are modular, self-contained packages that give Claude Code deep expertise in specific
domains. When a relevant skill is installed, Claude automatically loads the right context and
follows the right workflow — no extra prompting needed.

Each skill is a directory with a `SKILL.md` file (and optional reference docs and scripts):

```
skill-name/
├── SKILL.md          # Core instructions — triggers automatically
└── references/       # Deep-dive docs loaded only when needed
```

## Installing a skill

Copy the skill directory into `~/.claude/skills/`:

```bash
cp -r autoresearch ~/.claude/skills/
```

Claude Code picks it up immediately — no restart needed.

## Skills

### [autoresearch](./autoresearch/)

Set up and run autonomous AI research loops based on
[Andrej Karpathy's autoresearch](https://github.com/karpathy/autoresearch).

Turns Claude Code into an unsupervised overnight optimizer: it proposes experiments, runs them,
keeps improvements, reverts failures, and loops until you stop it — ~12 experiments/hour.

**The skill helps you:**
- Write a high-quality `program.md` strategy document (the human's primary lever)
- Install and run the original LLM training target (minimize `val_bpb`)
- Adapt the loop to any measurable target: bundle size, test speed, Lighthouse scores, API latency, build time

**Triggers on:** "autoresearch", "autonomous experiments", "overnight optimization", "program.md"

**References:**
- [`references/program-md-guide.md`](./autoresearch/references/program-md-guide.md) — Full program.md templates, section-by-section writing guide, simplicity criterion calibration, common mistakes
- [`references/adapting-targets.md`](./autoresearch/references/adapting-targets.md) — Step-by-step guides for 5 non-LLM optimization targets

---

## Contributing

Skills in this repo follow the structure defined by the
[Claude Code skill format](https://github.com/anthropics/claude-code).

To add a new skill:

1. Create a directory: `skill-name/`
2. Add `SKILL.md` with YAML frontmatter (`name`, `description`) and instructions
3. Add `references/` docs for anything too detailed for `SKILL.md`
4. Open a PR

**Good skill candidates:** any domain where Claude needs procedural knowledge it doesn't already
have — proprietary APIs, specialized workflows, domain-specific schemas, opinionated tooling.

**Keep skills lean:** the `SKILL.md` body should stay under 500 lines. Move details to
`references/` files and link to them from `SKILL.md`.
