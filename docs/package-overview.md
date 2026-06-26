# Package Overview — <package name>

> Fill this in for your package. This is the structured catalog required by
> `docs/DOCUMENTATION-STANDARD.md`. Keep it in sync with the code: the blocks
> listed here must match `get_blocks()` and the `README.md` block table.

## Purpose

<One paragraph: what scientific domain / task this package serves.>

## Scope and non-goals

- In scope: <...>
- Out of scope: <...> (and any sibling package this must not import)

## Data types

| Type | Core base | Represents | Key metadata |
| --- | --- | --- | --- |
| `ExampleSeries` | `Series` | A 1-D series of values | `unit`, `source_file` |

## Blocks

| Group | Block | Inputs → Outputs | Parameters | Notes |
| --- | --- | --- | --- | --- |
| example | `ExampleBlock` | `Collection[ExampleSeries]` → `Collection[ExampleSeries]` | — | Passthrough; replace with real logic |

## IO / format support (ADR-043)

<Only if the package has IO blocks. List advertised vs. deferred formats and
their `FormatCapability` records. Otherwise: "No IO blocks.">

## Previewers (ADR-048)

<Only if the package ships previewers. List each previewer, the type it
targets, and its capabilities. Otherwise: "No previewers.">

## Compatibility

- Requires `scistudio>=<floor>`.
- Python `>=3.11`.

## OTA hot-update (#1784)

This package supports SciStudio's in-app Package Manager hot-update. The update
source is declared once in `[tool.scistudio.ota]` (publish-time source of truth)
and mirrored in `PackageInfo.ota`; `scripts/validate_contract.py` enforces they
match.

To publish a new version:

1. Bump `[project].version` in `pyproject.toml` (and `__version__`).
2. Run `python scripts/ota_publish.py --notes "<what changed>"` (needs `gh`
   authenticated with write access). Use `--dry-run` to build locally first.

This packs `src/` into a snapshot, hashes it, writes `manifest.json`, and
uploads both to the package's own public `ota-<channel>` rolling pre-release —
the exact URL declared in `[tool.scistudio.ota].manifest_url`. Clients running
core `>= min_core_base` then see the update in the Package Manager.
