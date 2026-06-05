# LogiSmith Docs

[![Docs](https://img.shields.io/badge/docs-mkdocs--material-blue)](https://logismith.github.io/Docs/)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)

Source for the **LogiSmith documentation site** — user-facing guides for the
open-source FPGA toolchain: installing dependencies (Ubuntu, WSL2, Docker),
getting started and usage.

👉 **Read it here: <https://logismith.github.io/Docs/>**

> This is the *product / user* documentation. Component-level developer docs live
> with each component's code — e.g. the Anvil CLI internals at
> <https://logismith.github.io/Anvil/>.

## Local development

```bash
pip install -r docs/requirements.txt
mkdocs serve              # live preview at http://127.0.0.1:8000
mkdocs build --strict     # build + fail on broken links
```

## Structure

```
mkdocs.yml          ← site config + navigation
docs/
├── index.md        ← home
└── installation/   ← per-platform install guides (Ubuntu, WSL2, Docker)
```

Pages live in `docs/`; the navigation tree is the `nav:` section of `mkdocs.yml`.

## Deployment

Pushes to `main` that touch `docs/**` or `mkdocs.yml` trigger
[`.github/workflows/docs.yml`](.github/workflows/docs.yml), which builds the site
and publishes it to the `gh-pages` branch (served by GitHub Pages). The generated
`site/` directory is git-ignored.

## License

Released under the [MIT License](LICENSE). © 2026 LogiSmith.
