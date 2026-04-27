# CoBuddy V0 WebRTC Session Contract

Status: `v0.1-draft`  
Review owner: product + protocol discussion  
Last updated: 2026-04-27

## Purpose

This contract defines the first client-to-server realtime session interface for CoBuddy V0.

It covers the connection between:

```text
Chrome Extension Client <-> CoBuddy Server
```

It does not define the server-to-model implementation. The server may use OpenAI Realtime, Gemini Live, LiveKit, a chained STT/LLM/TTS pipeline, or another provider behind an adapter. The client should not depend on provider-specific model events.

## Goals

- Support low-latency voice-first interaction.
- Keep watch-channel and user-channel data distinct.
- Let the server own session timeline, context fusion, memory, and delegation.
- Provide clear events for UI indicators.
- Support configurable interruption/barge-in.
- Support explicit memory and external-context permissions.
- Keep the contract provider-neutral.
- Preserve a path to self-hosting and alternate clients.

## Non-Goals

- Define server-to-model APIs.
- Define OpenClaw's internal protocol.
- Define A2A, ACP, or MCP compatibility.
- Support all clients beyond Chrome extension.
- Support arbitrary media platforms beyond browser media.
- Standardize long-term memory storage.
- Define full E2E encryption or auth policy in detail.

## Transport Shape

V0 uses WebRTC for the realtime client-server session.

Required WebRTC components:

- User microphone audio track from client to server.
- Watch-channel audio support from client to server, either as a WebRTC audio track or encoded `watch.audio.segment` events.
- Assistant audio track from server to client.
- Data channel for session, media, memory, delegation, and UI state events.

Optional/future WebRTC components:

- Watch-channel video frame track or sampled image events.
- Assistant text stream over data channel.
- Multiple data channels by event class.

V0 should begin with one reliable ordered data channel unless testing shows it harms latency-sensitive events.

Suggested data channel:

```text
label: "cobuddy.session.v0"
ordered: true
reliable: true
```

Future optimization may split channels:

- `cobuddy.control`: reliable ordered
- `cobuddy.media`: unreliable or partially reliable for high-frequency watch events
- `cobuddy.status`: reliable ordered for indicators and task state

## Connection Lifecycle

### 1. Create Session

The client asks the CoBuddy server to create a WebRTC session through HTTPS or an equivalent signaling endpoint.

Client sends:

```json
{
  "client": {
    "type": "chrome_extension",
    "version": "0.1.0"
  },
  "capabilities": {
    "userAudio": true,
    "watchMetadata": true,
    "watchTranscript": true,
    "watchTranscriptSnapshot": true,
    "watchAudio": true,
    "watchVideoFrames": false,
    "textFallback": true,
    "wakePhrase": true,
    "pushToTalk": true,
    "clientBargeInDetection": true
  },
  "requestedSources": {
    "watchMetadata": true,
    "watchTranscript": true,
    "watchAudio": true,
    "userAudio": true
  }
}
```

Server returns:

```json
{
  "sessionId": "sess_...",
  "rtc": {
    "iceServers": [],
    "expiresAt": "2026-04-27T10:30:00Z"
  },
  "policy": {
    "memoryDefault": "explicit_only",
    "externalShareDefault": "confirm",
    "rawMediaRetention": "none",
    "interruption": {
      "mode": "client_or_server",
      "stopAssistantAudio": true,
      "allowProviderBargeIn": true
    }
  }
}
```

### 2. WebRTC Offer/Answer

The client and server exchange SDP through signaling.

The exact signaling endpoint is not part of the long-term contract, but V0 should keep it simple:

```text
POST /v0/realtime/sessions
POST /v0/realtime/sessions/{sessionId}/offer
POST /v0/realtime/sessions/{sessionId}/ice
```

### 3. Data Channel Ready

When data channel opens, the client sends:

```json
{
  "type": "event",
  "event": "client.ready",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:00.000Z",
  "payload": {
    "sessionId": "sess_..."
  }
}
```

Server responds with:

