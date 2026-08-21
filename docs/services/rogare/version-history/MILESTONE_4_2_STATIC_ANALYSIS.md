# Lumen Rogare M4.2 — Static Analysis Cleanup

This maintenance release keeps the M4.1 backchannel behaviour unchanged.

Changes:

- Excludes the local `.versions` snapshot directory from Ruff checks.
- Marks the embedded HTML/CSS/JavaScript UI template as exempt from Ruff E501 line-length checks.
  The template is intentionally embedded verbatim and formatting it solely for Python line length adds no runtime value.
- No backchannel, Pontis, session, or UI behaviour changes.
