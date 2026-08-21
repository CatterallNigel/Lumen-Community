# Lumen Servire 0.8.13 — Masked Credential Display

Servire now preserves the authentication shape of displayed connection URIs while masking credential values.

Example:

```text
mongodb://operator:secret@host:27017/?authSource=admin
```

is rendered as:

```text
mongodb://****:******@host:27017/?authSource=admin
```

Runtime configuration is unchanged. The masking applies only at the presentation boundary.
