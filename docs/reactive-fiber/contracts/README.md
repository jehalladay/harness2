# Proposed harness-protocol schema fixtures

These are **draft authoring and experimental interchange schemas**, not an assertion that runtime APIs or matching SDK implementations ship. They are owned by the Harness² integration and must be reconciled with actual SDK schemas and cross-language canonical codec before becoming normative. `sha256:` digests are placeholders in prose; machine-readable fixtures use full digests.

- `harness-definition-v1.schema.json`: content-addressed *definition*, not a mutable live graph or capability grant.
- `candidate-patch-v1.schema.json`: bounded typed patch with base revision, stated hypothesis and validation profile.
- `context-projection-v1.schema.json`: a model-visible projection and diagnostics, source basis and provenance receipt.
- `experiment-evidence-v1.schema.json`: public-to-improver non-grader run evidence, not a held-out grading payload.
- `example-candidate.json`: syntactically valid candidate-patch example.

Before using as a runtime contract: generate positive/negative vectors, canonical JSON digests across Python/TypeScript, SDK-installed validation, old-candidate migration/unknown-major refusal and independent reference checks. **No automatic compilation of arbitrary Python source from manifest input.**

The source of truth for future federated-port contracts is the Reactive Fiber SDK program and FROE/CFRR, not these Harness²-specific envelopes. Scrawl and visual input contracts belong to ReactiveFiberDevTools.