```json
{
  "type": "event",
  "event": "session.opened",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:00.050Z",
  "payload": {
    "sessionId": "sess_...",
    "state": "open",
    "capture": {
      "userAudio": true,
      "watchMetadata": true,
      "watchTranscript": true,
      "watchAudio": true,
      "watchVideoFrames": false
    }
  }
}
```

## Message Envelope

All data channel messages use a shared envelope.

```json
{
  "type": "event",
  "event": "watch.transcript.delta",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:01.000Z",
  "seq": 42,
  "payload": {}
}
```

Fields:

- `type`: `event`, `request`, `response`, or `error`
- `event`: event name for event messages
- `method`: method name for request messages
- `id`: unique message id
- `requestId`: matching id for responses
- `ts`: client or server wall-clock timestamp
- `seq`: monotonic sequence number per sender
- `payload`: event or method payload

Open question:

- Should timestamps be wall-clock only, monotonic session time only, or both?

Recommendation:

- Use both in v0 when possible: `ts` for wall-clock and `sessionMs` for monotonic alignment.

## Channel Model

CoBuddy V0 has two primary input channels.

### Watch Channel

The watch channel represents the media the user is consuming.

Event family:

```text
watch.*
```

Examples:

- `watch.media.metadata`
- `watch.playback.updated`
- `watch.transcript.delta`
- `watch.audio.segment`
- `watch.video.frame`

### User Channel

The user channel represents direct user interaction with CoBuddy.

Event family:

```text
user.*
```

Examples:

- `user.audio.started`
- `user.audio.ended`
- `user.transcript.delta`
- `user.text.submitted`
- `user.wake.detected`
- `user.interrupt`

### Assistant Channel

The assistant channel represents CoBuddy's response content and audio output.

Event family:

```text
assistant.*
```

Examples:

- `assistant.response.started`
- `assistant.response.ended`
- `assistant.audio.started`
- `assistant.audio.ended`
- `assistant.text.delta`
- `assistant.interrupted`

### System Channel

The system channel represents state, permissions, generic tool calls, memory content visibility, and delegation transparency.

Event families:

```text
session.*
state.*
capture.*
tool.*
memory.content.*
delegation.*
error.*
```

## Watch Channel Events

### `watch.media.metadata`

Sent by client when media is detected or changes.

```json
{
  "type": "event",
  "event": "watch.media.metadata",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:02.000Z",
  "sessionMs": 2000,
  "payload": {
    "source": "youtube",
    "url": "https://www.youtube.com/watch?v=...",
    "title": "Example long-form talk",
    "channel": "Example Channel",
    "durationMs": 5400000,
    "language": "en"
  }
}
```

### `watch.playback.updated`

Sent by client on meaningful playback changes.

```json
{
  "type": "event",
  "event": "watch.playback.updated",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:05.000Z",
  "sessionMs": 5000,
  "payload": {
    "positionMs": 812000,
    "state": "playing",
    "playbackRate": 1.0
  }
}
```

Suggested send policy:

- On play/pause/seek/rate change.
- Every 5-10 seconds while playing.
- Immediately before user utterance if stale.

### `watch.transcript.delta`

Sent by client when transcript/caption text is available.

```json
{
  "type": "event",
  "event": "watch.transcript.delta",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:06.000Z",
  "sessionMs": 6000,
  "payload": {
    "source": "youtube_captions",
    "positionStartMs": 812000,
    "positionEndMs": 816000,
    "speaker": "unknown",
    "text": "Personal agents need long-term memory, otherwise they are just chatbots.",
    "isFinal": true
  }
}
```

### `watch.transcript.snapshot`

Sent by client when a site exposes a full transcript/caption file or when the client can upload an at-once transcript for the current media.

V0 should support this because many media sites expose complete transcript artifacts. A snapshot lets the server index the whole media timeline while still using `watch.playback.updated` and recent deltas for current-moment grounding.

```json
{
  "type": "event",
  "event": "watch.transcript.snapshot",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:06.000Z",
  "sessionMs": 6000,
  "payload": {
    "source": "youtube_captions",
    "format": "segments",
    "language": "en",
    "mediaRevision": "rev_...",
    "segments": [
      {
        "positionStartMs": 812000,
        "positionEndMs": 816000,
        "speaker": "unknown",
        "text": "Personal agents need long-term memory, otherwise they are just chatbots."
      }
    ],
    "hash": "sha256:..."
  }
}
```

