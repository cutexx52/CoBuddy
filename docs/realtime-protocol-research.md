# CoBuddy Realtime Protocol And Stack Research

Research date: 2026-04-27

## Executive Take

CoBuddy should not bet on one protocol for everything.

The product has at least five different communication problems:

1. Low-latency audio/video transport for live voice.
2. Multi-channel live context organization for watch/media streams and user streams.
3. Session state and interaction events for indicators, interruptions, wake state, memory use, and task status.
4. Model realtime APIs for speech-to-speech or voice pipelines.
5. Agent/tool protocols for async background work, memory lookup, and delegation.

Trying to force all four through ACP, A2A, WebSocket, or a model vendor API would make the system brittle.

My current recommendation:

> Use WebRTC or a WebRTC-based framework for latency-critical media. Define a CoBuddy session/event contract for product-specific state. Use OpenClaw/A2A/ACP/MCP-style protocols only for background tools, memory, and agent delegation.

In other words:

- **Realtime media path:** optimized for latency.
- **Live context organization path:** optimized for aligning watch-channel and user-channel data.
- **CoBuddy session path:** optimized for product semantics.
- **Agent delegation path:** optimized for interoperability and async work.

## Important Naming Clarification

There are multiple "ACP" meanings in the market.

### Agent Communication Protocol

The BeeAI/IBM ecosystem uses **Agent Communication Protocol** for agent interoperability. Its official docs describe it as an open protocol for connecting agents, applications, and humans through a standardized RESTful API. IBM's overview emphasizes REST-based communication, async-first operation, and no SDK requirement.

Sources:

