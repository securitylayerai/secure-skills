# program.md Writing Guide

`program.md` is the research strategy document — the only file the human owns in an autoresearch
session. The agent reads it at the start of every experiment. Think of it as a combination of:
- **System prompt**: defines agent behavior and constraints
- **Research brief**: defines goals, directions, and dead-ends
- **Protocol spec**: defines the git/logging/crash-handling rules

A great `program.md` produces focused, productive experiments. A vague one wastes GPU hours.

---

## Table of Contents

1. [Full Template — LLM Training](#1-full-template--llm-training)
2. [Full Template — General Optimization Target](#2-full-template--general-optimization-target)
3. [Section-by-Section Guidance](#3-section-by-section-guidance)
4. [Research Directions — Writing Good Ideas](#4-research-directions--writing-good-ideas)
5. [Simplicity Criterion — How to Calibrate](#5-simplicity-criterion--how-to-calibrate)
6. [Common Mistakes](#6-common-mistakes)

---

## 1. Full Template — LLM Training

This is the standard template for the original [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
LLM training target.

```markdown
# Research Goal

Minimize `val_bpb` (bits-per-byte) on the held-out validation shard within a fixed 5-minute
training budget on a single GPU. Lower is better.

# Benchmark

\`\`\`bash
uv run train.py > run.log 2>&1
grep "^val_bpb:\|^peak_vram_mb:" run.log
\`\`\`

Parse `val_bpb` as the primary metric. Parse `peak_vram_mb` as a secondary constraint:
if it exceeds 80% of available VRAM, the experiment is too memory-hungry.

# What Can Be Changed

Only `train.py`. Specifically:
- Model architecture (attention, MLP, normalization, residual connections, layer order)
- Optimizer (algorithms, learning rate, schedule, weight decay, parameter groupings)
- Training loop (gradient accumulation, mixed precision, compile flags)
- Hyperparameter constants at the top of the file (DEPTH, WINDOW_PATTERN, LRs, batch size, etc.)
- Any imports from existing dependencies in `pyproject.toml`

# What Cannot Be Changed

- `prepare.py` (data loading, tokenizer, `evaluate_bpb` function — the metric is frozen)
- `pyproject.toml` (no new pip installs)
- `MAX_SEQ_LEN` (used in the frozen eval — changing it makes results incomparable)
- The 5-minute time budget (hardcoded in `prepare.py`)

# Research Directions

## Architecture
- Attention variants: sliding window sizes, attention patterns (SSSSL, SLSL, etc.)
- Alternative activation functions for the MLP: SwiGLU, GeGLU, ReGLU
- Normalization placement: pre-norm vs post-norm vs both
- Value residual connections (ResFormer): try different residual scalings
- Increasing head dim vs more heads at fixed parameter count
- Mixture of experts (MoE): route tokens to specialized FFN blocks

## Optimizer
- Muon-only (remove AdamW for embeddings, use Muon everywhere)
- Learning rate sensitivity: try MATRIX_LR ± 25%, see if current is optimal
- Warmup ratio: add a short warmup (0.02–0.05) and compare
- Different Adam betas: (0.9, 0.99) vs current (0.8, 0.95)
- Lion optimizer as an alternative to AdamW component

## Regularization
- Dropout at different positions (attention, residual, embedding)
- Weight tying (embedding and unembedding matrices)
- Gradient clipping values (currently none — try 1.0)

## Efficiency
- Reducing DEPTH and increasing width to match param count
- Smaller DEVICE_BATCH_SIZE for more gradient steps within the budget
- Torch compile with different mode settings

# Dead Ends

- [Update this section after each failed experiment]
- Example: "Removed value residuals entirely → +0.02 val_bpb (worse), 2025-01-15"

# Simplicity Criterion

Keep a change if:
- `val_bpb` improves by **any amount**, AND
- The code added is proportional: trivial change (swap a constant) can improve by 0.001;
  adding 30+ lines of new logic needs at least 0.005 improvement to justify the complexity

Actively prefer: removing code for equal or better performance ("simplification win").
Reject: complex changes with < 0.001 improvement.

# Experiment Loop Rules

**Never stop.** Do not ask "should I continue?" — run until manually interrupted (Ctrl+C).

**Git protocol:**
1. You are on branch `autoresearch/<date-tag>` (create it on first run)
2. Before each experiment: verify you are on this branch
3. After editing `train.py`: `git add train.py && git commit -m "<one-line description>"`
4. After running: check result
5. If improved: stay on current commit (branch advances)
6. If not improved: `git reset --hard HEAD~1` to revert

**Crash handling:**
- If `grep` returns empty (no val_bpb in log): it crashed
- Run `tail -n 50 run.log` to diagnose
- Attempt to fix once; if it crashes again, `git reset --hard HEAD~1` and try a different idea

**Timeout:**
- If a run exceeds 10 minutes wall clock, kill it and treat as a crash

**Baseline:**
- On the very first run, do NOT make any changes — run `train.py` as-is and log as "baseline"

# Logging

Maintain `results.tsv` (untracked by git — it persists across resets):

Columns: `commit`, `val_bpb`, `memory_gb`, `status`, `description`

- `status`: `improved` | `reverted` | `crash` | `baseline`
- `memory_gb`: peak_vram_mb / 1024, rounded to 2 decimal places
- `description`: the one-line commit message

After each experiment, append one row. Never modify previous rows.

Example:
\`\`\`
abc1234	0.9987	44.20	baseline	baseline run
def5678	0.9971	44.85	improved	try SwiGLU activation in MLP
ghi9012	0.9980	44.20	reverted	add dropout 0.1 before residual
jkl3456	0.9965	45.10	improved	sliding window pattern SSSSL
\`\`\`
```

---

## 2. Full Template — General Optimization Target

Use this when adapting autoresearch to a non-LLM target (bundle size, test speed, etc.).

```markdown
# Research Goal

Minimize [METRIC_NAME] produced by [BENCHMARK_COMMAND].
Lower/higher (specify which) is better. Baseline: [BASELINE_VALUE from first run].

# Benchmark

\`\`\`bash
[EXACT COMMAND TO RUN]
\`\`\`

Parse result: [HOW TO EXTRACT THE NUMBER — grep pattern, jq path, etc.]
Secondary constraints (optional): [memory, wall-clock time, correctness checks]

# What Can Be Changed

- [FILE 1]: [what aspects can be modified]
- [FILE 2]: [what aspects can be modified]
- [Specific patterns, configs, algorithms that are in scope]

# What Cannot Be Changed

- [Public API / interface contracts]
- [Evaluation/correctness tests — these must always pass]
- [External dependencies that can't be changed]
- [Files outside the target module]

# Research Directions

## [Theme 1]
- [Specific idea with brief rationale]
- [Specific idea with brief rationale]

## [Theme 2]
- [Specific idea with brief rationale]

# Dead Ends

- [Populate after each failed experiment]

# Simplicity Criterion

[Define explicitly. Example:]
Keep a change if it improves [METRIC] by more than [THRESHOLD] AND does not add more than
[N lines / complexity budget]. Prefer removing code for equal results.

# Experiment Loop Rules

**Never stop.** Run until manually interrupted.

**Git protocol:**
1. Branch: `autoresearch/<date-tag>`
2. Edit target file → `git add <file> && git commit -m "<description>"`
3. Run benchmark → parse metric
4. Improved → keep commit
5. Not improved → `git reset --hard HEAD~1`

**Crash/failure handling:**
- If benchmark fails (non-zero exit, parse error): diagnose once, attempt fix, revert if fails again
- If correctness checks exist (`checks.sh`): run after each passing benchmark; `checks_failed` status if they fail

**Baseline:** First run = no changes, just measure and log.

# Logging

Maintain `results.tsv`:
Columns: `commit`, `[METRIC]`, `status`, `description`
Status values: `baseline` | `improved` | `reverted` | `crash` | `checks_failed`
```

---

## 3. Section-by-Section Guidance

### Research Goal
- Be specific about direction: "minimize val_bpb" not "improve the model"
- State the constraint: "within 5 minutes of GPU training"
- One sentence is enough; don't over-explain

### Benchmark
- Must be **exactly** reproducible: copy-paste the command
- Include the grep/parse pattern so the agent doesn't have to infer it
- If the output format can vary, include a fallback parse strategy

### What Can Be Changed
- Err on the side of **explicit over implicit** — if it's not listed, the agent may avoid it
- Group by file or concern
- For LLM training: explicitly list hyperparameter constant names

### What Cannot Be Changed
- This is the most important section for correctness
- Always include the evaluation function (ensures fair comparison)
- Include API contracts if this is a library
- Include anything where a change would make results incomparable

### Research Directions
- Group by theme (architecture, optimizer, regularization, etc.)
- Each idea should be specific enough to act on: "try SwiGLU" not "try different activations"
- Include the **rationale** in parentheses for non-obvious ideas: "try weight tying (halves embedding params, may improve generalization)"
- 15–40 ideas is a good range. Too few → agent runs out; too many → agent gets unfocused
- Prioritize by likelihood of success: put highest-confidence ideas at the top of each section

### Dead Ends
- **Update after every session** — this compounds value across sessions
- Include: the idea, the direction of change, the magnitude of degradation, and the date
- Format: `"<idea> → <effect> (<date>)"`
- This prevents re-running failed experiments across context resets

### Simplicity Criterion
- Make it **quantitative** where possible: specify a threshold
- Two axes: improvement magnitude and code complexity added
- Include the "simplification win" concept: removing code for equal performance is always good
- Example calibration: a 0.001 val_bpb improvement (0.1%) justifies up to 5 lines; 0.01 (1%) justifies up to 50 lines

### Experiment Loop Rules
- **"Never stop"** must be explicit — agents tend to ask for permission
- Crash handling must be explicit — otherwise agents get stuck in crash loops
- Timeout rule prevents runaway experiments
- Git protocol must be unambiguous: branch name format, commit before run vs after, reset command

### Logging
- Untracked by git (add to `.gitignore`) so it survives `git reset --hard`
- TSV (not CSV) avoids quoting issues with description text
- Include the commit hash — essential for correlating log entries to code changes

---

## 4. Research Directions — Writing Good Ideas

The quality of research directions determines the quality of experiments. Follow these principles:

**Specific > vague**
- Bad: "try different attention patterns"
- Good: "try SSSSL pattern (5 short-window layers, 1 full) vs current SSSL"

**Mechanistic > intuitive**
- Bad: "try a bigger model"
- Good: "increase DEPTH from 8 to 10 while reducing ASPECT_RATIO from 64 to 52 to hold param count constant"

**Include the hypothesis**
- "Try value residual scaling of 0.5 (currently 1.0) — hypothesis: smaller residual reduces gradient variance through value pathway"

**Group related ideas**
- Grouping helps the agent see the exploration space and avoid fixating on one sub-area

**Include easy wins first**
- Hyperparameter sweeps are faster to implement and more likely to yield quick wins
- Reserve architectural changes for after hyperparameter tuning

**Periodically regenerate directions**
After a session, review `results.tsv`. Ideas that improved should spawn follow-up ideas
("SwiGLU helped — now try SwiGLU + gating scale as a learned parameter"). Add these to the next
session's `program.md`.

---

## 5. Simplicity Criterion — How to Calibrate

The simplicity criterion prevents the agent from accumulating technical debt. Without it, each
"improved" commit makes the codebase more complex, eventually becoming unmaintainable.

**The tradeoff space:**

| Improvement | Complexity Added | Decision |
|-------------|-----------------|----------|
| Any         | 0 lines removed | Always keep (simplification win) |
| > threshold | < budget lines  | Keep |
| > threshold | > budget lines  | Case-by-case (agent decides) |
| < threshold | Any             | Revert |
| 0           | Any             | Always revert |

**Calibrating the threshold:**

For `val_bpb` on a ~0.99 baseline:
- Conservative: threshold = 0.003 (0.3%), budget = 15 lines
- Moderate: threshold = 0.001 (0.1%), budget = 30 lines
- Aggressive: threshold = 0.0005 (0.05%), any complexity

For bundle size on a ~500KB baseline:
- Conservative: threshold = 5KB (1%), budget = 10 lines of config
- Moderate: threshold = 1KB (0.2%), budget = 20 lines

**Important:** Calibrate based on how many experiments you plan to run. If running 100 experiments,
a 0.001 threshold accumulates many small wins into a large total improvement. If running 10
experiments, raise the threshold so only meaningful changes survive.

---

## 6. Common Mistakes

**Vague constraints**: "don't change things that are important" → agent guesses wrong
Fix: list every frozen file and function explicitly

**No dead-ends section**: agent retries failed experiments across context resets
Fix: update dead-ends after every session, before starting the next

**Missing crash handling**: agent loops on broken code indefinitely
Fix: include explicit crash detection (empty grep), diagnosis step (tail log), and revert rule

**Over-specifying research directions**: "first try X, then Y, then Z in that exact order"
Fix: give directions as a priority-ordered pool, not a sequence; the agent picks based on context

**No simplicity criterion**: codebase accumulates complexity → becomes unmaintainable by run 50
Fix: always include explicit threshold and line budget

**Forgetting "never stop"**: agent asks "should I continue?" every few experiments
Fix: include "Never stop. Do not ask for permission to continue." in loop rules

**Forgetting the baseline run**: no reference point for measuring improvement
Fix: always include "first run = no changes, log as baseline" in loop rules
