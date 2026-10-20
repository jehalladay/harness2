# 02 — Reactive Harness Definition, typed mutation algebra and federation

**Status:** proposed implementation contracts. JSON examples illustrate intent, not working exports.

## Canonical Reactive Harness Definition (RHD-v1)

Source of truth is a canonical, pinned, schema-versioned manifest plus independently content-addressed code/assets. It is NOT a whole live graph snapshot and does not embed arbitrary Python closures in portable JSON.

Proposed candidate directory:
```text
reactive/
  harness.json                         # RHD-v1 manifest (required)
  components/<identifier>.py           # registered/pinned Python implementation (optional)
  graphs/<identifier>.json             # explicit typed definitions
  policies/<identifier>.json           # behavior/selection/demand policy
  ports/<identifier>.json              # exported/imported port declarations
  fixtures/<identifier>.json           # hermetic test observations
  tsx/<identifier>.tsx                 # opt-in only when host is installed & qualified
```
Keep c1..c8 legacy tree intact. In the opt-in reactive surface, an additional candidate is a structured record, not an unbounded `scripts/` loophole. Unrecognized files and undeclared dependencies are refused with actionable diagnostics.

Example:
```json
{
  "schema": "reactive.harness-definition/v1",
  "definition_id": "coding-worker",
  "revision": "sha256:<canonical-digest>",
  "runtime": {"engine": "reactive-agents", "mode": "react-fiber/v1"},
  "graph_ref": {"artifact": "graphs/worker.json", "digest": "sha256:..."},
  "components": [
    {
      "id": "diagnostic-context",
      "implementation": "components/diagnostic_context.py",
      "digest": "sha256:...",
      "input_ports": ["build-failure", "changed-files"],
      "output_ports": ["context-block", "diagnostic-value"],
      "purity": "pure"
    }
  ],
  "context": {"view": "worker", "budget_policy": "task-relevance/v1"},
  "behavior": {"feature_plan_ref": "policies/default.json"},
  "federation": {"exports": [], "imports": []},
  "authority_requirements": [],
  "compatibility": {"min_protocol": "reactive.harness/v1"}
}
```

### Definition vs instance vs execution basis

- `HarnessDefinition`: immutable program and declared ports/behaviors.
- `HarnessInstance`: a stateful, authorized mounting of a definition with an independent identity, graph generation, component state and effect bindings.
- `ExecutionBasis`: exact task, model capabilities, source-owner snapshots, component revisions, optional imported-port receipts, current graph generation, budget, policy, runtime and provider revisions.
- `ExecutionReceipt`: actual render, model, tool, publication, recovery, provenance, cost and outcome references.
- `CandidateProposal`: preconditions + operations + predicted effect/benefit + tests; never mutates the published instance by itself.

Only a host publishes an admitted candidate; the LLM proposer merely produces candidate artifacts.

## Mutation algebra

Define a typed operation union with preconditions, write/read footprints, test declarations and a required transition class:
```text
AddComponent, ReplaceComponent, RetireComponent
AddLensBinding, ReplaceLensBinding, RemoveLensBinding
ConnectPorts, DisconnectPorts, ChangePortSchema
AddQuery, ReplaceQuery, RemoveQuery
UpdateContextView, UpdateBudgetSelection, UpdateContextPriority
UpdateAgentFeature, UpdateBehaviorPolicy, UpdateModelDemandPolicy
AddFederatedExport, BindFederatedImport, RevokeFederatedBinding
UpdateRuntimeProfile (restricted to permitted resource limits; no authority minting)
```

Mutations carry `base_definition_digest`, `expected_graph_generation`, `expected_port_contracts`, stable `operation_id`, `effect_class`, `scope`, `required_tests`, and optional `compensation_plan`. Model explanations and predicted benefits are advisory; validator results and execution receipts are authoritative.

The typed result is one of:
```text
PREPARED  # static/schema/graph/port checks passed against a pinned base
REFUSED   # precise failure, nothing published
ADMITTED  # eligible for isolated candidate runtime experiment
TESTED    # tests ran; evidence may be complete/incomplete
PUBLISHED # host CAS of accepted revision + ownership/attempt fence
RETIRED   # intentional revocation/cleanup with dependent accounting
```
An experimental candidate never mutates the current shared graph. Publishing requires exact generation/base match and admission controls; stale proposals fail. In-flight model/tool attempts remain owned by their prior runtime generation, with explicit retirement/migration outcomes. Retaining candidate artifacts in Harness²'s library is not equivalent to publishing them to an active agent.

