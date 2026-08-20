# Common Fable Implementation Contract

This contract applies to Dream Awake Spatial OS repositories.

## Ground truth

A fresh agent must orient from repository files and git/PR state, not remembered conversation state. Verify every load-bearing premise before changing code. If a brief is stale, record the correction and build against reality.

## Boundary discipline

Each repository owns one layer. Do not solve a cross-repo problem by adding a private shortcut dependency that bypasses DASP.

Shared DASP contracts must never expose Unreal, Unity, visionOS/RealityKit, Android XR, Meta Quest/OpenXR, private story-engine/provider, media-tool, or robotics implementation types. Platform/provider concepts terminate at adapters. If a proposed abstraction only makes sense on one runtime or one provider, it belongs in that adapter/package rather than DASP.

Public repositories must not disclose private narrative-provider/product names, private endpoint names, or private wire models. Use neutral terms such as `story engine`, `narrative provider`, and `provider-native API`.

## Identity discipline

Node, experience-session, participant, persona, and embodiment identity are distinct. Do not infer one from another. Shared designs must allow one experience session to contain multiple participants, runtime nodes, personas, embodiments, and provider-session mappings.

## Runtime safety

AI/LLM systems may express semantic intent, but timing-critical, hardware-critical, and safety-critical operations are executed by deterministic local code. Every external side effect passes through a declared capability executor and produces observable status/results.

Never let a model invoke arbitrary methods, engine object paths, shell/code fragments, or raw motor commands through a shared protocol.

## Delivery discipline

- Work in small independently verifiable PR-sized units.
- Add unit/conformance tests at the same time as implementation.
- Prefer fake backends/simulators so integration development is not blocked by hardware or another repository.
- Keep transport, domain semantics, and implementation adapters separable.
- Avoid speculative framework layers. Generalize only after real consumers demonstrate the abstraction.
- Preserve idempotency and explicit error behavior across retries/reconnects.
- Preserve a stable DASP `messageId` when retrying the same logical event; preserve `actionId` for the same logical action.
- Document architectural decisions that constrain another repository.

## Protocol change rule

If an implementation needs a capability or event DASP cannot represent, stop and propose a protocol change. Do not invent an undocumented side channel for semantic control. High-rate media/tracking side channels are expected, but their semantic identity/lifecycle should be advertised through DASP when relevant.
