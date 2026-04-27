# CoBuddy V0 User Journey

## Scenario

The user is watching a long-form YouTube video in Chrome. They open CoBuddy because they want an intelligent companion to discuss the video, save thoughts, and run follow-up research without leaving the media flow.

## Journey Summary

1. User opens YouTube video.
2. User opens CoBuddy from the Chrome extension.
3. CoBuddy shows the current media context it can see.
4. User starts a live session.
5. CoBuddy enters listening mode with clear indicators.
6. User asks a voice question about the current moment.
7. CoBuddy answers with recent media context.
8. User asks CoBuddy to connect the idea to memory.
9. CoBuddy searches local/CoBuddy memory and optional OpenClaw context.
10. User asks CoBuddy to save a note.
11. CoBuddy saves the note with source context.
12. User asks CoBuddy to research the idea.
13. CoBuddy confirms context sharing and delegates the task.
14. User keeps watching and chatting.
15. CoBuddy reports task completion naturally.
16. User ends the session and chooses what to save or forget.

## Detailed Flow

### 1. Open Media

The user opens a long-form YouTube video.

CoBuddy should be able to detect:

- Page URL
- Video title
- Channel name
- Current timestamp
- Playback state
- Available transcript/captions if accessible

If transcript/captions are unavailable, CoBuddy should say so clearly and fall back to audio capture/transcription if the user allows it.

### 2. Open CoBuddy

The user clicks the CoBuddy Chrome extension.

The extension opens a compact companion panel or overlay. It should not block the media.

The panel shows:

- Session state: closed
- Media detected: title and source
- Capture options: transcript, audio, optional visual context
- Start button
- Text fallback input
- Settings or preferences entry point, kept secondary

The product should avoid a large configuration surface here. The first screen should make the next action obvious: start a live session.

### 3. Start Session

The user starts a session.

Possible inputs:

- Click start
- Press hotkey
- Say wake phrase if enabled

CoBuddy confirms:

- "I'm listening with this video."
- Or, in text-only mode: "Session started. I can follow the video context now."

Indicators should show:

- Listening/capture active
- What sources are active: transcript, audio, page context
- Whether external memory or OpenClaw context is connected

### 4. Ask About The Current Moment

The user asks by voice:

> "Wait, what do you think about that?"

CoBuddy should infer "that" from recent media context.

Good response:

> "They are arguing that personal agents only become useful when they can carry context across tools. I mostly agree, but the missing piece is trust: if the agent remembers too aggressively, people may stop using it."

Bad response:

> "Can you clarify what you mean by that?"

Clarification is acceptable only when recent context is genuinely insufficient.

### 5. Discuss And Think Together

The user continues:

> "Is this similar to what we discussed before about live context?"

CoBuddy searches relevant memory:

- Saved CoBuddy notes
- User preferences
- Prior session summaries if allowed
- OpenClaw memory/context if connected

CoBuddy responds:

> "Yes. This connects to your earlier point that live context alone is not enough. You argued CoBuddy needs long-term context and tools, otherwise it becomes another realtime chat agent."

The response should say when it is using memory:

> "I found this in your saved CoBuddy notes..."

or:

> "OpenClaw has related project context..."

This creates trust and helps the user understand where answers come from.

### 6. Save A Note

The user says:

> "Save this thought."

CoBuddy asks a short confirmation only if needed:

> "Saved. I titled it 'Live context is not enough.'"

The note should include:

- User thought
- CoBuddy-generated title
- Source video title
- URL
- Timestamp
- Optional short context excerpt or summary
- Tags or project if inferred with confidence

If uncertain, CoBuddy can ask:

> "Save this under CoBuddy product notes?"

The save should not interrupt the session.

### 7. Delegate Research

The user says:

> "Ask the research agent to find similar products or papers."

CoBuddy prepares a task with:

- User request
- Recent media context
- Saved note or summary
- Relevant memory references
- Source URL/timestamp

Before sending context externally, CoBuddy confirms:

> "I'll share the saved note, the video title, and the last few minutes of context with the research agent. Is that okay?"

If the user says yes, CoBuddy starts the task:

> "Research started. I'll keep listening and tell you when it finishes."

The task should run asynchronously.

### 8. Continue Session While Task Runs

