# Dream Awake Spatial Protocol (DASP)

DASP is an open, platform-neutral protocol for connecting realtime experiences, narrative intelligence, game engines, XR runtimes, robots, sensors, media systems, and other spatial-capable nodes.

**Status:** v0.1 pre-alpha  
**License:** Apache-2.0

## Core rule

DASP describes semantic runtime concepts, never engine implementation details. Unreal Actors, Unity GameObjects, RealityKit Entities, OpenXR handles, private story-engine internals, media-tool operators, robot joints, and similar vendor/runtime types terminate at adapters.

The same contract should be usable by an Unreal game, Unity application, Apple Vision Pro experience, Android XR / Meta Quest application, physical installation, robot, web client, and headless simulator.

## v0.1 control plane

- UTF-8 JSON
- JSON Schema 2020-12 is normative
- WebSocket is the initial transport profile
- Events describe facts that happened
- Intents describe desired semantic outcomes
- Actions request execution of a declared capability
- Results report what actually happened
- High-rate video/audio/pose streams stay on fit-for-purpose media transports

## Repository layout

```text
SPEC.md                       normative design
schemas/                      executable JSON Schemas
docs/                         architecture and agent rules
roadmap/STATE.md              implementation queue / ground truth
fixtures/                     shared conformance vectors
packages/                     language bindings (planned)
```

The first alpha is complete when two fake nodes can register, advertise a capability, exchange an idempotent action request/result flow, reject an expired action, and validate the exchange against the same schemas used by every SDK.

See [SPEC.md](SPEC.md), [docs/PLATFORM-NEUTRALITY.md](docs/PLATFORM-NEUTRALITY.md), [docs/NARRATIVE-RUNTIME-PROFILE-DRAFT.md](docs/NARRATIVE-RUNTIME-PROFILE-DRAFT.md), and [roadmap/STATE.md](roadmap/STATE.md).