### Composition rules

Two transformations are eligible for reordering/parallel execution only after proving or testing their read/write footprint independence, graph and port compatibility, state independence and effect noninteraction. Typed edge composition checks port schema, direction, multiplicity, dependency completeness, and cycles. Cyclic reactivity may be admitted only under an explicit delayed/fixed-point semantics and separate conformance; reject ordinary instantaneous cross-graph cycles.

Permit component reuse by immutable definition ref; sharing an implementation artifact never shares live hook state unless the host explicitly owns such a service. Compile and cache valid fragments under semantically complete keys (definition, authority, source basis, state and relevant policy). A digest is an identity witness, not a proof of observational equivalence.

### Validation ladder

```text
parse + canonicalize
  -> schema + referential closure
  -> source authority/effect scope
  -> typed port + graph topology
  -> component import/ABI and pure/effect boundaries
  -> frozen-source reference evaluation
  -> optional isolated live candidate run
  -> independent held-out assessment
  -> controlled publication (separate from library adoption)
```

Minimum falsifiers: undeclared file edit, malformed type, wrong base digest, source revision mismatch, unknown dynamic dependency, effect during render, graph cycle, publishing over in-flight attempt, invalid TSX host, reused mutable state, unverified replay closure.

## Host-independent federated public ports

Keep the previously approved federation model: **logical composition of independent graphs**. Native agents and RF-CLI workspaces may export/import typed public observations; FiberOptics sees each admitted imported value as a local source. The importing host does not read private remote graph internals.

A port contract includes:
```text
owner_id, owner_instance_id, export_id, resource_kind
definition_digest, value_schema_digest, direction, operations
sensitivity, scopes, protocol_range, supported_consistency
dependency_summary_contract, optional completeness_contract
```
An accepted observation adds:
```text
source_revision, stream_epoch, owner_cursor, observation_digest,
readiness_status, value/ref, provenance_refs, coverage_refs,
accepted_consumer_commit, consumer_policy_revision
```
Status is not just a nullable value: distinguish READY, PENDING, UNAVAILABLE, WITHHELD, INVALID and STALE, plus query evidence TRUE/FALSE/UNKNOWN where QuerySDK is enabled. Equal payload strings are not interchangeable if a consumer can observe differences in status, provenance, coverage, origin or source revision.

**Allowed operations:** `describe`, `read`, `subscribe`; separately admitted `evaluate` / `invoke`. A `read` grant cannot imply an effectful `evaluate`. Two owners maintain independent clocks/revisions. `PINNED` and `LATEST_ACCEPTED` must be represented honestly; causal cross-source cuts require a separate checker. Historical replay uses retained accepted observations and never reopens remote authority.

### Smallest useful federation vertical cut

1. Python SDK-native graph B publishes a declared, immutable typed value (with owner instance and revision).
2. SDK-native graph A explicitly imports and admits its snapshot; local ordinary Fiber source drives a context component.
3. Test equal output/provenance against the expected cold graph semantics and test loss/revocation cases.
4. Add snapshot+cursor and subscribe only after the cold/accepted path works; atomically commit imported observation and accepted event cursor, detect epoch changes/gaps and resynchronize before reporting fresh.
5. RF-CLI adapter remains **optional and separately gated** until its federation host design is actually implemented and installed.
6. Mixed Python/TSX consumption follows its own host-parity gate; do not require TSX just to run native Python agents.

Owner-specific linking/auth, retention and transport are adapter responsibilities. The portable contract does not require a CLI workspace, network server, Smithy compiler or MCP session. ReactiveMCP can expose approved resources through agent-facing tools but is not the default internal graph transport.

### Formal/research obligations, not automatically satisfied by examples

Reuse the existing FROE/CFRR conditions: observer-relative snapshot substitution, typed status/provenance equality, stateful refinement, dependency-complete invalidation, causal consistency and publication receipts. Preserve independently authoritative source identities. For a finite pure acyclic graph under identical admitted observations, test exact cold vs imported execution equality; stronger asynchronous/effectful/distributed claims require additional witnesses and counterexamples. Document what has actually been proven in mathematics vs what has been implemented and measured.

## Deferred authoring formats

Native typed Python/JSON authoring is P0. Smithy-authored port declarations remain a future optional front-end to the canonical port schema, but **no SmithyWorkshopCompiler interaction, compiled trait support, source dependency or implementation work is part of this program**.