For large transcripts, the client may send a URI or upload reference instead of inline segments:

```json
{
  "type": "event",
  "event": "watch.transcript.snapshot",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:06.000Z",
  "payload": {
    "source": "site_transcript_file",
    "format": "vtt",
    "transcriptRef": "upload_...",
    "hash": "sha256:..."
  }
}
```

### `watch.audio.segment`

V0 feature. Used when transcript is unavailable, incomplete, delayed, or when audio carries information not present in captions.

For continuous watch audio, a dedicated WebRTC audio track is preferred. `watch.audio.segment` is for explicit chunks, fallback paths, or debug/test clients.

```json
{
  "type": "event",
  "event": "watch.audio.segment",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:08.000Z",
  "sessionMs": 8000,
  "payload": {
    "source": "tab_audio",
    "positionStartMs": 818000,
    "positionEndMs": 820000,
    "mimeType": "audio/opus",
    "encoding": "base64",
    "data": "..."
  }
}
```

Raw watch audio should still follow session retention policy and should not be stored by default.

### `watch.video.frame`

Optional/future. Used for sampled frames or screenshots when the user permits visual context.

V0 should not require this.

## User Channel Events

### User Audio Track

The primary user audio should be a WebRTC audio track, not JSON over the data channel.

The server owns speech detection/transcription unless using client-side wake/push-to-talk metadata.

### `user.wake.detected`

Sent when client-side wake phrase activates listening.

```json
{
  "type": "event",
  "event": "user.wake.detected",
  "id": "evt_...",
  "ts": "2026-04-27T10:01:00.000Z",
  "sessionMs": 60000,
  "payload": {
    "mode": "wake_phrase",
    "phrase": "hey cobuddy",
    "confidence": 0.91
  }
}
```

### `user.text.submitted`

Text fallback from client.

```json
{
  "type": "event",
  "event": "user.text.submitted",
  "id": "evt_...",
  "ts": "2026-04-27T10:01:02.000Z",
  "sessionMs": 62000,
  "payload": {
    "text": "Can you explain that last claim?",
    "mode": "fallback_text"
  }
}
```

### `user.interrupt`

Sent when the user interrupts assistant speech.

```json
{
  "type": "event",
  "event": "user.interrupt",
  "id": "evt_...",
  "ts": "2026-04-27T10:01:04.000Z",
  "sessionMs": 64000,
  "payload": {
    "reason": "barge_in",
    "detectedBy": "client",
    "confidence": 0.93
  }
}
```

Server behavior:

- Stop assistant audio playback.
- Mark active response interrupted.
- Preserve interruption timestamp.
- Resume listening immediately.

Actual behavior depends on the active interruption policy.

## Interruption Policy

Interruption should be configurable because different transports and model providers can detect barge-in at different layers.

Supported policy modes:

```text
disabled
client
server
provider
client_or_server
server_or_provider
```

Mode meanings:

- `disabled`: user speech does not automatically interrupt assistant audio.
- `client`: client detects speech or explicit interrupt and sends `user.interrupt`.
- `server`: CoBuddy server detects user speech from the audio track and interrupts.
- `provider`: model/realtime provider owns interruption behavior.
- `client_or_server`: either client or server may trigger interruption.
- `server_or_provider`: server coordinates with provider-side interruption if available.

Recommended v0 default:

```json
{
  "mode": "client_or_server",
  "stopAssistantAudio": true,
  "clearQueuedAudio": true,
  "preservePartialAssistantText": true,
  "resumeListening": true,
  "minSpeechMs": 150,
  "confidenceThreshold": 0.75
}
```

Related events:

- `user.interrupt`: client-originated interrupt
- `assistant.interrupted`: server confirms assistant was interrupted
- `assistant.audio.stopped`: server confirms playback stopped
- `interruption.policy.updated`: server announces policy changes

Example `assistant.interrupted`:

