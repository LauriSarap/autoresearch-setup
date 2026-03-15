# Prompt to Generate a Repo-Specific `autoexp_program.md`

You are in a research repository. Your task is to create a repo-specific `autoexp_program.md` for autonomous experimentation.

This is a **two-phase process**: first discover what the repo does and ask the user what they want, then generate the program.

## Phase 1: Discovery and Goal Selection

### Step 1: Read the repository

Inspect at least:
- README and docs
- Any linked or referenced papers (read the paper if a URL or PDF is available)
- Training / experiment / analysis scripts
- Model files and config files
- Evaluation and metrics code
- Notebooks (read cell contents, not just filenames)
- Logging / checkpoint code
- Shell scripts / Makefile / pyproject / requirements
- CLI argument definitions

### Step 2: Classify the repository

Determine what kind of repo this is. It may be one or a combination of:

- **Training pipeline** — trains or fine-tunes models (has optimizer, loss, training loop)
- **Experiment/analysis pipeline** — runs experiments on pre-trained models (interventions, probing, interpretability, benchmarks)
- **Evaluation harness** — evaluates models on benchmarks or tasks
- **Data processing pipeline** — processes, generates, or transforms datasets
- **Library/toolkit** — provides reusable components for other projects
- **Notebook-driven research** — primary work is in Jupyter notebooks with inline results

### Step 3: Identify the research context

If a paper is linked or referenced:
- What are the paper's main claims and findings?
- What experiments were run to support those claims?
- What does the paper suggest as future work or limitations?
- What models, datasets, and metrics does the paper use?

If no paper is linked:
- What is the repo's stated purpose?
- What does it measure or produce?

### Step 4: Present findings and ask the user to choose a goal

**Do not generate the program yet.** Instead, present a brief summary of what you found and ask the user which experimentation goal they want to pursue. Offer these options (and any others that seem relevant given the repo):

**A) Optimize training pipeline** — improve validation metrics by tuning hyperparameters, architecture, optimizer, etc.
   *(Only applicable if the repo has a training loop)*

**B) Optimize execution performance** — make existing experiments run faster or use less memory without changing results (throughput, VRAM, precision, compilation, batching, etc.)

**C) Extend experiments to new settings** — run the existing experiment methodology on new models, datasets, scales, or configurations not covered in the original work.

**D) Modify or improve the experiment itself** — change the experimental methodology: add new metrics, try different intervention types, add new analysis, test new hypotheses using the existing codebase as a starting point.

**E) Pursue suggested future research** — follow up on limitations, open questions, or future work directions mentioned in the paper or README.

**F) Reproduce and validate** — systematically reproduce the paper's key results, verify claims, or test robustness of findings.

**G) Custom goal** — the user describes their own objective.

If multiple goals apply, the user may combine them (e.g., "C then B" = extend to new models, then optimize the runs).

**Wait for the user's response before proceeding to Phase 2.**

## Phase 2: Generate the program

Use the template at https://github.com/LauriSarap/autoresearch-setup/blob/main/program_template.md as the structural foundation, but adapt it based on the selected goal and repo type.

### Adaptation by goal type

#### Goal A: Optimize training
Follow the template closely — it was designed for this case. Key sections:
- Primary metric = validation metric (accuracy, loss, F1, etc.)
- Search space = hyperparameters, architecture knobs, optimizer settings
- Baseline = run training as-is
- Keep/discard = based on metric improvement

#### Goal B: Optimize execution
- Primary metric = throughput (tokens/sec, samples/sec, wall-clock time)
- Secondary constraint = **numerical equivalence** — outputs must match baseline within tolerance
- Search space = dtype, compilation, batching, caching, memory management, dead code elimination
- Baseline = run experiment as-is, measure time and validate outputs
- Keep/discard = faster + still correct

#### Goal C: Extend to new settings
- Primary metric = depends on what the original experiment measures (keep the same metric)
- The experiment loop is **not keep/discard** — it's a **coverage sweep**: run on each new setting and record all results
- Define the settings grid explicitly (which models, which datasets, which scales)
- Baseline = reproduce one original result first, to confirm the pipeline works
- Log format should include the setting (model name, dataset, etc.) as a column
- The agent should flag unexpected results (e.g., a model that behaves very differently) for user attention

#### Goal D: Modify the experiment
- Present the proposed modification clearly before implementing
- Define what changes and what stays fixed
- Baseline = run the original experiment for comparison
- Primary metric = depends on the modification (may be a new metric the user defines)
- The agent should commit the modification separately from runs, so it can be reviewed

#### Goal E: Future research
- The agent should extract the specific future work suggestions from the paper
- Present them to the user and ask which to pursue
- The program structure depends on what's chosen — may be A, B, C, or D in practice
- Document the connection to the paper's claims explicitly

#### Goal F: Reproduce and validate
- The experiment loop is a **checklist**: reproduce each key claim/figure from the paper
- Primary metric = match between reproduced result and reported result
- Log format should reference the paper's table/figure number
- The agent should report discrepancies, not try to fix them

#### Experiment report (all goals)

The generated program must include an "Experiment report" section instructing the agent to produce a LaTeX report (`autoexp_report.tex`) compiled to PDF. The timing depends on the goal:

- **Optimization goals (A, B):** Generate the report once at the end — when the optimization loop finishes or diminishing returns are clear. Not after every run.
- **All other goals (C, D, E, F, G):** Generate or update the report after each experiment run.

The report should be a concise, research-paper-style experiment overview with sections for introduction, setup, results (tables from `autoexp_results.tsv`), analysis, and conclusion. Use the structure from the template.

### General adaptation rules (all goals)

- If the repo prints a metrics summary, document that exact format
- If the repo writes metrics to JSON/CSV/TensorBoard/W&B/local files, define the extraction procedure clearly
- If the repo has multiple entry points, choose the most relevant one for the selected goal and explain why
- If the repo uses configs, specify which keys are tunable for this goal
- If no explicit time budget exists, define a practical per-run budget based on existing structure
- If the repo includes benchmark/eval harness files, treat them as read-only unless the selected goal explicitly requires changing them
- If some details are ambiguous, document assumptions instead of leaving placeholders

### Requirements for the generated `autoexp_program.md`

- It must be concrete, not generic — name actual files, commands, metrics from this repo
- It must define a baseline run first (even for non-training goals)
- It must define a clear experiment loop appropriate to the selected goal
- It must specify how to log results into `autoexp_results.tsv`
- It must define the primary metric and any secondary constraints
- It must explain what the agent can and cannot modify
- It must preserve the integrity of whatever the experiment measures
- It must avoid adding dependencies unless already supported
- It must prefer small, attributable experiments over chaotic rewrites
- It must be specific enough that a coding agent can follow it immediately
- It must include an experiment report section that produces a `.tex` and `.pdf` overview of results

### Output requirements

- Write only the final `autoexp_program.md`
- Do not output commentary outside the file
- Do not leave placeholder text like `<fill me>`
- If the selected goal requires new sections not in the template, add them
- If the selected goal makes template sections irrelevant, omit them
