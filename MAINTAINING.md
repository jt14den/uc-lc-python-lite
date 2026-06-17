# Maintaining this JupyterLite site

How to update content, change packages, and keep the deploy healthy. For an overview and learner-facing notes, see [README.md](README.md).

## How the deploy works

`.github/workflows/deploy.yml` runs on every push to `main` (and on manual dispatch). It:

1. Installs the build tools from `requirements.txt`.
2. Runs `jupyter lite build --contents content`, which produces a static site in `_output/`.
3. Uploads `_output/` as a Pages artifact and deploys it.

There's no separate `gh-pages` branch. Deployment goes straight to GitHub Pages from the artifact. Pages must be set to **Source: GitHub Actions** in the repo settings (Settings → Pages). The custom domain is configured there too.

Most pushes don't need a local build; commit the change and let CI publish. Build locally only when you want to preview before pushing.

## Update the lesson data

The data lives in `content/data/` and ships to learners as-is. To change it:

1. Add, replace, or remove files in `content/data/`.
2. If you rename a file, update any code in `content/welcome.ipynb` (and tell instructors, since lesson paths are hardcoded like `data/2011_circ.csv`).
3. Commit and push. CI rebuilds and redeploys.

Keep the data small. Everything in `content/` is downloaded into the learner's browser on first load, so large files mean slow starts. The `.pkl` files (`all_years.pkl`, `df_long.pkl`) are pre-built lesson artifacts. Regenerate them from the CSVs if the underlying data changes, rather than editing by hand.

## Change the preinstalled packages

Two separate files control two separate environments. This trips people up:

* **`environment.yml`**: the **in-browser kernel** (what learners' code runs against). This is the one to edit when learners need a new package like `scikit-learn`. Packages must exist for the Emscripten/WASM target, so they come from `emscripten-forge` (`https://prefix.dev/emscripten-forge-4x`) with `conda-forge` as fallback. Not every PyPI package is available for WASM, so check emscripten-forge before promising a package.
* **`requirements.txt`**: the **build tools** that run in CI (`jupyterlite`, `jupyterlite-xeus`, `jupyter-server`). Edit this only to bump the JupyterLite toolchain itself.

After editing `environment.yml`, push and confirm the new package imports in the deployed welcome notebook.

## Edit the welcome notebook

`content/welcome.ipynb` is the first thing learners see (set via `defaultUrl` in `jupyter-lite.json`). Keep it short: confirm the environment works, point at the lesson, and flag the JupyterLite limitations (no `!` shell commands). Test any code cell against the deployed site, not a local desktop Jupyter, since the WASM kernel behaves differently.

## Build and preview locally

```bash
pip install -r requirements.txt
jupyter lite build --contents content
python -m http.server -d _output 8000   # open http://localhost:8000
```

To start clean, delete the build cache and output:

```bash
rm -rf _output .jupyterlite.doit.db cache
```

All three are git-ignored.

## Troubleshooting

* **A package won't import in the browser:** it's probably not built for WASM. Confirm it's on emscripten-forge; if not, it can't be added to the kernel.
* **Deploy succeeded but the site is stale:** hard-refresh; JupyterLite caches aggressively in the browser and service worker. Try a private window to rule out cache.
* **Build fails in CI:** check the Actions log for the `jupyter lite build` step. A bad notebook JSON or a package that doesn't resolve for the Emscripten target are the usual causes.
* **Learner work disappeared:** expected. Notebooks save to browser local storage, not a server. There's no recovery; remind learners to download files they want to keep.

## Pointing a different lesson at this

This is wired specifically for `lc-python-intro` (data + welcome notebook). To reuse it for another lesson, fork it, swap `content/`, update `environment.yml` for that lesson's packages, and adjust `defaultUrl` in `jupyter-lite.json` if the welcome notebook is renamed.
