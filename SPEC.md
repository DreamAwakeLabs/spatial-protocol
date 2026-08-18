# Dream Awake Spatial Protocol (DASP) v0.1

**Status:** pre-alpha / normative design, executable schemas still under verification  
**Encoding:** UTF-8 JSON  
**Initial transport profile:** WebSocket  
**Schema source of truth:** JSON Schema 2020-12  
**License:** Apache-2.0

DASP is an application-level protocol for connecting realtime runtimes, narrative systems, XR clients, robots, media systems, sensors, and simulators without coupling any participant to another participant's implementation technology.

The platform-neutrality rule in `docs/PLATFORM-NEUTRALITY.md` is binding.

## 1. Goals

DASP v0.1 defines:

1. node presence and lifecycle;
2. capability discovery;
3. typed semantic events;
4. typed semantic intents;
5. concrete action requests;
6. action acceptance, progress, completion, failure, and cancellation;
7. state snapshot request/response;
8. high-rate stream advertisement/reference;
9. tracing and causality;
10. session scoping;
11. expiry, retry, and idempotency semantics;
12. protocol/version negotiation;
13. extension and authorization hooks.

DASP intentionally does **not** define raw media framing, Unreal replication, robot servo commands, MIDI timing, render synchronization, or vendor-specific tracking APIs.

## 2. Message families

Core message kinds use dotted lowerCamelCase:

```text
node.hello
node.welcome
node.heartbeat
node.goodbye
node.offline
capability.announce
capability.revoke
event.observed
intent.proposed
intent.result
action.request
action.cancel
action.result
state.request
state.snapshot
stream.announce
stream.revoke
protocol.error
```

Private/experimental kinds, event types, fields, or capabilities MUST be explicitly namespaced under `x.` until promoted to the shared protocol, for example `x.dreamawake.experimental.hologram.setPhase`.

## 3. IDs

Entity IDs use:

```text
<namespace>:<opaque-id>
```

Reserved v0.1 namespaces include:

```text
installation
session
node
frame
visitor
participant
persona
embodiment
character
robot
object
zone
surface
sensor
scene
quest
cue
stream
```

Recommended syntax:

```regex
^[a-z][a-z0-9-]*:[A-Za-z0-9][A-Za-z0-9._~-]{0,127}$
```

Static authored entities SHOULD use stable readable slugs. Dynamic entities SHOULD use ULIDs.

## 4. Coordinate frames and units

Cross-system geometry MUST identify a frame of reference. The recommended canonical installation frame is:

```text
frame:installation
```

For new installations it is Z-up, right-handed, and measured in meters. Adapters convert from native platform conventions at their boundaries.

A semantic transform is represented as:

```json
{
  "frameId": "frame:installation",
  "positionM": { "x": 1.2, "y": -0.4, "z": 1.7 },
  "rotation": { "x": 0.0, "y": 0.0, "z": 0.0, "w": 1.0 }
}
```

Core physical values use SI units. Field names SHOULD carry unit suffixes when ambiguity is plausible, such as `distanceM`, `speedMps`, `durationMs`, and `frequencyHz`.

High-rate poses are not DASP control messages. They use a stream transport whose descriptor declares frame and units.

## 5. Envelope

Every control message uses the same envelope:

```json
{
  "specVersion": "0.1",
  "messageId": "01K3S94Y3FCQEC33BHE3G6EVDA",
  "kind": "event.observed",
  "sessionId": "session:01K3S90TE6CK8TEKFG7JK0B5AA",
  "source": "node:unreal-primary",
  "target": "node:narrative-adapter",
  "timestamp": "2026-08-16T23:45:12.482Z",
  "sequence": 184,
  "traceId": "01K3S94XQVAGKMX9MVDJKX8QYJ",
  "causationId": null,
  "correlationId": null,
  "ttlMs": null,
  "payloadVersion": 1,
  "payload": {},
  "extensions": {}
}
```

Required fields are `specVersion`, `messageId`, `kind`, `source`, `timestamp`, `payloadVersion`, and `payload`.

`sessionId` is required for experience-scoped traffic and may be omitted during initial node registration or installation health operations.

`messageId` is a globally unique ULID. Retransmission of the same logical message SHOULD preserve the original message ID when deduplication is desired.

