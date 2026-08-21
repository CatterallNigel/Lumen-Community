# Lumen Vestigare package rename

The Python implementation package has been standardised from `lumen_trace` to `lumen_vestigare`.

Use:

```text
python -m lumen_vestigare
python -m lumen_vestigare clear-logs
```

The recording capability contract remains `trace`: `/trace/...` HTTP routes and the existing `trace_recordings` / `trace_messages` MongoDB collections are unchanged.