```json
{
  "type": "event",
  "event": "assistant.interrupted",
  "id": "evt_...",
  "ts": "2026-04-27T10:01:04.050Z",
  "sessionMs": 64050,
  "payload": {
    "reason": "barge_in",
    "triggeredBy": "client",
    "responseId": "resp_...",
    "audioStopped": true
  }
}
```

If the server-to-model provider has native interruption controls, such as a realtime API session setting, the CoBuddy server should map the CoBuddy interruption policy into provider-specific configuration. The client should not need to know which provider implements the interruption.

Semantic user input is only text or audio. Preferences are not a separate client event type. If the user says "be shorter" or types a customization instruction, the server should interpret that from normal user input and, when appropriate, persist it through a standard memory tool call such as `memory.set`.

## State And Assistant Output Events

### `state.updated`

CoBuddy uses generic state updates instead of one named event per state. State is multi-dimensional: watch capture, user input, assistant output, processing, and delegation can all be active independently.

For example, while the assistant is speaking, the user audio input can still be listening so barge-in works. While a delegated agent is running, the user can continue watching and talking.

```json
{
  "type": "event",
  "event": "state.updated",
  "id": "evt_...",
  "ts": "2026-04-27T10:02:07.000Z",
  "payload": {
    "states": {
      "session": {
        "lifecycle": "open"
      },
      "watch": {
        "capture": "active",
        "sources": ["metadata", "transcript"],
        "playback": "playing"
      },
      "user": {
        "audioInput": "listening",
        "wake": "armed"
      },
      "assistant": {
        "output": "speaking",
        "canInterrupt": true
      },
      "processing": {
        "turn": "responding",
        "memory": "idle"
      },
      "delegation": {
        "activeRuns": 2
      }
    },
    "changed": ["assistant.output", "delegation.activeRuns"]
  }
}
```

Clients should treat unknown state domains and values as non-fatal. This keeps the contract extensible as new clients, devices, and modalities arrive.

Suggested state domains:

- `session`: lifecycle and retention flow.
- `watch`: watch-channel capture, playback, source availability.
- `user`: user-channel input, wake/push-to-talk, mute/listening.
- `assistant`: assistant output state such as idle, speaking, interrupted.
- `processing`: model/tool/memory activity needed for indicators.
- `delegation`: aggregate delegated-agent activity.
- `network`: reconnecting/degraded/connected status.

The assistant audio itself is sent on the WebRTC assistant audio track. `state.updated` only tells the client what state to display.

### Assistant Response Lifecycle

CoBuddy should expose assistant-side response lifecycle events so the client can group text/audio deltas into a coherent assistant response.

This is not a strict alternating user-assistant "round" model. The session may have background delegation, tool calls, watch updates, and memory updates arriving between user input and assistant output. The response lifecycle only wraps one assistant response.

### `assistant.response.started`

```json
{
  "type": "event",
  "event": "assistant.response.started",
  "id": "evt_...",
  "ts": "2026-04-27T10:02:06.800Z",
  "payload": {
    "responseId": "resp_...",
    "trigger": {
      "type": "user_input",
      "inputId": "evt_..."
    },
    "modalities": ["audio", "text"],
    "canInterrupt": true
  }
}
```

### `assistant.response.ended`

```json
{
  "type": "event",
  "event": "assistant.response.ended",
  "id": "evt_...",
  "ts": "2026-04-27T10:02:10.100Z",
  "payload": {
    "responseId": "resp_...",
    "reason": "completed"
  }
}
```

End reasons:

```text
completed
interrupted
cancelled
failed
superseded
```

### `assistant.audio.started`

```json
{
  "type": "event",
  "event": "assistant.audio.started",
  "id": "evt_...",
  "ts": "2026-04-27T10:02:07.000Z",
  "payload": {
    "responseId": "resp_...",
    "canInterrupt": true
  }
}
```

### `assistant.audio.ended`

```json
{
  "type": "event",
  "event": "assistant.audio.ended",
  "id": "evt_...",
  "ts": "2026-04-27T10:02:10.000Z",
  "payload": {
    "responseId": "resp_...",
    "reason": "completed"
  }
}
```

### `assistant.text.delta`

Optional text mirror for accessibility and debugging.