`sequence` is optional and monotonically increasing within a node instance/session scope. It is diagnostic ordering, not a global total order.

`traceId` groups a causal chain. `causationId` points to the message that directly caused the message. `correlationId` groups application workflows when a trace is broader than one request.

`ttlMs` is measured from `timestamp`. Expired actions and intents MUST NOT begin execution.

Top-level schemas are strict. Vendor/private data belongs in namespaced `extensions` and MUST be safely ignorable by clients that do not understand it.

## 6. Node lifecycle

1. A node opens the DASP transport.
2. The transport authenticates it.
3. The node sends `node.hello`.
4. The router validates identity and compatible protocol versions.
5. The router sends `node.welcome` with the negotiated version and heartbeat policy.
6. The node announces capabilities and streams.
7. The node emits heartbeats.
8. The router makes node/capability presence available to authorized peers.

A node MUST NOT execute side-effecting actions from a connection before receiving `node.welcome`.

`instanceId` changes on process restart and distinguishes a new process from a reconnecting transport.

When heartbeat timeout is exceeded, capability availability from that node is revoked before new actions are routed to it.

## 7. Capability discovery

A capability descriptor identifies semantic behavior, not a function name:

```json
{
  "capability": "character.lookAt",
  "capabilityVersion": 1,
  "executorEntityId": "embodiment:mara-wall",
  "concurrency": "replaceByKey",
  "metadata": {}
}
```

Discovery is not authorization. The router/executor must still enforce policy, actor/target validity, local safety, and capability-specific validation.

Examples:

```text
character.speak
character.lookAt
character.emote
robot.lookAt
robot.gesture
visual.playCue
audio.playCue
pcg.applyState
music.setIntent
```

## 8. Events

`event.observed` states a fact that a source claims occurred. Events are immutable facts once accepted into a durable event log.

Examples:

```text
participant.enteredZone
participant.exitedZone
participant.interacted
participant.attentionChanged
object.interacted
scene.entered
world.stateChanged
quest.stateChanged
music.bar
music.sectionChanged
```

Events MAY carry provenance and confidence. Confidence never turns a speculative observation into authority; downstream policy decides what evidence is sufficient.

A physical/game runtime should be authoritative for facts it directly executes or senses, such as collision, object pickup, zone entry, device completion, and transforms.

## 9. Intents

`intent.proposed` represents a desired semantic outcome that still requires planning or execution selection.

Examples include "increase musical tension", "engage the visitor", or "make the north wall feel unstable".

An intent is not proof that anything happened. It must resolve into actions, a refusal, or a no-op result.

Use an action instead when the intended executor and capability are already known.

## 10. Actions

`action.request` asks an executor to perform a declared capability. The payload includes a stable `actionId`, capability name/version, optional executor and target entities, validated arguments, priority, optional concurrency key, and whether a terminal result is required.

An LLM or narrative system MUST NOT request arbitrary engine method names, robot SDK calls, UObject paths, or script fragments. It requests semantic capabilities.

Executors validate locally before side effects.

### Idempotency

`actionId` identifies the logical action. Re-delivery of an action with the same `actionId` MUST NOT perform the side effect twice. The executor returns the known state/result for that action when possible.

### TTL

An action whose envelope TTL has expired MUST be rejected before execution.

### Results

`action.result` status values are:

```text
accepted
rejected
started
progress
completed
failed
cancelled
timedOut
```

A result tells the requester what actually happened. `accepted` is not equivalent to `completed`.

Recommended stable error codes include `invalidArguments`, `capabilityUnavailable`, `targetUnavailable`, `unauthorized`, `expired`, `busy`, `safetyRejected`, `executionFailed`, and `cancelled`.

Every physically meaningful action SHOULD produce a terminal result so narrative/world state can remain grounded in reality.

## 11. Cancellation and concurrency

`action.cancel` requests cancellation by `actionId`. Cancellation is best-effort unless the capability contract says otherwise. The executor emits a terminal result reflecting what actually happened.

Capabilities may declare concurrency semantics such as `parallel`, `serial`, or `replaceByKey`. A `concurrencyKey` can make gaze, speech, visual cue, or similar streams replace an older action without inventing platform-specific APIs.

## 12. State recovery

