# 07 — Visual feedback, annotation evidence and heterogeneous causal correlation

**Status:** proposed addendum to the Harness² reactive evolution program, 2026-10-20. Implement through ReactiveFiberDevTools; no Scrawl or other capture-tool dependency in core Harness², FiberOptics, ReactiveAgentsSDK, OpenCode or the bridge.

## Thesis

A user-visible failure may be missed by tests, and a passing screenshot may conceal stale or incorrect reactive state. Combine three evidence classes without conflating them:
1. **Human visual reports:** annotations, spoken comments, selected screen regions and interaction sequence; a hypothesis or acceptance criterion, not automatically verified runtime truth.
2. **Source-authored runtime evidence:** Fiber transactions, graph generation, source/owner revisions, render inclusion, model/tool effects and recorded state.
3. **Controlled experiment outcomes:** baseline/candidate comparisons, independent task and visual acceptance checks with completeness/uncertainty and resource costs.

Scrawl is the **first external provider adapter**, not a new subsystem. Other screenshot/annotation providers and native TSX/TUI annotation emitters must use the same canonical observations, coordinate transforms and evidence-link contracts.

## Scrawl inspection baseline

Scrawl repo: https://github.com/kitlangton/scrawl. Its bundle reader/skill documents `scrawl.json`, `contact-sheet.png`, `frames/`, original optional video/audio, keyframes, marks, click/drag events, timed transcript, app context and retention flags. `scrawl inspect` and `scrawl zoom` can help when installed; offline manifest/images must still work. `agentContext` is an opt-in experiment; on supported hosts it can inspect accessibility or instrumented web element labels and source metadata. The feature is *not always active*. UI/React source metadata is **not** a FiberOptics component identity.

Do not depend on Scrawl internals or write into its bundle. Parse a pinned, validated manifest subset and retain opaque raw references to fields a version does not understand. No automated upload or user capture service is required. A Scrawl bundle may refer to missing media; represent that explicitly rather than inventing a frame.

## Source-neutral VisualEvidenceProvider

Proposed interface in DevTools, not an SDK runtime interface:

```ts
interface VisualEvidenceProvider {
  inspect(source: string): Promise<VisualManifest>
  observations(source: string, selection?: Span): AsyncIterable<VisualObservation>
  artifacts(source: string, selection?: Span): Promise<EvidenceArtifactRef[]>
  expand(source: string, selection: Span): Promise<EvidenceArtifactRef[]>
}
```

Manifest: provider+version, capture id/digest, source epoch, original/derived artifact refs, local time-origin definition, frame geometry and crop transforms, event/media coverage, redaction and retention. Observation: immutable id, capture-relative time interval and uncertainty, annotation kind/geometry/author or transcript reference, target hint, original artifact digest and projection descriptor. Work with frames/transcript independently from source video where available.

Data-plane operations are bounded and read-only; "expand" may derive crops/frame references without modifying the underlying capture. A decoder cannot use a caption or LLM guess to claim direct instrumentation.

## Three grades of UI ↔ Fiber linkage

- **DIRECT:** source app explicitly emits a trustworthy identity mapping `ui_component_ref → fiber_port_or_component_ref`, plus graph/renderer revision and source.
- **DERIVED:** deterministic adapter mapping under a validated source map or declared `data-fiber-ref` contract; include mapping rule, source revision and conflicts.
- **HEURISTIC:** matched labels, geometry, transcript or model interpretation. Never upgrade to DIRECT without new producer-authored evidence.

Each link records `relation` (e.g., `describes`, `rendered_by`, `possibly_describes`, `caused_by` only with causal witness), basis refs and verification status. A visual symptom is not proof of runtime cause. React's own internal fiber identifiers are not automatically equivalent to FiberOptics ports.

## Timeline and coordinate reconciliation

Keep source capture clock, UI event clock and Fiber runtime epoch distinct. Explicit anchors (e.g., app-emitted transaction+frame marker, UI revision, capture event callback) can estimate affine mapping `t_runtime ≈ a*t_capture+b` with uncertainty interval; clocks without common anchors allow *proximity* links only. A mere wall-clock coincidence never proves a data dependency or causal edge. Convert cropped-screen normalized coordinates to original-window coordinates using versioned transforms and test off-by-one/resize/HiDPI cases. Silence between frames isn't proof that nothing changed.

