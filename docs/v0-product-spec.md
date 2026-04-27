# CoBuddy V0 Product Spec: Live Media Companion

## Summary

CoBuddy V0 is a live media companion for user-controlled sessions.

The user opens CoBuddy while watching long-form YouTube or browser media. CoBuddy follows the live context, discusses naturally by voice, remembers selected moments when asked, and can run background tasks through tools or specialist agents. When the session ends, live capture stops.

The first product should prove:

> CoBuddy can share a live media moment with the user, connect it to memory, and coordinate useful work without breaking the flow of conversation.

## Target User

The first target user is a multimedia consumer who wants to think with an intelligent companion while consuming long-form content.

Likely early users:

- Podcast listeners
- Long-form YouTube viewers
- Online course learners
- Interview and lecture consumers
- Researchers and creators who consume media for ideas
- People who often have thoughts they want to capture while listening or watching

This is not primarily a coding-agent user. The user may use coding agents or research agents later, but the first relationship is with CoBuddy as a companion.

## First Use Case

The user is watching a long-form YouTube video or similar browser media.

They hear an interesting idea. Instead of pausing, switching apps, copying a link, explaining the context, and opening a separate AI chat, they talk to CoBuddy.

Example flow:

1. User opens CoBuddy from a Chrome extension while watching YouTube.
2. CoBuddy starts following the media context.
3. User says by voice: "Wait, what do you think about that?"
4. CoBuddy answers using the recent media context.
5. User says: "This reminds me of our previous discussion about agent memory. Can you connect it?"
6. CoBuddy searches relevant long-term context and responds conversationally.
7. User says: "Save this idea to my CoBuddy notes and ask the research agent to find related examples."
8. CoBuddy confirms, creates the note, launches the background task, and continues the conversation.
9. Later, CoBuddy reports: "The research agent finished. It found three related examples. Want the short version?"

## Core Jobs

### 0. Organize Watch Channel And User Channel

CoBuddy V0 has two live input channels:

1. Watch channel: the media the user is consuming.
2. User channel: the user's direct interaction with CoBuddy.

For YouTube/browser media, the watch channel may include:

- Video title
- URL
- Channel name
- Timestamp
- Playback state
- Transcript/captions
- Optional sampled audio
- Optional visual frames later

The user channel may include:

- User microphone audio
- User transcript
- Text fallback messages
- Wake phrase or push-to-talk events
- Interruptions
- Preference/customization commands

These channels should remain distinct internally. CoBuddy should fuse them only when building model context, and the fused context should preserve source labels and timestamps.

This prevents common failures:

- Treating media speaker claims as user instructions.
- Confusing user speech with video speech.
- Saving raw media as if it were user memory.
- Sending watch-channel context to external agents without permission.
- Failing to resolve "that" against the correct recent media moment.

### 1. Follow Live Media Context

CoBuddy should understand enough of the current media session to discuss it with the user.

V0 should support:

- Current media title or source when available
- Current timestamp or approximate position when available
- Recent audio/transcript context
- Recent user conversation
- Optional visual context for video when available

The goal is not perfect media understanding. The goal is that the user can ask about "what they just said" without manually explaining it.

### 2. Discuss Naturally By Voice

CoBuddy should feel like a smart companion listening or watching with the user.

It should be able to:

- Explain what was said
- Offer an opinion or counterpoint when asked
- Help the user reason through an idea
- Compare the current idea to prior context
- Ask a lightweight clarifying question when needed
- Stay quiet when the user is just consuming media

The conversation should not feel like a transcript summarizer. It should feel like shared attention.

V0 should be voice-first because the aha moment depends on natural live interaction. Text can remain available for precision, correction, and quiet environments, but pure text would make the experience feel much closer to ordinary chat agents.

### 3. Capture Selected Thoughts

The user should be able to save ideas without leaving the media session.

Examples:

- "Save that."
- "Write this down as an idea."
- "Add this to my notes about personal agents."
- "Make a document from this later."
- "Remember that I disagreed with this argument."

V0 should bias toward explicit saves. CoBuddy should not store every interesting thing by default.

### 4. Use Long-Term Context

CoBuddy should connect live media moments to longer-term user context when useful.

Possible context sources:

- CoBuddy memory
- User notes
- User preferences
- Imported documents
- Session history
- Previous saved ideas
- OpenClaw memory or context when connected
- External memory providers later
- Other local/context systems
- Specialist agents with relevant memory

V0 should include basic CoBuddy memory for notes, preferences, and session working context. It should also establish a connection path to OpenClaw so CoBuddy can read relevant external memory/context when available. CoBuddy should not depend on OpenClaw, but the connection is strategically useful: it validates openness, lets users reuse existing context, and connects CoBuddy to an active agent ecosystem.

V0 does not need perfect memory. It needs a clear model for asking: "What relevant context should I retrieve for this moment, from CoBuddy memory or connected systems?"

### 5. Run Background Tasks

CoBuddy should be able to start tasks while the conversation continues.

Examples:

- Ask a research agent to find related sources
- Create or append to a note
- Draft a summary
- Generate a question list
- Save timestamps and quotes
- Ask another agent to analyze a claim
- Create a follow-up task

CoBuddy should track task state and report completion naturally.

V0 should include two first background task paths:

1. Save note: simple, reliable, and immediately useful for extending live context into user-owned memory.
2. Research delegation: harder, slower, and more differentiated because it proves CoBuddy can send context to a specialist agent and report back fluently.

Good behavior:

- "I'll look into that in the background."
- "I saved the note."
- "The research agent is still working."
- "It finished. Want the short version now or later?"

Bad behavior:

- Blocking the conversation until every task completes
- Flooding the user with status messages
- Starting tasks without consent
- Losing track of delegated work

## Session Model

CoBuddy V0 should be session-based.

Session states:

- Closed: no live capture or live context following
- Open: CoBuddy can follow selected live context
- Paused: session exists, but live capture is temporarily stopped
- Ending: user decides what to save, summarize, forget, or export

The user should always know whether CoBuddy is listening, watching, or idle.

V0 should support simple commands:

- "Start listening."
- "Pause."
- "Resume."
- "Stop."
- "Forget this session."
- "Save the useful parts."
- "Summarize this session."

Wake words or push-to-talk may be important interaction options. For hands-free media, exercise, cooking, or gaming sessions, a wake phrase can make CoBuddy feel naturally available without forcing the user to switch apps. V0 should treat wake activation as a user-controlled input mode, not as permission for permanent ambient capture.

CoBuddy should also provide clear audio and visual indicators for session state:

- Listening
- Thinking
- Speaking
- Background task running
- Background task completed
- Paused
- Muted
- Not capturing

These indicators are not decorative. They are part of trust and interaction quality.

## Memory Behavior

V0 memory should be conservative and explicit.

Default memory policy:

- Do not save raw live media by default.
- Do not save full transcripts by default unless the user opts in.
- Save user-approved notes, ideas, and preferences.
- Save lightweight session summaries only with user permission or clear preset.
- Keep short-term working context during the open session.
- Allow the user to forget the session.

Memory types:

- User preference: "answer briefly," "no spoilers," "be more challenging."
- Saved idea: a user-approved thought or note.
- Session summary: optional summary of the live session.
- Source reference: media title, link, timestamp, or context pointer.
- Delegated task record: what was asked, what context was shared, what came back.
- External context reference: pointer to OpenClaw or another connected memory/context source when used.

Open question:

- What is the simplest useful storage format for V0 memory: local file, database, or both?

## Personality

Default personality:

- Warm
- Smart
- Helpful
- Lightly humorous
- Curious
- Respectful of silence
- Not overly intimate
- Not overly verbose

CoBuddy should adapt through conversation:

