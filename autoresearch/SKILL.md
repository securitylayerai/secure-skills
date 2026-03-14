---
name: autoresearch
description: >
  Set up and run autonomous AI research loops using Andrej Karpathy's autoresearch framework.
  Helps users write high-quality program.md strategy files, configure the experiment loop, and
  adapt autoresearch to any measurable optimization target (LLM training, bundle size, test speed,
  Lighthouse scores, etc.). Use when the user mentions "autoresearch", "autonomous experiments",
  "overnight optimization", wants to run self-improving AI research loops, or wants to set up
  Claude Code as an autonomous optimizer. Also triggers when the user wants to write or improve
  a program.md file.
---

# autoresearch Skill

Autoresearch is a framework by [Andrej Karpathy](https://github.com/karpathy/autoresearch) that
turns Claude Code (or any coding agent) into an autonomous research loop: it proposes experiments,
runs them, keeps improvements, reverts failures, and loops overnight — no human needed.

The human's only job is to write `program.md` well. That is this skill's primary purpose.

---

## How autoresearch works

The loop runs inside whatever coding agent is open on the repo:

```
1. Read program.md (strategy document)
2. Edit the target file with an experiment idea
3. git commit
4. Run the benchmark command
5. Parse the result metric
6. If improved → keep commit (advance branch)
7. If worse/equal → git reset --hard to prior commit
8. Log to results.tsv
9. GOTO 1 (never stop)
```

**Key constraint**: the benchmark command and evaluation metric are frozen. Only the target file
(e.g. `train.py`) is modified. This ensures every experiment is compared fairly.

**Throughput**: ~12 experiments/hour at 5 min/run. An overnight session = ~80–100 experiments.

---

## Installation (original LLM training target)

Requirements: single NVIDIA GPU, CUDA 12.8+, Python 3.11+.

```bash
# Clone
git clone https://github.com/karpathy/autoresearch
cd autoresearch

# Install uv (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# One-time data prep (downloads dataset shards + trains BPE tokenizer, ~2 min)
uv run prepare.py

# Verify setup: should train for 5 min and print val_bpb
uv run train.py
```

For smaller GPUs (4GB–16GB VRAM), edit `prepare.py` before data prep:
```python
MAX_SEQ_LEN = 256        # was 2048
EVAL_TOKENS = 5_000_000  # was ~20M
```
And adjust in `train.py`:
```python
DEPTH = 4               # was 8
TOTAL_BATCH_SIZE = 2**14 # was 2**19
WINDOW_PATTERN = "L"   # remove sliding window
```

---

## Launching Claude Code in autonomous mode

```bash
# In the repo directory:
claude --dangerously-skip-permissions
```

Then send the first message:
```
Have a look at program.md and kick off a new autoresearch experiment. Do the setup first.
```

Claude will: create a branch `autoresearch/<date>`, run the baseline, initialize `results.tsv`,
then loop indefinitely.

---

## Writing program.md — the core task

`program.md` is the research strategy document. It is the only thing the human edits between
sessions. A poor `program.md` produces unfocused experiments. A great one produces breakthroughs.

**When a user asks to write or improve program.md, gather these inputs first:**

### Required questions

1. **Optimization target**: What metric are you minimizing/maximizing?
   - Lower `val_bpb` (language model perplexity)
   - Smaller bundle size (`du -sb dist/`)
   - Faster test suite (`time bun test`)
   - Better Lighthouse score
   - Lower API latency
   - Other (ask for the exact command + what "better" means)

2. **Target file**: What file does the agent edit? (e.g. `train.py`, `vite.config.ts`, `index.ts`)

3. **Frozen constraints**: What must NOT be changed?
   - Evaluation function
   - Dataset / test suite
   - Public API surface
   - Dependencies list

4. **Research directions**: What ideas does the user want explored?
   - Architecture changes (attention variants, activation functions, normalization)
   - Optimizer tuning (learning rates, schedules, algorithms)
   - Data processing changes
   - Bundler/build config changes
   - Other domain-specific ideas

5. **Dead-ends** (optional): What has already been tried and failed?

6. **Simplicity preference**: How much complexity is acceptable per unit of improvement?
   - Conservative: "only keep changes that improve by >0.5% AND stay under 20 extra lines"
   - Aggressive: "keep anything that improves the metric, even complex"

### program.md anatomy

A complete program.md has these sections in order:

```markdown
# Research Goal
[What metric, what direction, why it matters]

# Benchmark
[Exact command to run + how to parse the result]

# What Can Be Changed
[Exhaustive list of allowed modifications]

# What Cannot Be Changed
[Frozen files, frozen functions, invariants]

# Research Directions
[Bullet list of ideas to explore, grouped by theme]

# Dead Ends
[Ideas already tried that hurt or had no effect — never retry these]

# Simplicity Criterion
[Explicit rule for when to keep vs revert a change]

# Experiment Loop Rules
[Agent behavior rules: never stop, crash handling, git protocol, logging format]

# Logging
[results.tsv column format and update instructions]
```

For a complete template with filled examples for both LLM training and general targets, see
[references/program-md-guide.md](references/program-md-guide.md).

---

## Adapting to non-LLM targets

autoresearch works for **any command that outputs a number**. See
[references/adapting-targets.md](references/adapting-targets.md) for step-by-step guides for:

- JavaScript/TypeScript bundle size
- Test suite speed
- Lighthouse / web performance
- API endpoint latency
- Compiler / build time

---

## After an overnight run

```bash
# Review all experiments
cat results.tsv

# See what survived (all commits on the branch)
git log --oneline

# Plot the improvement curve (if results.tsv has many rows)
# val_bpb column should trend downward
```

Bring the results back into a new Claude session to analyze patterns: which ideas worked, which
classes of experiment were most productive, and update `program.md` with new research directions
and confirmed dead-ends before the next run.
