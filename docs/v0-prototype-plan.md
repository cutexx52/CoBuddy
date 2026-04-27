# CoBuddy V0 Prototype Plan

## Purpose

This plan turns the v0 product spec and WebRTC session contract into an implementation sequence.

The goal is not to build the full product immediately. The goal is to prove the hardest assumptions in order:

1. Chrome extension can capture useful YouTube/watch context.
2. Client and server can maintain a low-latency WebRTC session.
3. Watch channel and user channel can be organized separately.
4. The server can fuse context and produce assistant audio/text.
5. Tool calls can save notes, request permissions, control client media, and run background research.
6. OpenClaw can be connected as optional external context.

## V0 Shape

```text
Chrome Extension Client
  - YouTube/watch context capture
  - User microphone capture
  - Assistant audio playback
  - State indicators
  - Client-required tool execution

WebRTC
  - User audio track
  - Optional watch audio track
  - Assistant audio track
  - Data channel for contract events

CoBuddy Server
  - WebRTC session endpoint/signaling
  - Session timeline
  - Watch/user channel separation
  - Context fusion
  - Tool call orchestration
  - Memory/notes
  - OpenClaw connector
  - Research delegation
  - Model provider adapter
```

## Milestone 0: Repo And Dev Skeleton

Goal:

- Establish project structure without locking frameworks too early.

Deliverables:

- `apps/extension`: Chrome extension client.
- `apps/server`: CoBuddy server.
- `packages/protocol`: shared TypeScript types for contract events.
- `packages/shared`: shared utility code.
- Local dev scripts.

Success:

- Extension and server can run locally.
- Shared protocol types compile.

## Milestone 1: YouTube Watch Context Capture

Goal:

- Prove the extension can capture the watch channel.

Extension should capture:

- Page URL
- Video title
- Channel name
- Current timestamp
- Playback state
- Playback rate
- Full transcript snapshot when available
- Transcript deltas or current transcript window when available
- Watch audio track or audio segment path

Contract events:

- `watch.media.metadata`
- `watch.playback.updated`
- `watch.transcript.snapshot`
- `watch.transcript.delta`
- `watch.audio.segment` or watch audio track metadata

Success:

- Open a YouTube video.
- Extension sends watch context to a local debug panel/server.
- Seek/play/pause updates are visible.
- Full transcript upload works when available.

Key risk:

- YouTube transcript access may vary by page, language, captions, and DOM changes.

## Milestone 2: WebRTC Session Skeleton

Goal:

- Prove Chrome extension client can establish WebRTC with CoBuddy server.

Required:

- Signaling endpoint.
- User audio track client to server.
- Assistant audio track server to client, initially synthetic tone or test audio.
- Reliable ordered data channel `cobuddy.session.v0`.
- Basic `client.ready` and `session.opened`.
- `state.updated` events.

Success:

- Extension starts a session.
- Server receives user audio track.
- Server sends assistant test audio.
- Data channel roundtrip works.
- UI shows connected/listening/speaking states.

Key risk:

- Browser extension permissions and audio routing may be more annoying than normal web app WebRTC.

## Milestone 3: Dual-Channel Timeline

Goal:

- Maintain separate watch and user channels on the server.

Server should store a rolling session timeline:

- Watch metadata/playback/transcript/audio events.
- User audio transcript or placeholder user utterances.
- Assistant responses.
- Tool calls.
- Delegation events.
- Memory content events.

Success:

- Server can answer debug queries like:
  - "What was the latest watch event?"
  - "What user input triggered this response?"
  - "What media timestamp was active when the user spoke?"

Key risk:

- Timing alignment between YouTube position, transcript time, audio capture, and user speech needs careful monotonic timestamps.

## Milestone 4: Voice Loop Prototype

Goal:

- Produce a first real voice interaction.

Preferred fast path:

- Server integrates one realtime model provider behind an adapter.
- Client contract remains provider-neutral.

Provider candidates:

- OpenAI Realtime WebRTC/server adapter for fast proof.
- Gemini Live later for video/multimodal comparison.
- LiveKit/Pipecat evaluation after first proof.

Success:

- User asks by voice: "What do you think about that?"
- Server fuses recent watch context and user utterance.
- Assistant responds by audio and optional text.
- `assistant.response.started`, `assistant.audio.started`, `assistant.text.delta`, `assistant.audio.ended`, `assistant.response.ended` work.
- Interruption policy works at least in one mode.

Key risk:

- Latency. The first spoken response must feel alive.

## Milestone 5: Generic Tool Call Layer

Goal:

- Prove tool calls are generic and client/server/delegated execution modes work.

Implement first tools:

- `notes.create`: server-executed.
- `memory.search`: server-executed.
- `memory.set`: server-executed.
- `permission.request`: client-required.
- `media.playback.control`: client-required.

Success:

- User says "save this thought."
- Server emits `tool.call.started` and `tool.call.completed`.
- Note is saved.
- Client can execute `permission.request`.
- Client can execute or reject `media.playback.control`.

Key risk:

- Tool event visibility can get noisy. Keep UI minimal.

## Milestone 6: Memory Visibility

Goal:

- Show memory use and updates without making memory operations special protocol events.

Implement:

- Basic local memory store for notes/preferences.
- `memory.content.used` when a memory item is used.
- `memory.content.updated` when a memory item changes.
- Debug panel or simple memory activity surface.

Success:

- User says "be shorter for videos."
- Server persists preference through `memory.set`.
- Client receives `memory.content.updated`.
- Future response uses preference.

Key risk:

- Memory visibility can feel creepy if too prominent. Default UI should be subtle.

## Milestone 7: OpenClaw Context Read

Goal:

- Establish optional OpenClaw connection without dependency.

Implement:

- Configurable OpenClaw endpoint.
- Read-only context search first.
- `openclaw.context.search` as a standard tool.
- Label OpenClaw-sourced memory/context through `memory.content.used`.

Success:

- User asks "connect this to my previous project context."
- Server searches CoBuddy local memory and optional OpenClaw context.
- Assistant can cite that OpenClaw context was used.
- CoBuddy works if OpenClaw is disconnected.

Key risk:

- Security and permission boundaries. Start read-only.

## Milestone 8: Research Delegation

Goal:

- Prove async background delegated-agent loop.

Implement:

- One research agent adapter.
- `delegation.requested`
- `delegation.started`
- `delegation.agent.event`
- `delegation.completed`
- Multiple concurrent delegated runs.
- Result delivered through normal assistant response/audio when appropriate.

Success:

- User says "ask research to look deeper."
- CoBuddy asks permission via `permission.request` tool if external context is shared.
- Research runs in background.
- User keeps watching/talking.
- Delegated agent events are visible in debug/activity surface.
- CoBuddy later says through normal assistant response: "The research finished..."

Key risk:

- Avoid turning async research into a separate chat product. It should feel like CoBuddy did it.

## Milestone 9: UX Polish For Trust

Goal:

- Make the session feel understandable and safe.

Required indicators:

- Session open/closed
- User mic listening/muted
- Watch capture active
- Assistant speaking
- Thinking/responding
- Background agents running
- Memory used/updated
- Permission requested
- Tool/client action needed

Success:

- User always knows what CoBuddy is doing.
- User can stop/pause quickly.
- User can see why CoBuddy knows something.

Key risk:

- Too many indicators. Prioritize calm, compact, and inspectable UI.

## Technical Spikes

### Spike A: Chrome Extension Media Capture

Question:

- Can extension capture YouTube transcript, timestamp, playback state, and tab audio reliably?

Output:

- Findings doc and demo.

### Spike B: WebRTC Data Channel Contract

Question:

- Can the extension and server exchange contract events over a reliable data channel while media tracks are active?

Output:

- Minimal session demo.

### Spike C: Provider Latency

Question:

- Which provider path gives the best first voice latency without breaking our server-owned contract?

Paths:

- Server-to-OpenAI Realtime
- Server-to-Gemini Live
- LiveKit/Pipecat route

Output:

- Latency comparison with time-to-first-audio and interruption latency.

### Spike D: OpenClaw Read-Only Context

Question:

- What is the safest smallest OpenClaw integration?

Output:

- Read-only context search demo.

## Build Order Recommendation

Recommended order:

1. Repo skeleton.
2. Chrome extension YouTube watch context.
3. WebRTC session skeleton.
4. Dual-channel timeline.
5. Voice loop.
6. Tool layer.
7. Memory visibility.
8. OpenClaw read path.
9. Research delegation.
10. UX polish.

## First Demo Target

The first meaningful demo:

1. User opens YouTube.
2. User opens CoBuddy extension.
3. CoBuddy shows watch context.
4. User starts session.
5. User asks by voice: "What do you think about that?"
6. CoBuddy answers using the current transcript/timestamp.
7. User says: "Save this thought."
8. CoBuddy saves a note via generic tool call.
9. User says: "Pause the video while you explain."
10. Server requests `media.playback.control`; client pauses video.
11. CoBuddy explains.

This demo proves:

- Watch channel
- User channel
- WebRTC session
- Voice-first interaction
- Generic tool calls
- Client-required tool execution
- Assistant response lifecycle

