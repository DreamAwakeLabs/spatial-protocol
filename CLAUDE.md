# CLAUDE.md — Dream Awake Spatial Protocol

This is the canonical agent guide for `DreamAwakeLabs/spatial-protocol`.

## Mission

Build the smallest correct, portable DASP contract that lets independent realtime, narrative, and device systems interoperate without importing one another's implementation details.

Read, in order:

1. `SPEC.md`
2. `docs/PLATFORM-NEUTRALITY.md`
3. `docs/NARRATIVE-RUNTIME-PROFILE-DRAFT.md`
4. `docs/COMMON-FABLE-CONTRACT.md`
5. `roadmap/STATE.md`
6. current git/PR reality

`roadmap/STATE.md` is the implementation queue. This file contains process and invariants, not mutable project status.

## Binding architecture rules

- DASP is platform-, engine-, device-, and vendor-neutral.
- Never put Unreal/Unity/RealityKit/OpenXR/private story-engine/media-tool/robot-SDK types into canonical schemas.
- Public protocol docs must not disclose private provider/product names or private provider wire types.
- JSON Schema 2020-12 is normative for v0.1.
- Core schemas use strict top-level validation. Private data belongs in namespaced `extensions`.
- Events are facts. Intents are desired outcomes. Actions are executable capability requests. Results state what actually happened.
- AI systems never send arbitrary method/function names to runtimes or hardware.
- `actionId` is the logical action idempotency key; retries must not duplicate side effects.
- Re-sending the same logical event preserves its DASP `messageId`; adapters may map that stable ID to a provider-native idempotency key.
- `sessionId`, node identity, and participant identity are distinct concepts. Never infer participant identity from a socket or node.
- TTL is enforced before execution.
- Raw media/high-rate tracking is out-of-band and referred to by stream descriptors.
- Do not create a second narrative-engine-specific protocol inside core DASP.

## Platform-neutrality gate

Before adding a canonical concept, ask:

> Could an Unreal game, Unity game, Apple Vision Pro application, Android XR / Meta Quest application, physical robot, and headless simulator all represent this concept without pretending to be another platform?

If not, the concept is probably at the wrong layer.

## Working style

- Verify brief claims against repository reality before implementation.
- Work one PR-sized roadmap row at a time.
- Prefer additive, testable changes.
- Add fixtures before generated-SDK sophistication.
- Breaking schema changes require a version decision and an ADR/spec update.
- Do not mark a roadmap row done because a scaffold exists. Done means acceptance tests pass.
- When another repo needs a new shared concept, propose the smallest semantic DASP change rather than copying its private wire format.

## Initial target

The first release target is `v0.1.0-alpha.1`: envelope/entity/node lifecycle, capability discovery, events/intents/actions/results, state/stream descriptions, conformance fixtures, and a Python reference implementation sufficient for the Spatial OS simulator.

The Narrative Runtime Profile follows core alpha as a normative schema milestone, but a non-binding vocabulary draft is maintained now so independent story-engine providers can align before names freeze.