## Evidence graph representation

DevTools creates a typed observational graph containing:
- visual capture/artifact/keyframe/annotation/utterance/interaction nodes;
- authoritative graph/component/port/transaction/render/source revisions by reference;
- experiment/run/candidate and verified-outcome nodes;
- typed links with source basis, confidence class, completeness and status.

This is an **analysis graph** and does not supplant FiberOptics scheduling, ReactiveQuerySDK evidence truth or source-owner admissions. A recorded trace with dropped events cannot become a proof of missing renders.

Sketch:

```json
{
  "schema": "reactive.visual-evidence/v1",
  "id": "visual:annotation:7",
  "provider": "scrawl",
  "capture_ref": {"digest":"sha256:...","manifest_version":"..."},
  "interval_ms": [11800,12400],
  "annotation": {"kind":"human_mark","geometry_ref":"mark:7","utterance_ref":"words:42-58"},
  "links": [{
    "target_kind":"fiber_component",
    "target_ref":"agent.build-status",
    "relation":"possibly_describes",
    "basis":"label-and-time",
    "confidence_class":"HEURISTIC",
    "status":"unverified"
  }],
  "evidence_status":"PARTIAL"
}
```

## Visual LX and QA

Proposed headless DevTools commands:
- `rfd visual import bundle.scrawl`, `inspect`, `expand --span`, `correlate --trace`.
- `rfd qa visual compare --baseline ... --candidate ...` reports alignment, changed regions and oracle constraints, never asserts visual success from pixels alone.
- `rfd qa visual explain <annotation-id>` returns supporting and contradicting structural evidence, confidence class and the missing data required to decide.
- `rfd lx visual journey --fixture ...` asks whether an agent can discover annotated evidence, inspect the correct frame, identify missing observability and recover without an ungrounded explanation.

Scrawl transcription may be wrong; preserve original spoken text reference and any uncertainty. Annotated human judgments should be linked to independent acceptance criteria where practical. Agent-generated visual assertions are hypotheses, and a trained evaluator must be isolated from the proposer in held-out tests.

## Harness² flow

1. A user or agent supplies an authorized visual bundle attached to a particular task/run; DevTools imports and validates it offline.
2. A task-scoped VisualEvidence subset may enter the proposer's *data context*, with artifact refs and missing-evidence labels; no injection into privileged system instructions.
3. Harness² proposes a component, observer, context policy, or agent/UI interaction change and declares what symptom it expects to correct.
4. Controlled baseline and candidate executions produce new traces and optionally comparable visual artifacts. DevTools links changed visible outputs to changed graph context and source revisions without pretending a historical counterfactual was actually executed.
5. A separate oracle/human label determines visual acceptability. The selector uses authorized data; private held-out verdict is not made visible to the proposal step.
6. Retain reusable diagnosis, falsification and successful component fragments with their environment bounds.

## Product gates

V0 manifest validator, fake Scrawl bundle + missing-media negative fixtures; no Scrawl executable required.
V1 offline import and frame/mark/transcript links; deterministic coordinate/crop transforms.
V2 correlation with an intentionally instrumented TSX component and an independent Fiber render receipt; fake/tampered/missing mapping refused.
V3 real capture and real reactive trace aligned by at least two anchors; show timing uncertainty.
V4 visually annotated regression fixture: correct change, wrong change, unchanged image but wrong state and correct image with incomplete trace.
V5 Harness² receives bounded permitted visual evidence without leaking independent grade; experiment comparison links original/revised visual/source evidence.
V6 native TSX/TUI annotations use same provider interface, only when their actual runtime instrumentation supports it.

Measure import/expansion cost, duplicate stored bytes, time spent resolving annotation to component, successful agent recovery, false confident mapping and trace-source completeness. Scrawl is optional at every gate except the actual Scrawl adapter test.