```json
{
  "type": "event",
  "event": "assistant.text.delta",
  "id": "evt_...",
  "ts": "2026-04-27T10:02:07.100Z",
  "payload": {
    "responseId": "resp_...",
    "text": "They are arguing that..."
  }
}
```

## Capture Events

### `capture.source.enabled`

```json
{
  "type": "event",
  "event": "capture.source.enabled",
  "id": "evt_...",
  "ts": "2026-04-27T10:00:00.500Z",
  "payload": {
    "source": "watchTranscript",
    "enabled": true
  }
}
```

## Tool And Memory Content Events

Notes, memory search, and memory updates are standard tool calls. The WebRTC contract should not create special note or memory-operation events for each product action.

Examples of standard tools:

- `notes.create`
- `notes.append`
- `memory.search`
- `memory.set`
- `memory.update`
- `openclaw.context.search`
- `permission.request`
- `media.playback.control`

The client receives generic tool-call events for transparency and memory-content events when memory is used or changed.

Tool calls may originate from the main CoBuddy server or from delegated agents. Provenance fields are required when the tool call is associated with a delegated agent run, so multiple concurrent delegated agents remain debuggable.

`tool.call.*` events are bidirectional when `executionMode` requires client execution. The server can request a client-required tool, and the client returns the result with the same `toolCallId`.

Tool calls also declare execution semantics:

- `server_executed`: server executes the tool; client receives notifications/results.
- `client_required`: client must execute or resolve the tool call and return a result.
- `client_optional`: client may execute the request if capable.
- `notification_only`: event is informational and does not require execution.

Permissions are modeled as tools. For example, before sharing context externally, the server can issue a `permission.request` tool call with `executionMode: "client_required"`. The client resolves it with a normal `tool.call.completed` event containing the user decision.

Media playback control is also modeled as a client-required tool. The server may request pause, resume, seek, or rate change, but the client is the authority that executes or rejects the action.

### `tool.call.started`

```json
{
  "type": "event",
  "event": "tool.call.started",
  "id": "evt_...",
  "ts": "2026-04-27T10:03:00.000Z",
  "payload": {
    "toolCallId": "tool_...",
    "toolName": "memory.search",
    "executionMode": "server_executed",
    "args": {
      "query": "live context vs long-term memory",
      "sources": ["cobuddy_notes", "preferences", "openclaw"]
    },
    "purpose": "Find context related to the user's question.",
    "visibility": "user_visible",
    "origin": {
      "type": "cobuddy"
    }
  }
}
```

### `tool.call.completed`

```json
{
  "type": "event",
  "event": "tool.call.completed",
  "id": "evt_...",
  "ts": "2026-04-27T10:03:00.500Z",
  "payload": {
    "toolCallId": "tool_...",
    "toolName": "memory.search",
    "status": "completed",
    "result": {
      "matches": 3
    },
    "resultSummary": "Found two relevant saved notes and one OpenClaw context item.",
    "visibility": "user_visible",
    "origin": {
      "type": "cobuddy"
    }
  }
}
```

### `tool.call.failed`

```json
{
  "type": "event",
  "event": "tool.call.failed",
  "id": "evt_...",
  "ts": "2026-04-27T10:04:00.000Z",
  "payload": {
    "toolCallId": "tool_...",
    "toolName": "notes.create",
    "status": "failed",
    "errorCode": "note_store_unavailable",
    "message": "Could not save the note.",
    "origin": {
      "type": "cobuddy"
    }
  }
}
```

Example client-required permission tool:

```json
{
  "type": "event",
  "event": "tool.call.started",
  "id": "evt_...",
  "ts": "2026-04-27T10:05:00.000Z",
  "payload": {
    "toolCallId": "tool_perm_...",
    "toolName": "permission.request",
    "executionMode": "client_required",
    "args": {
      "action": "share_context_with_agent",
      "target": "research_agent",
      "contextSummary": {
        "includes": [
          "saved_note",
          "video_title",
          "video_url",
          "timestamp",
          "last_3_minutes_transcript_summary"
        ],
        "excludes": [
          "raw_audio",
          "full_transcript"
        ]
      },
      "prompt": "Share this context with the research agent?"
    },
    "visibility": "user_visible",
    "origin": {
      "type": "cobuddy"
    }
  }
}
```

