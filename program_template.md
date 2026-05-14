# Autonomous Experimentation Program

This document defines how an autonomous coding agent should iteratively run experiments in this repository.

## Research context

Paper: `<paper title, authors, venue, year — or "no paper linked">`
Repository purpose: `<one-sentence description of what this repo does>`
Repo type: `<training pipeline | experiment/analysis pipeline | evaluation harness | data pipeline | library | notebook-driven>`

## Goal

`<one of: optimize training | optimize execution | extend to new settings | modify experiment | pursue future research | reproduce and validate | custom>`

`<1-2 sentence description of what we're trying to achieve, in concrete terms>`

Primary metric to optimize:
- `<metric_name>`: `<higher|lower>` is better

Secondary metrics / constraints:
- `<metric_name>`: `<higher|lower>` is better
- `<constraint: e.g. numerical equivalence, stability, memory budget, coverage completeness>`
- run must complete successfully
- complexity should only increase when justified by meaningful gains

Per-run budget:
- wall clock budget: `<e.g. 10 min / 1 epoch / 5k steps / until first eval>`
- if the code already defines a fixed budget, follow it exactly
- if not, use the smallest budget that still yields a meaningful signal

Success criteria (adapt to goal type):
- `<For training: improve the primary metric>`
- `<For execution: faster + outputs match baseline within tolerance>`
- `<For extension: complete coverage of the settings grid>`
- `<For modification: produces valid, interpretable results under new methodology>`
- `<For reproduction: results match paper's reported values>`
- do not crash
- do not exceed the run budget
- prefer simpler and cleaner changes when performance is similar

## Repository scope

Read these files first for context:
- `<README / docs / primary scripts / config / evaluation / model / utils>`

Primary entrypoint:
- ``<command to run>``

Primary output / evaluation path:
- `<how results are produced / where metrics come from>`

Artifacts and logs to inspect:
- `<stdout / log file / tensorboard / wandb / checkpoints / metrics json / csv>`

Experiment workspace:
- use `exp/<experiment-name>/` for each new idea or setting
- copy or replicate only the scripts/config folders needed to run that idea
- keep baseline/canonical repo files unchanged until an experiment has clearly won
- run each experiment from its own `exp/...` workspace or with explicit paths to that workspace
- use Git commits only as provenance for the canonical state; do not create Git branches unless the user explicitly asks for branch-based experiments

## What can be modified

The agent may modify:
- `<list specific canonical files and what can change in them after promotion>`
- `exp/<experiment-name>/...` copies of scripts/configs needed for isolated experiments

## What cannot be modified

The agent must NOT:
- modify evaluation ground truth, benchmark targets, or validation data
- alter the meaning of the primary metric
- install new packages unless already present in repo dependencies
- rewrite unrelated infrastructure or application code
- bypass evaluation or hardcode outputs
- leave the repo in a broken state

Do NOT modify:
- `<list specific files or directories that are read-only>`

## Baseline run

The first experiment must always be a baseline:
1. run the pipeline exactly as-is
2. capture the full output
3. extract the primary metric and key runtime stats
4. record the result in the experiment log
5. use this baseline as the comparison point for all subsequent experiments

For **reproduction** goals, the baseline should reproduce one key result from the paper.
For **extension** goals, the baseline should run the original configuration before trying new ones.

## Output format

At the end of each run, extract or derive the following fields when available:

```text
primary_metric:     <value>
secondary_metric:   <value>
training_seconds:   <value>
total_seconds:      <value>
peak_vram_mb:       <value>
num_steps:          <value>
num_epochs:         <value>
throughput:         <value>
num_params_M:       <value>
checkpoint_path:    <value>
status:             <ok|crash|timeout|oom|nan>
```

If the script already prints a summary block, parse that.
If not, extract the fields from logs, metrics files, or evaluation outputs.
If some fields do not exist, omit them or mark them as `NA`.

## Logging results

Log every experiment to:
- `autoexp_results.tsv`

Use tab-separated format with this header:

```text
experiment_id	experiment_path	base_state	primary_metric	status	description	runtime_sec	peak_vram_mb	notes
```

For **extension** goals, add a `setting` column:
```text
experiment_id	experiment_path	setting	base_state	primary_metric	status	description	runtime_sec	peak_vram_mb	notes
```

For **reproduction** goals, add a `reference` column:
```text
experiment_id	experiment_path	base_state	reference	expected	actual	status	description	runtime_sec	notes
```

Definitions:
- `experiment_id`: short stable name, usually matching the `exp/<experiment-name>` folder
- `experiment_path`: path to the isolated experiment workspace, e.g. `exp/lr-sweep`
- `base_state`: short git commit hash or `working-tree` identifier for the canonical files copied into the experiment
- `primary_metric`: numeric value, or `NA` if unavailable
- `status`: `keep`, `discard`, `crash`, `timeout`, `oom`, `nan`, or for reproduction: `match`, `mismatch`
- `description`: short description of the idea tested
- `setting`: model/dataset/config identifier (extension goals)
- `reference`: paper table/figure number (reproduction goals)
- `expected`: value reported in paper (reproduction goals)
- `actual`: value observed (reproduction goals)
- `runtime_sec`: total runtime in seconds if available
- `peak_vram_mb`: peak memory if available
- `notes`: brief reason for keep/discard/crash, or explanation of mismatch

