# DASP Platform-Neutrality Rule

## Binding architectural rule

DASP is a platform-, engine-, device-, and vendor-neutral protocol.

No canonical DASP schema, identifier, capability, event, intent, action, state model, or Narrative Runtime Profile concept may depend on implementation-specific concepts from Unreal Engine, Unity, Apple visionOS/RealityKit, Android XR, Meta Quest, OpenXR, media-authoring tools, a particular story/narrative engine, a particular robot platform, or another runtime/device SDK.

Canonical DASP MUST NOT expose concepts such as:

- Unreal `Actor`, `UObject`, `Component`, `GameplayTag`, Gameplay Ability, Blueprint, Niagara, MetaSound, PCG graph, `FVector`, or Unreal replication semantics;
- Unity `GameObject`, `MonoBehaviour`, `ScriptableObject`, component types, or Unity-specific serialization;
- RealityKit/ARKit implementation classes, Swift types, window/volume/immersive-space types;
- Android XR, Meta Quest, or OpenXR handles, action paths, session objects, anchors, or vendor-extension identifiers as canonical semantics;
- private story-engine session classes, quest-ledger implementation classes, database identifiers, or provider-native WebSocket frames;
- media-tool operator/node types;
- robot SDK classes, joints, motors, or raw hardware command structures.

DASP instead describes semantic concepts such as:

```text
entity
participant
persona
embodiment
device
space
zone
transform
anchor
capability
event
observation
intent
action
result
conversation
utterance
quest
objective
narrative state
world state
media cue
```

Each platform/provider adapter maps these neutral concepts into native implementation mechanisms.

## Identity neutrality

DASP keeps these identities distinct:

- **node identity**: a connected runtime/process/device;
- **session identity**: the experience-instance scope for related traffic;
- **participant identity**: a human/player/agent taking part in the experience;
- **persona identity**: a narrative character/personality;
- **embodiment identity**: a concrete visual/robotic/audio body used by a persona.

A protocol design MUST NOT assume one participant per node, one participant per session, one persona per socket, or one embodiment per persona.

## Physical-space neutrality

DASP MUST NOT assume an experience takes place inside a physical installation. The same protocol must support a projection-mapped room, passthrough AR, immersive VR, a conventional game, an XR multiplayer world, a robot-only experience, or a headless simulator.

## Extensions

Platform-specific data MAY appear under explicitly namespaced extension metadata, for example:

```json
{
  "extensions": {
    "com.dreamawake.unreal": {},
    "com.apple.visionos": {},
    "org.khronos.openxr": {}
  }
}
```

Extension data:

1. must not be required to understand canonical message meaning;
2. must not alter canonical semantics;
3. must be safely ignorable by unaware clients;
4. must not be promoted into core merely because one implementation needs it.

## Acceptance test

For every proposed shared concept, ask:

> Could an Unreal game, Unity game, Apple Vision Pro application, Android XR application, Meta Quest application, physical robot, and headless simulator all represent this concept without pretending to be another platform or provider?

If the answer is no, redesign the abstraction or move it into a platform/provider adapter.