The client responds:

```json
{
  "type": "event",
  "event": "tool.call.completed",
  "id": "evt_...",
  "ts": "2026-04-27T10:05:05.000Z",
  "payload": {
    "toolCallId": "tool_perm_...",
    "toolName": "permission.request",
    "status": "completed",
    "result": {
      "decision": "approved"
    },
    "origin": {
      "type": "client"
    }
  }
}
```

Example client-required media playback control:

```json
{
  "type": "event",
  "event": "tool.call.started",
  "id": "evt_...",
  "ts": "2026-04-27T10:06:00.000Z",
  "payload": {
    "toolCallId": "tool_media_...",
    "toolName": "media.playback.control",
    "executionMode": "client_required",
    "args": {
      "action": "pause",
      "reason": "user asked CoBuddy to pause the video"
    },
    "visibility": "user_visible",
    "origin": {
      "type": "cobuddy"
    }
  }
}
```

Example delegated-agent tool provenance:

```json
{
  "origin": {
    "type": "delegated_agent",
    "delegationId": "del_...",
    "delegationGroupId": "group_...",
    "agentRunId": "agent_run_...",
    "agentId": "research_agent"
  }
}
```

Delegated-agent tool calls may be surfaced either as top-level `tool.call.*` events with delegated provenance or as payloads inside `delegation.agent.event`. If surfaced inside `delegation.agent.event`, the nested payload should preserve the same `toolCallId` and origin fields.

### `memory.content.used`

Sent when memory content is used in the assistant's reasoning or response. This is a transparency/debug event, not the memory search operation itself. Clients may ignore it.

```json
{
  "type": "event",
  "event": "memory.content.used",
  "id": "evt_...",
  "ts": "2026-04-27T10:03:00.700Z",
  "payload": {
    "memoryId": "mem_...",
    "source": "openclaw",
    "label": "OpenClaw project context",
    "contentPreview": "User believes live context alone is not enough...",
    "confidence": 0.82,
    "visibleToUser": true,
    "cause": {
      "type": "tool_call",
      "toolCallId": "tool_...",
      "toolName": "memory.search"
    },
    "responseId": "resp_..."
  }
}
```

### `memory.content.updated`

Sent when a memory tool changes memory content, including preferences. This is a transparency/debug event. Clients may ignore it or show it in a memory/debug surface.

```json
{
  "type": "event",
  "event": "memory.content.updated",
  "id": "evt_...",
  "ts": "2026-04-27T10:04:00.500Z",
  "payload": {
    "memoryId": "mem_...",
    "source": "cobuddy",
    "operation": "set",
    "memoryType": "preference",
    "contentPreview": "For videos, answer more briefly.",
    "trigger": "user_audio",
    "visibleToUser": true,
    "cause": {
      "type": "tool_call",
      "toolCallId": "tool_...",
      "toolName": "memory.set"
    },
    "responseId": "resp_..."
  }
}
```

Memory content events are not memory operations. They are optional visibility events describing memory content that was used or changed by a generic tool call.

## Delegation Events

Delegation events are for transparency, debugging, and task state. Delegation completion does not create a special report event. If CoBuddy wants to tell the user the result, it should do so through the normal assistant loop: assistant text/audio and conversation state.

Multiple delegated agents may run at the same time. Each delegated agent run must have its own `agentRunId`; several runs may share a `delegationGroupId` if they were launched from one user request.

### `delegation.requested`

```json
{
  "type": "event",
  "event": "delegation.requested",
  "id": "evt_...",
  "ts": "2026-04-27T10:05:10.000Z",
  "payload": {
    "delegationId": "del_...",
    "delegationGroupId": "group_...",
    "target": "research_agent",
    "task": "Find similar products or papers.",
    "requiresPermission": true
  }
}
```

### `delegation.started`

```json
{
  "type": "event",
  "event": "delegation.started",
  "id": "evt_...",
  "ts": "2026-04-27T10:05:20.000Z",
  "payload": {
    "delegationId": "del_...",
    "delegationGroupId": "group_...",
    "agentRunId": "agent_run_...",
    "agentId": "research_agent",
    "status": "running"
  }
}
```