- "Be more concise."
- "Challenge me more."
- "For this podcast, be more opinionated."
- "For movies, do not interrupt."
- "Use a calmer voice."

Personality should affect tone and behavior, not only wording.

It should influence:

- How often CoBuddy speaks
- How much detail it gives
- Whether it asks follow-up questions
- How it reports background tasks
- How it handles uncertainty
- How it uses humor

## Customization

Customization should be conversational first.

The user should not need to open a large settings dashboard to personalize CoBuddy.

V0 customization examples:

- "Only remember what I explicitly ask you to remember."
- "When I say 'research this,' ask my research agent."
- "During videos, answer in short form."
- "During podcasts, feel free to discuss more."
- "Always ask before sending context to another agent."

CoBuddy can maintain inspectable learned rules, but the first interface should be natural language.

## Openness

V0 should preserve the open direction even if not every open feature is fully built.

Design commitments:

- Protocol-oriented session interface
- Import/export for saved notes and memory records
- Bring-your-own model/API-key path eventually
- Specialist agent interface for delegation
- OpenClaw memory/context connection path
- External memory provider interface later
- Self-hostable architecture eventually

V0 can start with one high-quality preset, but should not paint the product into a closed architecture.

## Privacy And Control

CoBuddy deals with intimate live context. Trust is part of the product.

V0 should make clear:

- What is being captured
- Whether audio, video, screen, or transcript context is active
- What is saved
- What is shared with external tools or agents
- What can be deleted or forgotten

Required controls:

- Start session
- Pause session
- Stop session
- Forget session
- Save selected memory
- Confirm before sharing context with external agents
- Clear audio or visual indicators for capture, mute, task state, and completion

## Non-Goals For V0

V0 should not try to do everything.

Non-goals:

- Full always-on life logging
- Perfect long-term memory
- Full mobile, desktop, Chrome, iOS, and Android support at once
- Meeting assistant parity with Granola, Otter, or Fireflies
- Gaming overlay support
- Movie/series spoiler intelligence
- Full self-hosting polish
- Full protocol standardization
- Complex multi-agent marketplace
- Heavy customization dashboard

These may matter later, but they are not required to prove the first product.

## Success Criteria

V0 is successful if users feel:

- "It understood what I was listening to or watching."
- "I could discuss ideas without stopping the media flow."
- "It remembered what I explicitly wanted saved."
- "It connected this moment to something I already cared about."
- "It got background work started without making me manage another agent."
- "It reported back naturally when the work finished."
- "I trusted when it was listening and what it saved."

Behavioral success:

- User opens multiple sessions per week.
- User saves thoughts during sessions.
- User asks follow-up questions grounded in live media.
- User delegates at least one background task.
- User returns to saved notes or task results later.

## Key Risks

### Novelty Risk

Users may try media companionship once and not return. V0 needs repeated value: saved thoughts, useful follow-up, memory connections, and async research.

### Context Quality Risk

If CoBuddy misunderstands the media too often, the experience collapses. V0 should pick media contexts where transcript/audio access is reliable.

### Interruption Risk

If CoBuddy talks too much, it ruins the media experience. Conservative proactivity is important.

### Memory Trust Risk

If CoBuddy saves too much or makes memory hard to inspect, users may lose trust.

### Delegation Complexity Risk

Background tasks are a key differentiator, but too many agent integrations can delay v0. Start with one or two clear task types.

## Suggested V0 Slice

The smallest credible V0:

- Chrome extension as the first client surface
- YouTube or browser long-form media as the first media source type
- Voice-first conversation, with text fallback
- Session start/pause/stop
- Recent transcript/audio context
- Explicit note saving
- Simple memory search over saved notes and preferences
- OpenClaw memory/context read path when configured
- Research delegation as the first hard background task integration
- Natural task completion report

Candidate first slice:

> Chrome extension for long-form YouTube companion with voice conversation, notes, OpenClaw memory/context read path, and one research-agent delegation path.
