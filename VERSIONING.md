# API Versioning

Current version: 1.0.0

The Access Gate API uses semantic versioning.

- MAJOR: breaking changes, such as removing fields, renaming fields, changing enum meanings, or changing response formats.
- MINOR: backward-compatible changes, such as adding optional fields, new endpoints, or new examples.
- PATCH: documentation fixes, example corrections, or non-breaking schema clarifications.

Breaking changes must be negotiated between Core Business and Access Gate before release. The agreed contract version is recorded in `openapi.yaml` under `info.version`.