- [Agent Communication Protocol docs](https://agentcommunicationprotocol.dev/)
- [IBM: What is Agent Communication Protocol?](https://www.ibm.com/think/topics/agent-communication-protocol)

### Agent Client Protocol

Another ACP is **Agent Client Protocol**, which is JSON-RPC based and focused on connecting clients such as editors to coding agents. It is useful to study for agent/client interaction patterns, but it is not designed for CoBuddy's voice/media session problem.

Sources:

- [Agent Client Protocol overview](https://agentclientprotocol.com/protocol/overview)
- [Agent Client Protocol GitHub](https://github.com/agentclientprotocol/agent-client-protocol)

### Practical Conclusion

ACP may be useful for agent interoperability, but it is not enough for CoBuddy's realtime companion session. CoBuddy needs its own session contract for live context, audio indicators, wake state, memory visibility, and async task reporting.

## Layer 1: Realtime Media Transport

### WebRTC

WebRTC is the strongest default for low-latency browser voice and possible video.

Why it matters:

- Built for realtime audio/video.
- Supports browser media capture.
- Handles echo cancellation, jitter, packet loss adaptation, NAT traversal, and data channels.
- Works well for Chrome extension/browser-first v0.
- Can carry data-channel events alongside media.

Official/relevant sources:

- [MDN WebRTC API](https://developer.mozilla.org/docs/Web/API/WebRTC_API)
- [MDN WebRTC data channels](https://developer.mozilla.org/en-US/docs/Games/Techniques/WebRTC_data_channels)
- [OpenAI Realtime API with WebRTC](https://platform.openai.com/docs/guides/realtime-webrtc)
- [LiveKit voice agents guide](https://livekit.com/voice-agents)

OpenAI explicitly recommends WebRTC rather than WebSockets for browser/mobile client connections to its Realtime API when performance consistency matters. LiveKit also calls WebRTC the "gold standard" for realtime audio transport and argues WebSockets/HTTP streaming add too much delay for natural conversation.

Tradeoffs:

- More complex than WebSocket.
- Needs signaling, ephemeral credentials, ICE/STUN/TURN concerns.
- Debugging can be harder.
- Browser extension permissions and audio routing need careful design.

CoBuddy view:

> WebRTC should be the default media path for v0 voice.

## Layer 1.5: Multi-Channel Live Context

CoBuddy's input side has at least two distinct live channels:

1. **Watch channel:** the media or environment the user is consuming or sharing.
2. **User channel:** the user's direct interaction with CoBuddy.

For v0 YouTube:

- Watch channel includes YouTube video title, URL, timestamp, playback state, transcript/captions, optional sampled audio, and later optional visual frames.
- User channel includes microphone audio, user transcript, text fallback, wake/push-to-talk events, interruptions, and preference commands.

These channels must remain separate in the internal model even when they are fused for reasoning.

Why this matters:

- The model must know who said what: media speaker vs user vs CoBuddy.
- User intent should have higher priority than passive media context.
- The system should avoid treating media claims as user instructions.
- Permission rules differ: user voice, media transcript, and external memory should not have the same sharing policy.
- Timing matters: user asks "what did they mean by that?" and CoBuddy must resolve "that" against the watch channel's recent timeline.
- Memory rules differ: explicit user thoughts may be saved; raw media should not be saved by default.

CoBuddy should organize live input as timestamped, typed context events:

- `watch.media.metadata`
- `watch.playback.updated`
- `watch.transcript.delta`
- `watch.audio.segment`
- `watch.video.frame`
- `user.audio.segment`
- `user.transcript.delta`
- `user.text.submitted`
- `user.wake.detected`
- `user.interrupt`
- `user.preference.command`

The session layer should maintain a rolling context window with source attribution:

- Recent watch-channel content
- Recent user-channel utterances
- Recent CoBuddy responses
- Active memory/context snippets
- Active background task state

The model should receive fused context, but the fusion should preserve source labels and timestamps.

Example:

```text
[watch 00:13:21 speaker:podcast_guest] "Personal agents need long-term memory, otherwise they are just chatbots."
[user 00:13:28] "Wait, what do you think about that?"
[memory source:cobuddy_note] "User believes live context alone is not enough; tools and open agent ecosystems matter."
```

CoBuddy view:

> The watch channel and user channel should be separate streams at ingestion, aligned in a session timeline, then selectively fused for model context.

### WebSocket

WebSocket is useful for control/events and some server-side model connections. It is not ideal as the main browser audio transport if we care about extreme voice latency and robustness.

Good uses:

- Session events
- Transcript events
- Task status
- Memory query status
- Browser extension to backend control messages
- Server-to-model connections where vendor requires it

Weak uses:

- Raw live audio/video transport from browser when WebRTC is available
- Jitter-sensitive voice conversations

Sources:

- [Gemini Live API overview](https://ai.google.dev/gemini-api/docs/live-api)
- [OpenAI Realtime API reference](https://platform.openai.com/docs/api-reference/realtime)

CoBuddy view:

> WebSocket is a good control plane, not the primary voice media plane.

### WebTransport

WebTransport is interesting for future low-latency data over HTTP/3/QUIC, especially reliable and unreliable streams/datagrams. It is not yet the obvious v0 choice because WebRTC is more mature for browser audio/video and supported by model/voice-agent infrastructure.

CoBuddy view:

> Track WebTransport for future data-heavy low-latency streams, but do not use it as v0 foundation.

## Layer 2: Realtime Model APIs

### OpenAI Realtime API

OpenAI Realtime supports realtime models over WebRTC, WebSocket, and SIP. Its voice-agent docs describe two architectures: direct speech-to-speech sessions for natural low-latency conversation, or chained STT → LLM → TTS pipelines for more explicit control.

Strengths:

- Strong voice-first path.
- Browser WebRTC path is officially supported.
- Good fit for fast v0.
- Data channel can carry realtime events.
- SDK support through OpenAI Agents SDK.

Risks:

- Vendor lock-in if we build too directly against it.
- Cost and rate limits.
- Model behavior and audio personality may be less controllable than a pipeline.
- Tool/background task semantics need CoBuddy's own orchestration around the session.

Sources:

- [OpenAI Realtime API with WebRTC](https://platform.openai.com/docs/guides/realtime-webrtc)
- [OpenAI Realtime API reference](https://platform.openai.com/docs/api-reference/realtime)
- [OpenAI Voice agents guide](https://platform.openai.com/docs/guides/voice-agents)

CoBuddy view:

> Best candidate for fastest voice-first prototype, but wrap it behind a provider interface.

### Gemini Live API

Gemini Live API supports low-latency realtime voice/video interactions and continuous streams of audio, video, or text. Google documents WebSocket usage and partner integrations over WebRTC/WebSockets through LiveKit, Pipecat, Agora, Firebase, and others. Vertex AI announced GA for Gemini Live API with native audio in late 2025.

Strengths:

- Strong multimodal story: audio, text, and video input.
- Native audio models reduce chained pipeline latency.
- Google ecosystem and video understanding are strategically relevant.
- LiveKit plugin exists.

Risks:

- API maturity and reliability need testing.
- WebSocket/browser proxy patterns may add latency if implemented poorly.
- Community reports mention latency or audio issues in some setups; treat as anecdotal but worth testing.

Sources:

- [Gemini Live API overview](https://ai.google.dev/gemini-api/docs/live-api)
- [Google Cloud: Gemini Live API native audio in Vertex AI](https://cloud.google.com/blog/topics/developers-practitioners/how-to-use-gemini-live-api-native-audio-in-vertex-ai)
- [LiveKit Gemini Live API plugin](https://docs.livekit.io/agents/models/realtime/plugins/gemini/)
- [Google DeepMind: Gemini 2.5 native audio](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/gemini-2-5-native-audio/)

CoBuddy view:

> Important second provider to test, especially for video-aware sessions. Do not assume it is lower-latency than OpenAI without measuring.

### Chained STT → LLM → TTS Pipeline

This is the traditional voice-agent architecture.

Strengths:

- More control over each stage.
- Easier to swap models.
- Easier to inspect transcripts, apply memory retrieval, and manage tools.
- Can self-host some pieces.

Risks:

- Latency can accumulate across STT, LLM, TTS, network hops, VAD, and tool calls.
- More operational complexity.
- More tuning required for interruptions and endpointing.

Recent research and community reports suggest sub-second or roughly one-second time-to-first-audio is possible with careful tuning, but not automatic.

Sources:

- [OpenAI Voice agents guide](https://platform.openai.com/docs/guides/voice-agents)
- [LiveKit voice agents guide](https://livekit.com/voice-agents)
- [Realtime voice agents tutorial paper](https://arxiv.org/abs/2603.05413)

CoBuddy view:

> Better long-term modularity, but likely slower than direct realtime model APIs for first v0.

## Layer 3: Realtime Agent Frameworks

### LiveKit

LiveKit is a mature WebRTC stack and has an Agents framework for realtime voice/video AI. It supports realtime models and chained voice pipelines. It has plugins for providers like Gemini Live and can be self-hosted or used as cloud infrastructure.

Strengths:

- WebRTC infrastructure is production-oriented.
- Handles rooms, participants, tracks, audio routing, network adaptation.
- Works across browser/mobile.
- Supports multi-provider agents.
- Good fit if CoBuddy eventually needs multiple clients, sessions, and media tracks.

Risks:

- Another infrastructure layer.
- Can add complexity before product-market proof.
- Latency still depends on model/provider path and deployment region.

Sources:

- [LiveKit voice agents](https://livekit.com/voice-agents)
- [LiveKit voice AI quickstart](https://docs.livekit.io/agents/start/voice-ai/?pipeline-type=Realtime+model)
- [LiveKit Gemini Live API plugin](https://docs.livekit.io/agents/models/realtime/plugins/gemini/)
- [LiveKit: Why WebRTC beats WebSockets for voice AI agents](https://livekit.com/blog/why-webrtc-beats-websockets-for-voice-ai-agents)

CoBuddy view:

> Strong candidate if we want provider flexibility and self-host/open-source credibility early. Direct OpenAI WebRTC may be faster for prototype; LiveKit may be better as the session/media backbone.

### Pipecat + RTVI

Pipecat is an open-source framework for realtime voice and multimodal agents. RTVI defines a client-server message standard for realtime voice/video inference, with events for speaking state, transcription, LLM processing, TTS, and function calls.

Strengths:

- RTVI is close to the product surface CoBuddy needs.
- Has explicit events for speaking, transcription, LLM function calls, TTS state, and metrics.
- Open-source and provider-pluggable.
- Useful reference for CoBuddy's interaction contract.

Risks:

- Another framework dependency.
- We may still need CoBuddy-specific events for media context, memory visibility, OpenClaw context, task delegation, and session lifecycle.

Sources:

- [RTVI introduction](https://docs.pipecat.ai/server/frameworks/rtvi/introduction)
- [RTVI standard](https://docs.pipecat.ai/client/rtvi-standard)
- [Pipecat GitHub](https://github.com/pipecat-ai/pipecat)

CoBuddy view:

> Study RTVI carefully. It is probably the closest existing event-contract reference, but CoBuddy will still need its own product-level session semantics.

### Daily / Vapi / Retell / Managed Voice Platforms

Managed platforms can speed up prototypes, especially telephony or support-agent style voice. For CoBuddy, they may be less ideal because openness, self-hosting, model flexibility, and client-specific media context are core.

CoBuddy view:

> Useful for learning and benchmarking, but probably not the core foundation.

## Layer 4: Agent And Tool Interoperability

### OpenClaw Gateway

OpenClaw's gateway protocol is WebSocket-based and exposes a broad gateway API for clients, nodes, sessions, agents, approvals, models, config, and more. OpenClaw also has strong open/self-hosted positioning, many channels, many skills, and community energy.

Strengths:

- Strong ecosystem signal.
- Self-host and open-source alignment.
- Can provide external memory/context and specialist action.
- Gateway protocol already models roles, scopes, capabilities, permissions, sessions, and nodes.

Risks:

- Security concerns are real. Recent reporting has highlighted exposed deployments, broad permissions, RCE risk, and unsafe workstation usage.
- We should connect narrowly and permissionfully, not blindly.
- CoBuddy should not depend on OpenClaw uptime or local installation.

Sources:

- [OpenClaw home](https://openclawdoc.com/)
- [OpenClaw Gateway Protocol](https://docs.openclaw.ai/gateway/protocol)
- [TechRadar: exposed OpenClaw deployments](https://www.techradar.com/pro/security/the-math-is-simple-openclaw-trojan-horse-ai-agents-give-hackers-full-control-of-28-000-systems)
- [TechRadar: OpenClaw workstation risk](https://www.techradar.com/pro/security/microsoft-says-openclaw-is-unsuited-to-run-on-standard-personal-or-enterprise-workstation-so-should-you-be-worried)

CoBuddy view:

> Connect to OpenClaw as an optional memory/context and action provider, but design a strict permission boundary and never make it part of the latency-critical voice path.

### OpenClaw Voice And Talk Mode Lessons

OpenClaw has several voice-related patterns worth learning from, but it does not appear to define the exact contract CoBuddy needs for low-latency browser WebRTC plus watch/user dual-channel context.

Relevant OpenClaw surfaces:

- **Talk Mode:** a continuous voice conversation loop: listen for speech, send transcript to the main session, wait for response, speak through a configured Talk provider.
- **Voice Call plugin:** phone-call style voice via providers such as Twilio, Telnyx, Plivo, and mock. It supports outbound notifications, multi-turn inbound conversations, streaming transcription, TTS, call status, and latency summaries.
- **Gateway protocol:** WebSocket JSON frames with request/response/event framing, roles, scopes, node capabilities, permissions, session control, talk config, talk mode, TTS config, voice wake config, and session messaging.

What OpenClaw gets right:

- **Explicit phase states:** Talk mode models Listening → Thinking → Speaking. This maps directly to CoBuddy indicators.
- **Interruption support:** Talk mode can stop playback when the user starts speaking. CoBuddy needs barge-in from day one.
- **Voice provider abstraction:** Talk/TTS and Voice Call let providers be configured and swapped.
- **Wake-word configuration:** Gateway exposes `voicewake.get`, `voicewake.set`, and `voicewake.changed`.
- **Permission and capability model:** Gateway clients/nodes declare roles, scopes, capabilities, commands, and granular permissions.
- **Session control methods:** OpenClaw has session create/send/steer/abort/patch/reset/delete/compact style controls.
- **Security posture:** The Voice Call plugin documents webhook signature verification, replay protection, unauthenticated socket limits, per-IP caps, and stale-call reaping.
- **Spoken-output contract:** Voice Call appends a strict JSON spoken-output contract and defensively extracts caller-facing speech to avoid leaking planning/meta text into TTS.
- **Latency instrumentation:** Voice Call has a latency command that summarizes turn latency and listen-wait percentiles from logs.

What does not transfer directly:

- **Transport:** Gateway is WebSocket control-plane oriented. Voice Call is telephony-provider oriented. CoBuddy v0 should use WebRTC for browser voice/media.
- **Input model:** OpenClaw Talk sends a user transcript into `chat.send`; CoBuddy needs at least two live input channels: watch channel and user channel.
- **Context fusion:** OpenClaw voice appears centered on user speech as a normal message. CoBuddy must align user utterances with media transcript/audio/video, timestamps, memory snippets, and background task state.
- **Latency target:** OpenClaw's talk/call modes are useful, but CoBuddy's companion feel depends on stricter realtime turn-taking and interruption latency.
- **Product semantics:** OpenClaw does not model CoBuddy-specific events such as `watch.transcript.delta`, `memory.source.used`, `delegation.context_share.confirmed`, or `report.ready`.

Sources:

- [OpenClaw Talk Mode](https://docs.openclaw.ai/nodes/talk)
- [OpenClaw Voice Call plugin](https://docs.openclaw.ai/plugins/voice-call)
- [OpenClaw Gateway Protocol](https://docs.openclaw.ai/gateway/protocol)
- [OpenClaw client gateway protocol mirror](https://open-claw.bot/docs/gateway/protocol/)

CoBuddy view:

> Borrow OpenClaw's explicit state model, interruption behavior, provider abstraction, voice/wake config, permissions, and latency instrumentation. Do not copy its voice transport or chat-message-centered contract for CoBuddy's realtime session.

### A2A

A2A is an open protocol for agent-to-agent interoperability. Its spec uses JSON-RPC 2.0 over HTTP and Server-Sent Events for streaming.

Strengths:

- Good fit for async agent delegation and task state.
- Agent discovery/capability patterns are relevant.
- More appropriate for background research agents than for realtime voice.

Risks:

- Not optimized for sub-second voice interaction.
- Agent ecosystem still evolving.
- Identity and permission concerns remain active research areas.

Sources:

- [A2A Protocol specification](https://a2a-protocol.org/dev/specification/)
- [Linux Foundation A2A announcement PDF](https://s24.q4cdn.com/538403808/files/doc_news/Linux-Foundation-Launches-the-Agent2Agent-Protocol-Project-to-Enable-Secure-Intelligent-Communication-Between-AI-Agents-2025.pdf)
- [AIP paper: identity across MCP and A2A](https://arxiv.org/abs/2603.24775)

CoBuddy view:

> Good candidate for delegated research/task agents, not the live session protocol.

### MCP

MCP standardizes model/tool/data access using JSON-RPC with stdio and Streamable HTTP transports.

Strengths:

- Very strong ecosystem for tool access.
- Good fit for local tools, memory providers, note systems, and external services.
- CoBuddy can expose or consume MCP servers.

Risks:

- Tool protocol, not agent companionship protocol.
- Not designed for realtime audio/video.
- Tool security and prompt-injection boundaries must be strict.

Sources:

- [MCP basic specification](https://modelcontextprotocol.io/specification/2025-11-25/basic)
- [MCP transports](https://modelcontextprotocol.io/specification/draft/basic/transports)

CoBuddy view:

> Use MCP for tools and memory providers, not voice/session transport.

## Candidate Architecture Options

### Option A: Direct OpenAI Realtime WebRTC First

Chrome extension connects to CoBuddy backend for ephemeral credentials, then uses OpenAI Realtime WebRTC directly. CoBuddy backend handles notes, OpenClaw, research tasks, and memory.

Pros:

- Fastest path to a voice-first aha moment.
- Strong browser WebRTC support.
- Less infra to operate.

Cons:

- Vendor-specific session model.
- Harder to support Gemini/other realtime providers cleanly.
- CoBuddy event contract must wrap/normalize OpenAI events.

Best for:

- Fast proof of v0 experience.

### Option B: LiveKit As Media/Session Backbone

Chrome extension joins a LiveKit room. CoBuddy agent runs as a participant/service. The agent uses OpenAI Realtime, Gemini Live, or a chained pipeline behind LiveKit.

Pros:

- More provider-flexible.
- Good media/session infrastructure.
- Better path to mobile, multi-client, self-host, and multiple tracks later.

Cons:

- More infrastructure.
- More setup and deployment complexity.
- Must measure latency carefully.

Best for:

- Open/self-hostable architecture with multi-provider future.

### Option C: Pipecat/RTVI As Realtime App Framework

Use Pipecat and RTVI as the voice/multimodal application layer, adding CoBuddy-specific memory/task/media events.

Pros:

- Event model is close to our interaction contract.
- Open source and modular.
- Better control over pipeline behavior.

Cons:

- Framework maturity and fit need hands-on evaluation.
- May still need LiveKit or another transport underneath.

Best for:

- Product-event contract inspiration and modular voice stack.

### Option D: Raw WebSocket Pipeline

Chrome extension streams audio to backend over WebSocket; backend handles STT, LLM, TTS and streams audio back.

Pros:

- Simple conceptually.
- Full backend control.

Cons:

- More latency risk.
- Rebuilds media features WebRTC already solves.
- Harder to make robust under real network conditions.

Best for:

- Internal experiments, not production v0 voice.

## Recommended Direction For CoBuddy V0

I recommend a two-track prototype:

### Track 1: Fast Aha Prototype

Use:

- Chrome extension
- YouTube transcript/timestamp/page context
- OpenAI Realtime WebRTC for voice
- CoBuddy backend for notes, memory, OpenClaw read path, research task
- WebSocket or WebRTC data channel for CoBuddy session events

Goal:

- Prove the live media conversation and async background task loop.

### Track 2: Open Architecture Evaluation

In parallel or immediately after:

- Test LiveKit as the session/media backbone.
- Test Gemini Live via LiveKit plugin or direct WebSocket.
- Study RTVI as an interaction event model.
- Design CoBuddy Session Protocol independent of any one model provider.

Goal:

- Avoid getting trapped in one vendor API.

## CoBuddy Session Protocol: Why We Still Need It

Even if we use OpenAI Realtime, Gemini Live, LiveKit, RTVI, ACP, A2A, or MCP, none of them fully model CoBuddy's product semantics.

CoBuddy needs events such as:

- `session.opened`
- `session.paused`
- `session.ended`
- `capture.source.enabled`
- `media.context.updated`
- `voice.wake.detected`
- `agent.listening`
- `agent.thinking`
- `agent.speaking`
- `memory.lookup.started`
- `memory.lookup.result`
- `memory.source.used`
- `note.saved`
- `delegation.requested`
- `delegation.context_share.confirmed`
- `delegation.started`
- `delegation.progress`
- `delegation.completed`
- `delegation.failed`
- `report.ready`
- `user.preference.learned`

This protocol can run over WebRTC data channel or WebSocket. The transport can change; the product contract should remain stable.

## Latency Principles

For CoBuddy, latency is not a nice-to-have. It is product identity.

Principles:

1. Do not route live audio through agent protocols.
2. Use WebRTC for browser voice when possible.
3. Keep model/provider close to the user region where possible.
4. Minimize server hops in the voice path.
5. Separate realtime conversation from background tasks.
6. Let background tasks be slow without blocking voice.
7. Support barge-in/interruption from day one.
8. Measure time-to-first-audio, end-of-turn detection, interruption latency, and task-report latency separately.
9. Prefer direct speech-to-speech for first magic; keep pipeline path for future control.
10. Treat transcript/context retrieval as parallel work, not serial blocking work.

## Open Questions To Argue

1. Should v0 use direct OpenAI Realtime WebRTC first, or LiveKit first?
2. Should CoBuddy's first session event path be WebRTC data channel or WebSocket?
3. Should we adopt RTVI message names where possible or define a clean CoBuddy-specific event vocabulary?
4. Should Gemini Live be tested before or after the first OpenAI prototype?
5. What is the minimal OpenClaw integration: read memory only, list agents, or delegate tasks too?
6. How strict should context-sharing confirmation be before sending media-derived context to external agents?

## Current Bet

My current bet:

> Build the first Chrome/YouTube v0 with OpenAI Realtime WebRTC for speed, define a provider-neutral CoBuddy Session Protocol immediately, and evaluate LiveKit + Gemini + RTVI before hardening architecture.

This gives us a fast product proof without surrendering the open architecture.
