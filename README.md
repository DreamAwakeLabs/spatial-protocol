# Dream Awake Spatial Protocol (DASP)

DASP is an open, platform-neutral protocol for connecting realtime experiences, narrative intelligence, game engines, XR runtimes, robots, sensors, media systems, and other spatial-capable nodes.

**Status:** v0.1 pre-alpha  
**License:** Apache-2.0

## Core rule

DASP describes semantic runtime concepts, never engine implementation details. Unreal Actors, Unity GameObjects, RealityKit Entities, OpenXR handles, Seer internals, TouchDesigner operators, robot joints, and similar vendor/runtime types terminate at adapters.

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

See `SPEC.md` and `roadmap/STATE.md`.
