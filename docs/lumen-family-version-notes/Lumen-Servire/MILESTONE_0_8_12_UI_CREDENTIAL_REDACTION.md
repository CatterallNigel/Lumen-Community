# Lumen Servire 0.8.12 — UI Credential Redaction

Servire must never expose credentials embedded in service connection URIs through its operator UI.

## Changes

- Connection URI user-information is removed before rendering service health values.
- The same sanitization is applied to configuration details, resolved configuration YAML, validation messages, operational log messages, availability targets, and interface URLs.
- Host, port, path, query, and fragment information remain visible for operational diagnosis.
- Runtime configuration remains unchanged; sanitization occurs only at the presentation boundary.
- `config.example.yml` uses placeholder MongoDB credentials rather than deployment credentials.

## Acceptance

- A URI such as `scheme://user:password@host:port/path` is rendered as `scheme://host:port/path`.
- Plain non-credential URLs are unchanged.
- Nested resolved configuration data is sanitized recursively before YAML rendering.
