# Lumen Repetere package naming standardisation

The Python implementation package has been renamed from `lumen_replay` to
`lumen_repetere` to align the implementation name with the Lumen component
vocabulary.

Use:

```text
python -m lumen_repetere
python -m lumen_repetere clear-logs
```

The distribution/CLI identity is now `lumen-repetere` and the environment
prefix is `LUMEN_REPETERE_`.

Replay remains the capability name. Existing replay API concepts, MongoDB
collection names, and protocol headers such as `x-lumen-replay-*` are retained
because they describe the replay contract rather than the Python package name.