Do not commit `autoexp_results.tsv` unless the repo explicitly wants experiment logs tracked.

## Promoting winning experiments

When an isolated experiment beats the current best result and satisfies all constraints:
1. compare its `exp/<experiment-name>/` files against the canonical source files they were copied from
2. identify the minimal set of changes responsible for the improvement
3. apply only those changes back to the canonical repo files
4. rerun the canonical entrypoint outside `exp/...` to verify the improvement survives promotion
5. mark the experiment `keep` only after the promoted canonical run passes

If multiple isolated experiments win independently, merge them deliberately:
- combine one idea at a time into the canonical files
- rerun after each merge
- if two ideas interact badly, keep the stronger one and record the failed combination in `autoexp_results.tsv`

## Experiment report

Generate a LaTeX experiment report (`autoexp_report.tex`) and compile it to PDF (`autoexp_report.pdf`).

### When to generate

- **Optimization goals (A, B):** Generate the report **once** — when the optimization loop has finished or enough runs have been completed that diminishing returns are clear. Do not regenerate after every run.
- **All other goals (C, D, E, F, G):** Generate or update the report **after each experiment run**.

### Report structure

Write the report as a concise research-style experiment overview. Use this structure:

```latex
\documentclass{article}
\usepackage{booktabs, graphicx, hyperref, amsmath, pgfplots}
\pgfplotsset{compat=1.18}
\title{AutoExp Report: <goal description>}
\author{<GitHub username> \and <agent/model name, e.g. Claude Opus 4.6, GPT-4o, Codex>}
\date{\today}
\begin{document}
\maketitle

\section{Introduction}
% Brief context: what repo/paper this is, what goal was pursued, why.

\section{Experimental Setup}
% Entry point, hardware, key configuration, per-run budget.
% For extension/reproduction: the settings grid or claims list.

\section{Results}
% Table(s) built from autoexp_results.tsv.
% For optimization: show progression from baseline to best result.
% For extension: show results across all settings.
% For reproduction: show expected vs actual with match/mismatch.
% For modification: show baseline vs modified side by side.
%
% REQUIRED: Include at least one visual (chart/plot) of the results using pgfplots.
% Examples:
%   - Optimization: line plot of primary metric over experiments, bar chart comparing top runs.
%   - Extension: grouped bar chart across settings.
%   - Reproduction: bar chart of expected vs actual values.
%   - Modification: side-by-side comparison chart.

\section{Analysis}
% What worked, what didn't, notable patterns or anomalies.
% For optimization: which knobs had the most impact.
% For reproduction: explain any mismatches.
% Reference the visuals from the Results section in your analysis.

\section{Conclusion}
% Summary of outcome relative to the goal.

\end{document}
```

### Compilation

Compile with:
```bash
pdflatex -interaction=nonstopmode autoexp_report.tex
```

If `pdflatex` is not available, leave the `.tex` file for the user and note that compilation was skipped.

The report **must** include visual charts/plots of the results using `pgfplots` (inline in the `.tex` file). Do not rely on external image files. Every report should have at least one figure visualizing the key results alongside the data tables.

## Search space

### For training optimization (Goal A)
High-priority knobs:
- `<learning rate, batch size, scheduler, optimizer>`
- `<model width / depth / heads / dropout>`

Low-priority or risky knobs:
- `<data pipeline changes, loss changes, architecture rewrites>`

### For execution optimization (Goal B)
High-priority knobs:
- `<dtype (fp16/bf16), torch.compile, batching, caching>`
- `<eliminating redundant computation, pre-computation>`
- `<memory management, dead code removal>`

Low-priority or risky knobs:
- `<algorithmic changes that may affect numerical results>`

### For experiment extension (Goal C)
Settings grid:
- `<models: list of model names/sizes to test>`
- `<datasets: list of datasets if applicable>`
- `<configurations: list of config variations>`

### For experiment modification (Goal D)
Proposed changes:
- `<what to change about the methodology>`
- `<what new metrics or analyses to add>`
- `<what hypothesis to test>`

### For future research (Goal E)
Directions from paper:
- `<future work item 1>`
- `<future work item 2>`
- `<selected direction and rationale>`

### For reproduction (Goal F)
Claims to verify:
- `<paper claim 1 — table/figure reference — expected value>`
- `<paper claim 2 — table/figure reference — expected value>`

Known repo-specific traps:
- `<OOM risk areas>`
- `<config mismatches>`
- `<files that must remain fixed>`
- `<environment/dependency gotchas>`

## Experimentation principles

- Always make one coherent change per experiment, or one tightly-related bundle
- Prefer changes that are easy to reason about
- Prefer improvements that hold under the fixed evaluation protocol
- Prefer simplifications when performance is equal
- Leave failed or non-improving experiments isolated in `exp/...` and keep canonical files unchanged
- Record all results even for failed runs (for extension/reproduction goals)
- Keep good changes by promoting them into the canonical files and advance from the best known canonical state
- Avoid large speculative rewrites unless smaller changes are exhausted

