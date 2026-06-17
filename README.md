# UC Library Carpentry: Python (JupyterLite fallback)

A browser-based Python environment for the [Library Carpentry: Introduction to Python][lesson] lesson. It runs entirely in the browser using [JupyterLite][jupyterlite]: no install, no account, no setup. It exists as a fallback for learners who can't install Python locally (locked-down laptops, install trouble during a workshop, or wanting to start fast).

**Live site:** <http://www.tim-dennis.com/uc-lc-python-lite/>

The lesson data and a welcome notebook are bundled in, so file paths like `data/2011_circ.csv` work as-is.

---

## For learners

1. Open the live site: <http://www.tim-dennis.com/uc-lc-python-lite/>. It loads JupyterLab in your browser. First load can take a few seconds while the Python kernel downloads.
2. The `welcome.ipynb` notebook opens automatically. Run its first code cell to confirm everything works.
3. To start a fresh notebook: click **+** at the top of the file browser, then choose **Python (XPython)**.
4. Follow the lesson at <https://librarycarpentry.github.io/lc-python-intro/>. The Chicago Public Library circulation data (2011–2022) used throughout is already in the `data/` folder.

### What's different from a normal Jupyter install

* **No shell commands.** Lines starting with `!` (such as `!pip install` or `!ls`) don't work in JupyterLite. Everything you need is preinstalled.
* **Preinstalled packages:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`. If the lesson asks you to install something else, it won't be available here.
* **Your work lives in the browser.** Notebooks you create are saved to your browser's local storage, not to a server. Clearing site data wipes them. Download anything you want to keep (right-click the file → Download).

---

## For maintainers

This repo is a JupyterLite site that GitHub Actions builds and publishes to GitHub Pages on every push to `main`.

### Layout

| Path | Purpose |
|---|---|
| `content/` | Everything served to learners: `welcome.ipynb` and the `data/` files. The build flag `--contents content` points here. |
| `environment.yml` | The in-browser kernel and its packages (xeus-python + the scientific stack), resolved from emscripten-forge. |
| `requirements.txt` | The build-time tools that run in CI (`jupyterlite`, `jupyterlite-xeus`, `jupyter-server`). |
| `jupyter-lite.json` | JupyterLite config. Sets the notebook that opens on load (`welcome.ipynb`). |
| `.github/workflows/deploy.yml` | Build + deploy to Pages. |

### Build locally

```bash
pip install -r requirements.txt
jupyter lite build --contents content
# serve the result
python -m http.server -d _output 8000   # then open http://localhost:8000
```

`_output/` is git-ignored; it's a build artifact.

See [MAINTAINING.md](MAINTAINING.md) for how to update the data, change packages, edit the welcome notebook, and troubleshoot the deploy.

---

## License

The lesson content this supports is maintained by [The Carpentries][carpentries] under CC BY 4.0. The Chicago Public Library circulation data is public.

[lesson]: https://librarycarpentry.github.io/lc-python-intro/
[jupyterlite]: https://jupyterlite.readthedocs.io/en/stable/
[carpentries]: https://carpentries.org/
