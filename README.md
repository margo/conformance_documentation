# Conformance Documentation
This repo shall be utilized for all content related to Conformance documentation along with Test Toolkit content. 

Current specification baseline: **v1.0.0-rc.3 (PR3)**. Start with the [overview](docs/overview/overview.md).

## Documentation site

The `docs/` folder is published with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/). The navigation is defined in [mkdocs.yml](mkdocs.yml).

Preview locally:

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Then open <http://127.0.0.1:8000>. Run `mkdocs build --strict` to check for broken links before opening a pull request.

The [Docs workflow](.github/workflows/docs.yml) builds the site on every pull request and deploys it to GitHub Pages on pushes to `main`. To enable deployment, set **Settings > Pages > Build and deployment > Source** to **GitHub Actions**.