Examples of reasonable experiments:
- `<goal-specific examples — e.g., for training: LR sweep; for extension: adding a new model; for execution: switching to fp16>`

Examples of unreasonable experiments:
- changing the validation set, evaluation logic, or metric definition to look better
- skipping evaluation or hardcoding outputs
- major refactors unrelated to the goal
- changing many unrelated things at once with no attribution

## The experiment loop

### For optimization goals (A, B)

LOOP FOREVER:

1. Inspect current best result.
2. Choose one promising experiment.
2a. Create an isolated experiment workspace at `exp/<experiment-name>/`.
2b. Copy the relevant scripts/config folders from the current best canonical state into that workspace.
3. Edit only the copied files inside `exp/<experiment-name>/`.
4. Record the base state and changed files in the experiment notes.
5. Run the experiment with output redirected to a log file:
   - ``<run command using exp/<experiment-name>/ paths> > exp/<experiment-name>/run.log 2>&1``
6. Extract the primary metric and key runtime stats.
7. If the run crashed or produced invalid results:
   - inspect the tail of the log
   - determine whether the failure is trivial to fix
   - retry only if the fix is small and the experiment idea still makes sense
8. Record the result in `autoexp_results.tsv`.
9. If the primary metric improved (and secondary constraints are met):
   - promote the minimal winning changes back into the canonical repo files
   - rerun the canonical entrypoint to verify the promoted state
   - mark status `keep`
   - make the promoted canonical files the new best state
10. If the result is worse, equal, invalid, or too costly:
    - mark status `discard` or failure status
    - leave the canonical files unchanged
11. Repeat until the budget, search space, or improvement potential is exhausted.

### For extension goals (C)

FOR EACH setting IN the settings grid:

1. Create an isolated workspace for this setting (`exp/<setting-name>/`) from the current canonical files.
2. Configure the copied pipeline files for this setting.
3. Keep the setting-specific changes inside the `exp/<setting-name>/` workspace unless the user asks to make them canonical.
4. Run the experiment.
5. Extract metrics.
6. Record in `autoexp_results.tsv` with the `setting` column filled.
7. Flag any anomalous results for user attention.
8. Do NOT discard — all runs are kept for analysis.
9. Proceed to next setting.

### For reproduction goals (F)

FOR EACH claim IN the claims list:

1. Create an isolated workspace for this claim (`exp/reproduce-<claim-ref>/`) from the current canonical files.
2. Configure the copied pipeline files to match the paper's setup for this claim.
3. Run the experiment.
4. Extract the relevant metric.
5. Compare to the expected value from the paper.
6. Record in `autoexp_results.tsv` with `reference`, `expected`, `actual` columns.
7. Mark `match` if within reasonable tolerance, `mismatch` otherwise.
8. For mismatches, investigate and document possible causes.

### For modification goals (D, E)

1. Present the proposed modification to the user before implementing.
2. Create an isolated workspace for this modification (`exp/<modification-name>/`) from the current canonical files.
3. Implement the modification in copied files inside that workspace before any runs.
4. Run baseline (original methodology) for comparison.
5. Run modified experiment.
6. Record both results.
7. Analyze the difference and report findings.

## Failure handling

Treat these as failures unless quickly fixable:
- Python exception
- OOM
- divergence / NaN / Inf
- shape mismatch
- missing files
- timeout beyond allowed budget
- metric missing or unparsable

Timeout rule:
- if a run exceeds `<timeout threshold>`, terminate it and mark as `timeout`

Crash handling:
- small typo/import/config mistakes may be fixed and retried
- fundamentally bad ideas should be logged and skipped

## Experiment isolation and reproducibility

Each distinct experiment type or goal must run in its own filesystem workspace under `exp/`. Create the workspace before the first run and do all exploratory edits inside it.

Workspace naming:
- `exp/<goal-short-name>` — e.g. `exp/lr-sweep`, `exp/extend-llama-models`, `exp/reproduce-table2`
- If multiple experiments share the same goal type but test different ideas, use separate workspaces: `exp/opt-lr`, `exp/opt-batch-size`, etc.

Rules:
- Copy only the files/folders needed to run the experiment into the workspace
- Do not mix unrelated experiments in the same workspace
- Keep a short `exp/<experiment-name>/README.md` or notes entry describing what was copied, what changed, and how to run it
- Preserve the canonical repo files as the best known state until a variant is promoted
- Avoid modifying benchmark data, evaluation targets, or unrelated infrastructure in either canonical files or experiment workspaces
- When an optimization experiment is `keep`, promote the minimal winning changes into the canonical files and rerun the canonical entrypoint before treating it as the new best state
- Do not create Git branches by default; use Git only to identify the base canonical commit/state unless the user explicitly asks for branches

## Final instruction to the agent

You are acting as an autonomous research experimenter.
Your job is to make disciplined, incremental, well-logged progress toward the selected goal.
Do not chase novelty for its own sake.
Always confirm the baseline works before trying anything new.
Log everything — failed experiments are data too.
The best outcome is one that makes clear, reliable progress while keeping the codebase clean.
