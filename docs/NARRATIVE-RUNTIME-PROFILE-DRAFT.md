# DASP Narrative Runtime Profile — Vocabulary Draft

**Status:** non-binding design draft  
**Normative schemas:** deferred until DASP core `v0.1.0-alpha.1`  
**Purpose:** align independent story/narrative providers and realtime runtimes before names freeze.

This document is deliberately provider-neutral. It must be useful to a cloud story engine, local model, game-native narrative system, or future provider without exposing private provider names, endpoints, storage models, or wire frames.

## 1. Core identity model

The profile distinguishes five identities:

```text
node          connected runtime/process/device
session       DASP experience-instance scope
participant   human/player/agent participating in that experience
persona       narrative character/personality
embodiment    visual/robotic/audio body presenting a persona
```

These relationships are many-to-many unless a specific experience constrains them.

A DASP session MAY contain:

- multiple participants;
- multiple runtime/device nodes;
- multiple personas;
- multiple embodiments;
- zero or more provider-native sessions behind adapters.

A provider adapter owns the mapping from a DASP `sessionId` to provider-native session identifiers. Provider-native session IDs MUST NOT become canonical DASP identity.

## 2. Participant attribution

Participant identity is explicit. It MUST NOT be inferred from:

- socket identity;
- node identity;
- transport connection;
- session membership;
- provider-native user/session identity.

Participant-scoped events and observations SHOULD carry `participantId` explicitly.

This allows one shared DASP experience session to represent a solo player today and a party/shared instance later without changing the protocol model.

## 3. Runtime authority

The realtime runtime remains authoritative for physical/gameplay facts it executes or senses, including:

- transforms and movement;
- collision and interaction;
- inventory ownership;
- object pickup;
- zone/space entry;
- interactability;
- device state;
- whether a requested action actually executed;
- local safety limits.

A narrative provider may mirror these facts, reason about them, remember them, or derive quest/story consequences. A mirror is not authority.

## 4. Stable event identity and provider idempotency

DASP retries the same logical event with the same envelope `messageId`.

A provider adapter SHOULD map that stable DASP event `messageId` one-to-one onto the provider's idempotency token when the provider requires one.

Rules:

1. one genuine event -> one stable DASP event ID;
2. retries preserve that ID;
3. a new state transition or repeated occurrence receives a new ID even if its payload resembles an earlier event;
4. adapters do not invent a fresh provider idempotency key on retry.

For actions, DASP `actionId` remains the logical action idempotency key.

## 5. Narrative perception snapshot

Some story engines need low-rate semantic perception rather than raw media. DASP should be able to express a provider-neutral snapshot containing the information needed for narrative reaction selection.

Draft semantic shape:

```json
{
  "participantId": "participant:player-1",
  "spaceId": "zone:bar",
  "presentEntities": [
    {
      "entityId": "character:merchant",
      "personaId": "persona:merchant",
      "distanceM": 2.4,
      "inView": true
    }
  ],
  "attentionTargetId": "persona:merchant",
  "speaking": false
}
```

The load-bearing semantic fields are:

- participant identity;
- present entities/personas;
- participant attention/gaze target when known;
- distance/proximity when known;
- participant speaking state when known.

Unknown values are omitted rather than fabricated.

Snapshots are preferred over deltas for low-rate narrative perception because a lost snapshot self-heals on the next one.

DASP does **not** mandate a perception cadence. A provider may coalesce snapshots at 2 Hz, 1 Hz, on-change, or another policy without changing DASP semantics.

Raw camera/audio/pose streams remain outside the Narrative Runtime Profile and use appropriate media/tracking transports.

## 6. Personas and embodiments

`personaId` identifies narrative identity. `embodimentId` identifies where/how that persona is currently presented.

A persona may:

- have no current embodiment;
- have one embodiment;
- move between embodiments;
- be represented simultaneously by multiple embodiments if the experience permits it.

Embodiment-specific animation, rig, robot, display, and audio details remain adapter/runtime concerns.

## 7. Addressing and attention

The profile should support explicit addressing independent of the provider's internal arbitration model.

Portable concepts include:

```text
addressedPersonaId
attentionTargetId
participantId
```

A narrative provider may use those inputs plus its own story state, presence, cooldowns, relationships, or dialogue-floor logic to decide who responds.

Silence/no-response is a valid outcome.

## 8. Utterance identity and audio binding

Narrative speech should expose provider-neutral metadata independent of audio codec or transport:

```text
personaId
utteranceId
participantId?      # if directly addressed / scoped
conversationId?     # if used by the experience
```

`utteranceId` identifies one logical spoken utterance. Audio chunks, viseme/blendshape data, captions, and completion status can refer to that ID.

DASP does not require audio bytes to travel over the DASP control plane. A side-channel audio stream may bind chunks to the active `utteranceId` through its stream/profile metadata.

Rig-specific facial expansion is runtime/adapter-side. The narrative provider supplies source speech/viseme/blendshape data; the runtime decides how to drive MetaHuman, RealityKit, Unity, a robot face, or another embodiment.

## 9. Quest and objective projection

The profile should standardize read-only portable projections such as:

```text
questId
questStatus
objectiveId
objectiveStatus
progress
revision / sequence
```

The story engine/provider may remain authoritative for deterministic narrative quest state, while the realtime runtime remains authoritative for physical facts feeding it.

DASP does not require every provider to implement quests.

## 10. Conversation and provider sessions

DASP `sessionId` is the experience-instance scope, not necessarily the provider's conversation/session object.

A provider adapter may map:

```text
1 DASP session + 1 participant -> 1 provider session
```

or later:

```text
1 DASP session + N participants -> 1 shared provider instance
```

or:

```text
1 DASP session -> multiple provider-native sessions
```

without changing the DASP-facing runtime contract.

If a distinct portable conversation identity proves necessary, it should be added explicitly rather than overloading `sessionId`.

## 11. Provider conformance principle

A provider integration conforms to the Narrative Runtime Profile by translating between its private/native concepts and DASP concepts in an adapter.

The provider itself does not need to use DASP internally.

A realtime runtime does not need to know which provider is behind the adapter.

## 12. Open questions before NARR-00 ratification

- Do perception snapshots deserve a dedicated message/profile kind or remain typed `event.observed`/`state.snapshot` payloads?
- Should portable conversation identity be first-class in v0.1?
- Which quest/objective fields are genuinely cross-provider?
- How should utterance lifecycle events be named and ordered?
- Is persona-to-embodiment binding best represented as state projection or events?
- Which interruption/cancellation semantics belong in the profile versus ordinary DASP action cancellation?

These remain design questions. This draft must not be treated as normative schema until `NARR-00`/`NARR-01` land.
