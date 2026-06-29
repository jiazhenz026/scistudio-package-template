# scistudio-package-template

A GitHub **template repository** for building a new
[SciStudio](https://github.com/jiazhenz026/SciStudio) package. It ships a
minimal but complete example package plus the governance every package
should have: CI (lint, type, test, wheel build, contract check), an
`AGENTS.md` + PR checklist, and a documentation standard.

The conventions follow `scistudio-blocks-spectroscopy`, the reference package.

## Use this template

1. On GitHub, click **Use this template → Create a new repository**. Name it
   `scistudio-blocks-<domain>`.
2. Add the repository secret **`SCISTUDIO_CORE_TOKEN`** (Settings → Secrets and
   variables → Actions). It needs read access to the private `scistudio` core
   repo so CI can install it. Until core is on PyPI, every block repo needs
   this secret.
3. Rename the package to your domain:
   - `src/scistudio_package_example/` → `src/scistudio_blocks_<domain>/`
   - In `pyproject.toml`: `[project].name`, the three `[project.entry-points...]`
     references, `[tool.hatch.build.targets.wheel].packages`, and
     `known-first-party`.
   - Update `__init__.py` imports and `PackageInfo`.
4. Replace the example type/block/previewer with your own, and **fill in the
   MUST skeletons**: `ExampleSeries.from_arrays(...)` and `describe_public_api()`
   raise `NotImplementedError` until you implement them (ADR-052 §13.3), so a
   half-finished package fails loudly instead of shipping a partial contract.
5. Fill in `README.md`, `docs/package-overview.md`, and `CHANGELOG.md` to
   `docs/DOCUMENTATION-STANDARD.md`.

## What's inside

```
.
├── AGENTS.md                       # contributor + AI-agent rules (lightweight)
├── CONTRIBUTING.md                 # dev setup, local checks, release
├── hatch_build.py                  # stages _scistudio_docs before wheel build
├── LICENSE                         # MIT
├── mkdocs.yml                      # generated API reference (mkdocstrings/griffe)
├── pyproject.toml                  # hatchling + ruff/mypy/pytest + entry points
├── .github/
│   ├── CODEOWNERS                  # owner-review for the frozen contract surface
│   ├── workflows/ci.yml            # lint · type · test · contract · freeze · docs · wheel
│   └── pull_request_template.md    # the gate is this checklist
├── docs/
│   ├── DOCUMENTATION-STANDARD.md   # what every package's docs must contain
│   ├── package-overview.md         # fill-in catalog (incl. the §13.1 table)
│   ├── reference.md                # generated API reference page
│   └── ui-style-guide.md           # make a previewer/panel UI look like SciStudio
├── scripts/
│   ├── build_package_docs.py       # builds wheel-bundled docs for project injection
│   ├── validate_contract.py        # entry-point + registry + §13.1 reuse-surface check
│   └── snapshot_api.py             # compute/freeze the public surface (ADR-052 §15)
├── src/scistudio_package_example/   # example: 1 type (+ contract skeletons), 1 block, previewers stub
└── tests/                          # packaging · contract · developer-contract · freeze · block tests
```

## Governance in one breath

- No gate ledger, no multi-step workflow. The gate is the PR checklist
  (`.github/pull_request_template.md`) enforced by CI.
- Every PR closes an issue, adds/updates tests for behavior changes, and keeps
  docs to the standard.
- `python scripts/validate_contract.py` + `scistudio blocks` prove the package
  still installs into core. CI runs both.
- `python -m build` stages `src/<module>/_scistudio_docs/` and includes it in
  the wheel, so SciStudio core can inject installed package docs into projects.

## The developer-facing contract (ADR-052 §13)

A package exposes two surfaces: a **registration surface** to core (the three
entry points) and a **reuse surface** to authors — the types, their
constructors, and the inherited accessors someone imports to write their own
block, plot, or script. This template makes the reuse surface
**self-enforcing**, so a scaffolded package is correct-by-construction:

- **MUST members ship as skeletons that raise.** `ExampleSeries.from_arrays(...)`
  (the domain-native constructor, on the type) and `describe_public_api()` (the
  discovery hook) raise `NotImplementedError` until you implement them.
- **SHOULD members ship as empty placeholders.** `helpers.py` is the home for
  optional public cross-type helpers — fill it in or leave it empty.
- **Every public symbol carries a stability marker.** `@stable` / `@provisional`
  + a `Since` against *this package's* version line (`scistudio.stability`,
  ADR-052 §5), read back by the contract check and the generated API reference.
- **`scripts/validate_contract.py` enforces it.** Beyond the entry-point
  handshake it checks the §13.1 reuse surface: public types at the top level, no
  `to_pandas` / `to_numpy` shadowing, no underscore-named author-facing helpers,
  and a stability marker on every public symbol. CI runs it.

The authoritative contract is the core spec
(`docs/specs/adr-052-public-api-surface.md` §13) and ADR-052; this template
models it. Your package transcribes its own §13.1 table into
`docs/package-overview.md`.

## Wheel-bundled docs

During wheel builds, Hatch runs `hatch_build.py`, which calls
`scripts/build_package_docs.py`. The script generates a package-local
`_scistudio_docs/` bundle containing:

- `manifest.json` with package name, version, slug, and description.
- `agent-reference/README.md` for embedded agents.
- `api-reference/index.md`, `mkdocs-reference.md`, `package-overview.md`, and
  the frozen public surface snapshot.
- `user-guide/` copies of the README and package docs.

SciStudio core discovers this directory from installed packages and copies it
into each project's package reference docs. The generated directory is ignored
in git; the wheel build recreates it.

## Develop the example locally

```bash
pip install "scistudio @ git+https://github.com/jiazhenz026/SciStudio.git@main"
pip install -e ".[dev]"
pytest
scistudio blocks   # the example block should appear
```