### `delegation.agent.event`

Normalized passthrough event from a delegated agent. These events are primarily for debugging and transparency. Clients may ignore them, show a compact activity view, or expose them in a developer/debug surface.

The server should normalize delegated-agent events to this envelope even if the upstream agent uses OpenClaw, A2A, ACP, MCP, or a custom protocol. The nested `payload` may contain provider-specific details, but the outer envelope should stay stable.

```json
{
  "type": "event",
  "event": "delegation.agent.event",
  "id": "evt_...",
  "ts": "2026-04-27T10:06:20.000Z",
  "payload": {
    "delegationId": "del_...",
    "delegationGroupId": "group_...",
    "agentRunId": "agent_run_...",
    "agentId": "research_agent",
    "agentEvent": "progress",
    "upstreamProtocol": "a2a",
    "payload": {
      "summary": "Found several meeting-agent and memory-agent examples."
    },
    "visibility": "debug_and_user_summary"
  }
}
```

### `delegation.completed`

```json
{
  "type": "event",
  "event": "delegation.completed",
  "id": "evt_...",
  "ts": "2026-04-27T10:07:20.000Z",
  "payload": {
    "delegationId": "del_...",
    "delegationGroupId": "group_...",
    "agentRunId": "agent_run_...",
    "agentId": "research_agent",
    "status": "completed",
    "resultAvailable": true
  }
}
```

## Session State Events

### `session.paused`

```json
{
  "type": "event",
  "event": "session.paused",
  "id": "evt_...",
  "ts": "2026-04-27T10:10:00.000Z",
  "payload": {
    "reason": "user_request"
  }
}
```

### `session.resumed`

```json
{
  "type": "event",
  "event": "session.resumed",
  "id": "evt_...",
  "ts": "2026-04-27T10:11:00.000Z",
  "payload": {
    "captureRestored": ["userAudio", "watchMetadata", "watchTranscript"]
  }
}
```

### `session.ending`

```json
{
  "type": "event",
  "event": "session.ending",
  "id": "evt_...",
  "ts": "2026-04-27T10:30:00.000Z",
  "payload": {
    "options": ["keep_explicit_notes", "save_summary", "export_notes", "forget_session"]
  }
}
```

### `session.ended`

```json
{
  "type": "event",
  "event": "session.ended",
  "id": "evt_...",
  "ts": "2026-04-27T10:30:30.000Z",
  "payload": {
    "retention": "keep_explicit_notes",
    "rawMediaRetained": false
  }
}
```

## State Model

The contract does not use a single exclusive state machine for the whole session. CoBuddy state is a set of parallel state domains reported through `state.updated`.

Minimal session lifecycle values:

```text
closed
opening
open
paused
ending
ended
error
```

Parallel state examples:

- `watch.capture`: `inactive`, `active`, `paused`, `unavailable`
- `watch.playback`: `unknown`, `playing`, `paused`, `seeking`, `ended`
- `user.audioInput`: `muted`, `listening`, `speaking`, `disabled`
- `assistant.output`: `idle`, `speaking`, `interrupted`
- `processing.turn`: `idle`, `thinking`, `responding`
- `processing.memory`: `idle`, `searching`, `updating`
- `delegation.activeRuns`: number of active delegated-agent runs

State notes:

- Assistant speaking does not imply user audio input is disabled. CoBuddy can keep listening while speaking so interruption works.
- Watch-channel capture can remain active while the assistant is speaking or while delegated agents run.
- Delegation is orthogonal task state tracked through `delegation.*` events and summarized through `state.updated`.
- Assistant output follows the active interruption policy.
- Paused means live capture stopped; existing background tasks may continue unless user cancels them.
- Ending means capture stopped and memory-retention choice is pending.

## Reliability Rules

### Ordering

- Each sender maintains monotonic `seq`.
- The server should tolerate missing or duplicated events.
- Side-effecting events should use idempotency keys when needed.

### Backpressure

- Client should throttle high-frequency watch events.
- Transcript deltas should be batched where possible.
- Playback updates should not spam more than necessary.

