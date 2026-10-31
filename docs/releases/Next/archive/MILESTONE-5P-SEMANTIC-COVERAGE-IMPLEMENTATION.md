# Milestone 5P — Semantic coverage implementation (incremental)

This package adds **explicit source-protocol generation/control semantics** to CIM via `protocol_controls[]`. Unlike `extensions[]`, these fields are recognised by name and recorded with their declared source protocol and semantic identity. The source protocol OUT Adapter restores them exactly; a different destination protocol reports an explicit semantic loss rather than guessing equivalence.

The implementation is **not** a claim of complete semantic certification. Complex multimodal, nested message structures, response-specific fields and some request controls still require purpose-built canonical semantics and round-trip fixtures. The field register explicitly distinguishes the implemented control mappings from unresolved areas. Streaming remains 5Q.

The previous `NOT_CERTIFIED` flag means that no full-coverage conformance certification has been established. It does **not** mean existing tests fail. The audit results are at `docs/architecture/adapter-protocol-coverage/NON_STREAMING_FIELD_COVERAGE_REGISTER.csv` and `NON_STREAMING_COVERAGE_STATUS.json`.