The user continues watching.

CoBuddy should:

- Keep following current media context
- Allow normal discussion
- Avoid repeated task updates
- Show subtle task-running indicator

If the user asks:

> "Is the research done?"

CoBuddy answers with state:

> "Still running. It has found a few memory and meeting-agent examples, but it has not finished the comparison yet."

### 9. Report Completion

When the research task finishes, CoBuddy should report at a natural moment.

Default behavior:

- Wait until the user finishes speaking.
- Avoid interrupting an intense media moment if possible.
- Use a short notification sound or visual indicator.
- Ask whether the user wants the summary now.

Example:

> "The research agent finished. Short version: it found three close categories, but none combine live media context, long-term memory, and background agent delegation in quite this way. Want the highlights?"

If the user says yes, CoBuddy summarizes.

If the user says later, CoBuddy stores the result and marks it available.

### 10. End Session

The user ends the session.

CoBuddy enters ending state and asks what to do:

> "Session stopped. Do you want to save a short summary, keep only your notes, or forget the session?"

Options:

- Save summary
- Keep only explicit notes
- Export notes
- Forget session
- Continue later

Default should be conservative:

- Keep explicit notes.
- Do not save full transcript unless user opted in.
- Do not retain raw audio/video.

## Key UI States

### Closed

CoBuddy is not capturing live context.

User can:

- Start session
- Review saved notes
- Adjust basic preferences

### Open Listening

CoBuddy is following selected live context.

Indicators:

- Listening
- Media source active
- Transcript/audio active
- External memory connected or disconnected

### Thinking

CoBuddy is processing a user question.

Indicators:

- Thinking animation or audio cue
- Optional short text status

### Speaking

CoBuddy is responding.

Indicators:

- Speaking state
- Stop or interrupt control

### Background Task Running

CoBuddy has delegated work.

Indicators:

- Task running icon/status
- Task name
- Optional progress if available

### Paused

Session exists, but live capture is stopped.

User can:

- Resume
- Stop
- Save/forget

### Ending

Live capture has stopped. User decides memory outcome.

## Voice Interaction Rules

Default rules:

- Voice-first during active session.
- Text fallback always available.
- User can interrupt CoBuddy while it is speaking.
- CoBuddy should keep answers short unless asked to go deeper.
- CoBuddy should not speak proactively except for user-approved task completion or important session-state changes.
- Wake phrase or push-to-talk should be user-controlled.

Example commands:

- "Hey CoBuddy, explain that."
- "Save this."
- "Ask research to look deeper."
- "Pause."
- "Resume."
- "Stop listening."
- "Forget this session."
- "Be shorter."
- "Don't interrupt."

## Memory Rules

Default rules:

- Keep working context only during the session.
- Save explicit notes.
- Save preferences the user clearly expresses.
- Ask before saving summaries.
- Ask before sharing context with external agents.
- Allow user to forget the session.
- Label memory sources when used.

CoBuddy should make memory visible enough that the user can trust it.

## OpenClaw Connection Journey

If OpenClaw is not connected:

- CoBuddy can still run with its own basic memory and notes.
- The extension may show "Connect OpenClaw context" as an optional enhancement.

If OpenClaw is connected:

- CoBuddy can ask OpenClaw for relevant project/user context.
- CoBuddy should label when OpenClaw context was used.
- CoBuddy should respect user permissions before sharing new session context back.

The OpenClaw connection should feel like an interoperability benefit, not a dependency.

## Failure Cases

### No Transcript Available

CoBuddy says:

> "I cannot access captions for this video. I can use audio transcription if you allow it."

### Weak Context

CoBuddy says:

> "I may need a bit more context. Are you referring to the speaker's claim about memory?"

### Research Agent Fails

CoBuddy says:

> "The research task failed before completing. I saved the request, so we can retry later."

### User Interrupts

CoBuddy stops speaking immediately and listens.

### User Stops Session

CoBuddy stops capture immediately.

## Journey Success

This journey succeeds if the user feels:

- Starting a session is easy.
- Voice interaction feels natural.
- CoBuddy understands "that" from the media context.
- Saving an idea is effortless.
- Research delegation feels useful without taking over the session.
- Completion reporting feels fluent.
- Memory and capture boundaries are clear.

