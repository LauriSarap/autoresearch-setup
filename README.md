# autoresearch-setup

> Inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch). Original structured experiment framework by [wizwand/autoexp](https://github.com/wizwand/autoexp).

A one-line setup that turns any AI/ML research project into an autoresearch-style workflow, enabling AI coding agents (such as Codex or Claude Code) to run experiments automatically and optimize them iteratively.

Works with any research repo — training pipelines, analysis/interpretability experiments, evaluation harnesses, and more.

## One-time setup

In your project directory, send your coding agent (such as Codex or Claude Code) this message:
```
Read https://github.com/LauriSarap/autoresearch-setup/blob/main/make_autoexp.md and follow the instructions to create `autoexp_program.md` in the current directory.
```
What it does under the hood:
- Your coding agent will scan the project directory, read linked papers, and understand the research context.
- It will ask you what kind of experimentation you want to do:
  - **A) Optimize training** — tune hyperparameters and architecture to improve validation metrics
  - **B) Optimize execution** — make experiments faster without changing results
  - **C) Extend experiments** — run on new models, datasets, or configurations
  - **D) Modify experiments** — change the methodology, add metrics, test new hypotheses
  - **E) Future research** — pursue directions suggested by the paper
  - **F) Reproduce and validate** — verify the paper's claims systematically
  - **G) Custom goal** — define your own objective
- It will then create a `autoexp_program.md` file tailored to your chosen goal.

## Run automated experiments

To kick off automated experiments, send your coding agent (such as Codex or Claude Code) this message:
```
Hi, have a look at autoexp_program.md and let's kick off the experiments!
```
What it does under the hood:
- Your coding agent will read `autoexp_program.md` and understand the research context and your goal.
- It will run the baseline experiment and record the results in `autoexp_results.tsv`.
- It will then iteratively run experiments appropriate to your goal — optimizing, extending, reproducing, or exploring as specified.

## Customization

To customize the experiment setup, edit `autoexp_program.md` freely to adjust the configuration as needed.

## License

MIT
