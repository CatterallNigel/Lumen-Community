# Replay command list and quality-gate corrections

Replay now exposes an explicit operator command for discovering prepared sessions:

```text
\obt replay list
```

The command returns ready prepared sessions with their experiment names and Replay IDs, followed by the exact `start` syntax. The bare `\obt replay` command remains help rather than silently performing a list operation.

The command parser was also corrected to satisfy the project quality gates:

- removed the redundant `if`/`else` branch reported by Ruff;
- replaced `Any` annotations with typed `object` handling;
- corrected the optional text assignment reported by mypy;
- retained transparent pass-through for unknown Replay subcommands.
