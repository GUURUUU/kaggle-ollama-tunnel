# Contributing to kaggle-ollama-tunnel

Thanks for the interest! This repo is small on purpose: one notebook cell, one job.
Contributions that keep it simple are the best ones.

## Project layout

- `kaggle-ollama-tunnel.ipynb` — the entire product. One code cell, self-contained.
- `docs/` — demo GIF + social preview image (regenerate via scripts in the repo's history or open an issue).
- `.github/` — CI + issue templates.

## How to propose a change

1. **Issues first.** Open an issue describing the problem/idea before a big PR.
   Look for the `good first issue` label if it's your first contribution.
2. **For notebook changes:** edit the notebook (Jupyter, VS Code, or nbformat script),
   strip outputs before committing (`Notebook > Clear Outputs`, or `jupyter nbconvert --clear-output`).
3. **For CI-relevant changes:** run the checks locally:

```bash
pip install nbformat
python -c "import nbformat; nbformat.validate(nbformat.read('kaggle-ollama-tunnel.ipynb', as_version=4))"
```

4. Keep PRs single-purpose. Explain *why*, not just *what*.
5. CI must be green (`validate` + `syntax-check` + secret scan).

## Ground rules

- No model outputs, tunnel URLs, or tokens in commits — CI scans for them.
- Don't hard-code API keys; the whole point is that Ollama is keyless.
- New contributors get their PR reviewed within **24 hours**.

## Ideas that need owners

See the repo's issue tracker — anything labeled `good first issue` has a write-up meant to get you from zero to PR.
