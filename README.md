# Simple Project Documentation

This repository contains a minimal documentation setup using Sphinx and Read the Docs.

## Quick start

1. Create a virtual environment (optional):
   - Windows PowerShell: `python -m venv .venv`
2. Install documentation dependencies:
   - `pip install -r requirements.txt`
3. Build docs locally:
   - `sphinx-build -b html docs docs/_build/html`
4. Open generated docs:
   - `docs/_build/html/index.html`

## Publish on GitHub

1. Initialize git:
   - `git init`
2. Add files and commit:
   - `git add .`
   - `git commit -m "Add minimal docs for Read the Docs"`
3. Connect remote and push:
   - `git branch -M main`
   - `git remote add origin <your-repo-url>`
   - `git push -u origin main`

## Connect to Read the Docs

1. Go to [Read the Docs](https://readthedocs.org/).
2. Import your GitHub repository.
3. Ensure build config file `.readthedocs.yaml` is detected.
4. Trigger first build.

Your docs entry page is `docs/index.rst`.
Current pages included in navigation: `setup`, `guide`, and `picodingagent`.