DASP supports `state.request` and `state.snapshot` so a reconnecting or newly joined node can recover authoritative projections without replaying every historical message.

Snapshots identify the authority/source, scope, schema version, and monotonic source revision where available.

DASP does not prescribe a global state database. Each subsystem owns its authoritative domain and exposes the projections other nodes need.

## 13. Streams and media

DASP control traffic must not become a media pipe. `stream.announce` advertises a high-rate stream and how to consume it.

Examples of side channels:

- WebRTC for camera/audio;
- NDI or shared-memory/video transport for textures;
- Live Link or platform tracking APIs for high-rate pose;
- OSC for dense low-latency scalar control when appropriate;
- device-native protocols for timing-critical hardware.

The stream descriptor may carry semantic source identity, frame, unit, codec/format, endpoint, and lifecycle information. DASP events/actions may refer to the stream ID.

## 14. Narrative Runtime Profile

The core protocol remains narrative-engine neutral. A Narrative Runtime Profile will standardize portable concepts such as persona identity, utterances, addressed personas, quest/objective projections, narrative state, attention, and conversation lifecycle.

Seer, another cloud narrative engine, or a local model can implement that profile through an adapter. Runtime clients must not need Seer-specific types.

The Narrative Runtime Profile is intentionally a post-core-alpha milestone in `roadmap/STATE.md` so its abstractions are built on stable core envelopes/actions rather than creating a second wire system.

## 15. Platform neutrality

Canonical DASP schemas MUST NOT expose Unreal, Unity, visionOS/RealityKit, Android XR, Meta Quest/OpenXR, TouchDesigner, Seer, or robot-SDK implementation types.

A platform adapter maps neutral concepts to native concepts locally:

```text
DASP entity             -> Unreal Actor
                        -> Unity GameObject
                        -> RealityKit Entity
                        -> XR/platform object

DASP character.lookAt   -> GameplayAbility/StateTree task
                        -> Unity behaviour
                        -> RealityKit animation/transform operation
                        -> robot driver command
```

DASP also MUST NOT assume the experience occurs inside a physical room. `space`, `zone`, `participant`, `entity`, and `embodiment` can represent a projection installation, AR/passthrough experience, immersive VR world, flatscreen game, robot-only system, or simulator.

See `docs/PLATFORM-NEUTRALITY.md` for the binding acceptance test.

## 16. OpenUSD

OpenUSD may describe authored spatial nouns: rooms, zones, sensors, projection surfaces, devices, anchors, and stable `dasp:entityId` metadata. DASP describes runtime verbs and facts.

USD is not a runtime action bus and must not become the gameplay/state database.

## 17. Initial transport profile

Recommended WebSocket subprotocol:

```text
dasp.v0.1.json
```

Each text frame contains one complete JSON envelope.

Recommended endpoints for a router implementation:

```text
GET /health
GET /ready
WS  /dasp
```

Transport authentication is implementation/deployment policy. The authenticated identity must agree with the envelope `source`.

## 18. Conformance strategy

JSON Schema is normative. The repository maintains language-neutral valid/invalid fixtures. Every SDK must run the same fixtures.

A tiny correct SDK is preferred to an elaborate generator that produces subtly incompatible models.

v0.1 alpha acceptance requires at least:

1. envelope/entity/node schemas;
2. capability schemas;
3. event/intent/action/result schemas;
4. state and stream schemas;
5. at least 20 valid/invalid fixture vectors;
6. a Python reference validator/client;
7. two fake nodes in the simulator;
8. capability advertisement;
9. accepted/completed action flow;
10. duplicate action deduplication;
11. expired action rejection;
12. heartbeat timeout removing capability availability;
13. causal trace of the whole flow;
14. one Unreal test client passing the same envelope/action fixtures.

## 19. Deferred beyond v0.1

Deferred until a concrete need exists:

- binary control encoding;
- NATS/MQTT broker profiles;
- generalized state patches / CRDTs;
- automatic mDNS discovery;
- federation across installations;
- synchronized future action scheduling;
- formal Dream Awake OpenUSD schema;
- resource/blob transfer protocol;
- capability composition/planning language;
- standardized skeletal stream schema;
- generic robot navigation semantics;
- persistent visitor identity.
