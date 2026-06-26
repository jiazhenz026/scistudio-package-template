# Changelog

All notable changes to this package are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/) and the project uses
[Semantic Versioning](https://semver.org/).

## [Unreleased]

- OTA hot-update support (#1784): the package self-declares its update source
  via `PackageInfo.ota` (mirrored in `[tool.scistudio.ota]`), and
  `scripts/ota_publish.py` publishes `manifest.json` + a source snapshot to the
  package's own public `ota-<channel>` GitHub pre-release. The in-app SciStudio
  Package Manager checks this source and offers updates.

## [0.1.0]

- Initial example package generated from `scistudio-package-template`:
  one example type (`ExampleSeries`), one passthrough block (`ExampleBlock`),
  and an empty previewers stub.
