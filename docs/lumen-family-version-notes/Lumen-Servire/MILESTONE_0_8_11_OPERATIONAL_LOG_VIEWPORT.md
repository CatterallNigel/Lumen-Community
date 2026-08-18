# Lumen Servire 0.8.11 — Operational Log Viewport

## Scope

This is a presentation-only refinement to the Servire operational dashboard.

## Change

The Operational Log viewport now reserves 24rem of vertical space, providing room for at least twelve normal log rows while retaining internal vertical scrolling when more entries are present.

The change deliberately leaves the log toolbar, filters, search, pause, export and clear controls unchanged.

## Behaviour

- Empty or lightly populated logs retain the full operational viewport instead of collapsing to only the rendered rows.
- Normal operational use exposes approximately twelve or more log rows at once.
- Longer logs remain contained within the panel and scroll vertically.
- Filtering the visible entries does not collapse the log viewport.

## Acceptance

- The Servire dashboard renders `.log-view` with a fixed 24rem viewport.
- Existing operational-log behaviour and controls are unchanged.
- Existing tests remain green.