### Reconnect

If WebRTC disconnects:

- Client enters reconnecting state.
- Server keeps session alive for a short grace period.
- Background tasks may continue.
- Capture is considered inactive until media tracks reconnect.

V0 suggested grace period:

```text
10-30 seconds
```

## Latency Metrics

The contract should support measuring:

- User speech end to assistant audio start.
- User barge-in to assistant audio stop.
- Wake detected to listening active.
- Watch transcript arrival lag.
- Tool call duration.
- Memory content update visibility delay.
- Delegation start latency.
- Delegation completion to next assistant message/audio latency when CoBuddy chooses to notify the user.

Suggested events:

- `metrics.turn`
- `metrics.interruption`
- `metrics.tool_call`
- `metrics.delegation`

OpenClaw's voice-call latency summaries are a useful pattern to borrow.

## Security And Permissions

V0 permission principles:

- No raw media retention by default.
- No full transcript retention by default.
- Explicit notes are saved.
- External sharing requires confirmation by default, represented as a standard `permission.request` tool call.
- OpenClaw context reads should be labeled.
- Sending new CoBuddy session context back to OpenClaw should require permission.
- Client-declared capabilities are claims, not trust anchors.
- Server should enforce allowed sources and scopes.

## Review Loop

This contract should be reviewed in loops.

### Review 1: Product Semantics

Questions:

- Are the states right for the user experience?
- Are indicators sufficient?
- Are watch/user channels clearly separated?
- Are permission-as-tool moments understandable?
- Are tool calls, memory-content updates, and delegation passthrough events visible enough?

Output:

- Update event names and state machine.

### Review 2: Latency And Transport

Questions:

- Which events must ride on WebRTC data channel?
- Which events can use WebSocket/HTTP fallback?
- Should the first data channel be reliable ordered?
- Where do we need unreliable/partial reliability later?
- How do we measure and display latency?
- Which layer should own interruption for v0: client, server, provider, or combined?

Output:

- Transport mapping table.

### Review 3: Implementation Feasibility

Questions:

- Can Chrome extension reliably capture required YouTube context?
- Can client-side transcript extraction be done safely?
- Can the client support full transcript snapshots and watch audio segments/tracks reliably?
- Where should wake phrase detection run?
- Where should barge-in detection run?
- Which client-required tools should v0 support beyond permission and media playback control?
- What does reconnect look like?
- What is the minimum viable signaling API?

Output:

- Prototype milestone list.

### Review 4: Interoperability

Questions:

- How does OpenClaw context read plug in?
- Which events map to A2A/ACP/MCP later?
- What should be public protocol vs private implementation?
- What should third-party clients be required to implement?

Output:

- Public contract subset.

## Open Questions

1. Should the client send `watch.transcript.delta`, or should the server retrieve transcripts from URL?
2. Should wake phrase detection run locally in the extension, on the server, or both?
3. Should text transcript of user speech be sent back to client as `user.transcript.delta` for display/debugging?
4. Should assistant text deltas always mirror audio for accessibility?
5. Should delegation task state have additional aggregate events for groups with multiple concurrent delegated agents?
6. Should memory-content visibility be mandatory for every answer using memory?
7. Should OpenClaw reads require per-session approval or persistent user approval?
8. Should WebRTC data channel carry all events in v0, or should session/delegation events use WebSocket?
9. What should the default interruption mode be when using a realtime model provider that has native interruption/session behavior?
10. Should assistant response lifecycle events be called responses, turns, rounds, or something else?
11. Should server-requested media playback control require user confirmation every time or follow a persistent session permission?

## Current Draft Position

For v0:

- Use WebRTC between Chrome extension and CoBuddy server.
- Send user audio as a WebRTC audio track.
- Receive assistant audio as a WebRTC audio track.
- Send watch metadata/transcript and session events over WebRTC data channel.
- Support full transcript snapshots and watch-channel audio input in v0.
- Keep watch channel and user channel distinct.
- Let server own context fusion and model-provider calls.
- Use standard tool calls for permissions and client-executed actions.
- Use OpenClaw only outside the latency-critical voice path.
- Make interruption configurable and provider-neutral.
