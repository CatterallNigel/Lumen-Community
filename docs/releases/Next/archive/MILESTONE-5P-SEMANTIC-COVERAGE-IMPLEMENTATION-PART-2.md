# Milestone 5P — Semantic Coverage Implementation, Part 2

Status: **IN PROGRESS — NOT CERTIFIED**

## Implemented in this increment

- OpenAI Chat Completions `messages[].content[].image_url.detail` is now a **declared supported field** (auto, low, high, original), explicitly mapped to `CIM.messages[].content[].attributes.image_detail` and materialised in both OpenAI OUT image branches.
- New same-protocol conformance fixture demonstrates exact round-trip of text-plus-image URL and its detail control, without placing that known field in opaque `extensions`.
- Existing non-streaming Adapter behaviour remains unchanged for other mappings.

## Verification

- `PYTHONPATH=src pytest -q`: **503 passed**.
- `python -m compileall -q src tests`: passed.
- Ruff and mypy unavailable in this environment; run locally.

## Coverage work remaining

This increment does **not** certify the declared OpenAI or Anthropic non-streaming profiles. The remaining known schema/mapping gaps include additional nested message variants, audio/video/document content where applicable, richer Anthropic content blocks, provider-specific usage/response metadata, and complete per-field IN/OUT conformance. Existing protocol_controls are source-qualified preservation of known control values, not proof of shared cross-protocol semantics. Opaque extensions are not a substitute for explicit mappings.

## Acceptance criterion

For every field in the declared supported non-streaming profiles, prove semantic preservation across IN and same-protocol OUT with fixtures. Record unsupported variants and cross-protocol representational losses explicitly. Do not declare 5P complete on the strength of the test count alone.
