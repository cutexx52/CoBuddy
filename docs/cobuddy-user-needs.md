# CoBuddy: Vision And User Needs

CoBuddy is a realtime companion agent that can share live experience with the user across work, media, play, movement, learning, entertainment, conversations, and devices during user-controlled sessions. The long-term ambition is broad: when the user opens a session, the agent should be able to understand what the user is seeing, hearing, saying, typing, and asking, then respond privately and helpfully through text or voice until the user closes it.

This document intentionally focuses on goal, ambition, vision, and real user needs. Implementation details, technical constraints, and architecture choices should come later.

## Vision

CoBuddy should feel like an agent that lives with the user.

Most agents are powerful when the user packages a task and sends it to them. CoBuddy should be different. It should be present during sessions the user chooses to open: listening to a podcast, watching a video, exercising, playing a game, watching a movie or series, following a livestream, sitting through a meeting, reading a page, seeing a screen, hearing the user's voice, receiving an image, or coordinating with other agents in the background.

The core idea is live context connected to long-term context and external action.

CoBuddy does not only hold static context such as memory, files, profile, project history, and preferences. It also holds the current stream of experience: what is happening now, what changed recently, what the user is paying attention to, what the user might want to ask next, and what other agents may need to know in order to help.

Live context alone is not enough. Without broader memory, external tools, and fluent access to other agents, CoBuddy would become another realtime conversation assistant like Gemini Live or ChatGPT Realtime. The product needs live context so it can be present, long-term context so it can understand the user over time, and open agent/tool connectivity so it can act beyond the conversation.

The product should make the user feel:

- "It is here with me."
- "It understands what is happening right now."
- "I can talk to it naturally while I am doing something else."
- "It can quietly coordinate with other tools and agents without pulling me out of the moment."
- "It remembers the right things, forgets the right things, and lets me control the boundary."
- "I can open or close its presence whenever I want."

## Ambition

CoBuddy aims to become a live companion layer for human-computer interaction, not only a productivity agent.

It should be:

- Media-rich: fluent with audio, video, images, text, screens, and streams.
- Live: grounded in what is happening now, not only what was stored before.
- Memory-aware: able to use broader long-term context through user-owned memory, imports, connected systems, or external ecosystems.
- Companion-like: able to talk, listen, think together, and stay nearby without always demanding a task.
- Action-capable: able to use tools, external systems, and device capabilities when the user wants something done.
- Agent-connected: able to coordinate with existing agents asynchronously in the background.
- Personally adaptable: able to learn preferences through conversation rather than forcing users through a large settings dashboard.
- Client-agnostic: able to work across devices and surfaces without being trapped inside one app.
- Protocol-oriented: open enough that clients and agents can talk to CoBuddy through stable interfaces.

The distinguishing feature is not "another agent that can use tools" or "another realtime voice assistant." The distinguishing feature is an open agent that continuously consolidates live context, long-term context, and external action into useful companionship, answers, memory, and delegated work.

CoBuddy should be useful when the user is trying to produce something, but also when the user is solo and simply doing something: exercising, watching movies or series, following online streams, playing games, learning casually, or thinking out loud. Productivity is a mode, not the identity.

## What Makes CoBuddy Different

CoBuddy is built from four inseparable layers:

1. Live context: what the user is seeing, hearing, saying, typing, watching, or doing now.
2. Long-term context: what the user has saved, imported, connected, delegated, or built over time.
3. Companion interaction: the ability to talk, listen, stay quiet, answer privately, and think with the user in the right mode.
4. Open action: the ability to use tools, external systems, local gateways, and other agents through interoperable protocols.

If CoBuddy has only live context, it becomes a realtime conversation assistant. If it has only memory, it becomes a lifelogging or recall product. If it has only tools, it becomes an automation agent. If it has only emotional presence, it becomes a companion app. The product becomes distinct only when these four layers work together inside user-controlled sessions.

## Product Belief

People increasingly live inside streams: podcasts, videos, live events, meetings, chats, browser tabs, games, workouts, mobile apps, desktop apps, and agent workspaces. The hard part is not only answering a question or completing a task. The hard part is being present in the same context as the user, at the same time, with enough memory and permissions to help or accompany without forcing the user to stop what they are doing.

CoBuddy should feel like a private, context-aware companion layer that can join the moment with the user and coordinate outside the moment when needed.

## Core User Needs

### 1. I Need The Agent To Share The Current Moment With Me

The user may be watching, listening, reading, working, browsing, attending, speaking, or thinking. They should not have to manually compress the situation into a prompt before the agent can help.

Examples:

- "What is happening here?"
- "Why did they say that?"
- "Remember this part."
- "What should I pay attention to?"
- "Connect this to what we discussed earlier."
- "Can you keep listening while I check something else?"

The key need is shared live context. CoBuddy should reduce the burden of explaining the moment.

### 2. I Need Someone To Think With Me While I Consume Media

The user may be listening to a podcast, watching a video, following a course, reading a talk transcript, watching a movie, or following a livestream. They want to pause, ask, explore, challenge, or connect ideas without losing context.

Examples:

- "Explain what they mean by this."
- "Do you agree with that argument?"
- "How does this connect to what I was reading yesterday?"
- "Save this idea for later."
- "Turn this into a research question."
- "What should I look up next?"

The key need is shared attention. CoBuddy is not only a summarizer; it is a thinking partner inside the media experience.

### 3. I Need Company While I Am Solo

The user may be exercising, walking, cooking, playing a game, watching a movie, watching a series, following a streamer, or spending time alone. They may not want a productivity tool. They may want company, reactions, explanations, encouragement, lightweight conversation, or someone to share the moment with.

Examples:

- "Watch this with me."
- "Talk to me while I work out."
- "What just happened in this episode?"
- "Help me think through this game strategy."
- "Keep me company while I cook."
- "Don't spoil anything, just explain what I missed."

The key need is companionship during ordinary life. CoBuddy should not require every interaction to become a task.

### 4. I Need A Natural Live Conversation With The Agent

The user should be able to speak or type to CoBuddy while doing something else. In many scenarios, voice is the natural interface: podcast listening, walking, cooking, watching videos, browsing casually, or brainstorming. In other scenarios, text is safer: meetings, classrooms, public spaces, or situations where the user is talking to other people.

Examples:

- "Wait, explain that."
- "I disagree. Help me think it through."
- "Keep this in mind but do not interrupt."
- "Answer in text only for this meeting."
- "Talk to me while we watch this."

The key need is mode awareness. CoBuddy should understand when to speak, when to stay quiet, when to answer privately, and when to work in the background.

### 5. I Need Private Help While I Am In A Live Social Situation

The user may be in a meeting, lecture, interview, sales call, livestream, or shared screen session. They need to ask private questions without interrupting the live situation.

Examples:

- "What did they just decide?"
- "What should I say next?"
- "Can you summarize the objections?"
- "Find the document they mentioned."
- "Draft a reply I can say out loud."
- "Tell me if I am missing something important."

The key need is not transcription. It is private, realtime situational support.

### 6. I Need Continuity Across Devices And Contexts

The user may start on macOS, continue on iOS, send an image from Android, or use a Chrome extension while another agent is working in the background. They should not have to re-explain the situation each time.

Examples:

- A meeting starts on desktop, but follow-up happens on mobile.
- A user asks about a podcast on their phone, then later asks the desktop agent to turn notes into a doc.
- A Chrome extension observes a web video while the main agent coordinates with existing coding or research agents.

The key need is continuity of context and intent, not identical UI everywhere.

### 7. I Need The Agent To Work With My Existing Agents

Users will already have specialized agents: coding agents, research agents, workflow agents, personal knowledge agents, browser agents, and local automation agents. CoBuddy should coordinate with them instead of replacing them.

Examples:

- "Ask my coding agent to open an issue from this meeting decision."
- "Send this research question to my research agent."
- "Have my local desktop agent pull up the file."
- "Use the OpenClaw gateway to invoke a device action."

The key need is interoperability. CoBuddy should be a realtime context and orchestration layer that can delegate to capable agents through open protocols.

### 8. I Need My Existing Context To Come With Me

The user may already have context scattered across files, notes, chats, calendars, codebases, browser history, recordings, transcripts, task systems, local memories, and other agent ecosystems. CoBuddy should not require the user to start from zero.

Examples:

- "Use my previous notes when we discuss this talk."
- "Import this archive as background context."
- "Connect to my local agent memory."
- "Ask OpenClaw what it knows about this project."
- "Remember this only inside the project context."

The key need is context portability. CoBuddy should support user-controlled import, export, and reuse of existing memory systems rather than becoming a closed memory silo.

### 9. I Need Trust, Control, And Social Safety

CoBuddy may hear meetings, see screens, process user audio, and observe media. This is intimate. The user needs clear control over what is being captured, where it is sent, what is stored, and what other agents can access.

Examples:

- "Listen only to this meeting, but do not store it."
- "Use the transcript for the next hour, then forget it."
- "Do not send raw audio to external agents."
- "Show me what context you are using."
- "Mute yourself for this segment."

The key need is not just privacy policy compliance. It is felt control.

## Primary Use Cases

### Media Companion

CoBuddy listens to or watches media with the user: podcasts, videos, courses, movies, live streams, or talks. It can explain, discuss, summarize, connect to prior context, and help the user turn passive consumption into active thinking.

Success looks like:

- The agent can answer questions grounded in the current media moment.
- The user can naturally voice chat or text chat while consuming media.
- The agent can remember user-selected insights and ignore unimportant noise.
- The experience works across mobile, desktop, and browser surfaces.

### Solo Companion

CoBuddy accompanies the user during solo activities such as exercising, cooking, gaming, watching movies or series, following livestreams, or spending time alone. It can talk, listen, react, encourage, explain, stay quiet, or help the user feel less alone without forcing the interaction into productivity.

Success looks like:

- The agent can match the activity: energetic during workouts, quiet during films, tactical during games, conversational during casual media.
- The user can set boundaries conversationally: "no spoilers," "only talk when I ask," "keep me motivated," or "be casual."
- The agent can use live context without dominating the experience.
- The session can end cleanly without turning every moment into saved memory.

### User-Controlled Live Companion Session

CoBuddy stays available inside a user-opened session as a live context layer. The user can speak, type, send images, share video, or let CoBuddy observe media and device context. CoBuddy can answer immediately, hold thoughts for later, or quietly ask other agents to work in the background. When the user closes the session, live capture and live presence stop.

Success looks like:

- The user can interact without packaging a full prompt.
- The agent can distinguish active user intent from background context.
- The agent can consolidate live context into useful memory, notes, questions, or tasks.
- The agent can remain present without becoming noisy or intrusive.
- The user clearly controls when the session starts, pauses, resumes, and ends.

### Meeting Copilot

CoBuddy joins or observes a live meeting with the user. In this mode, the user is primarily talking to other people, not to the agent. CoBuddy should usually stay quiet and answer through private text unless the user explicitly chooses another mode.

Success looks like:

- The user can ask privately by text without disrupting the meeting.
- The agent can refer to recent meeting context accurately.
- The user can request actions during or after the meeting.
- The agent respects capture, storage, and delegation boundaries.

## Interaction Inputs And Outputs

CoBuddy should eventually accept:

- Media video stream
- Media audio stream
- User image input
- User text input
- User video stream
- User audio stream

CoBuddy should eventually produce:

- Text stream
- Audio stream

This does not mean every client must support every modality on day one. The protocol should make modality support explicit so clients and agents can declare what they can send, receive, and process.

## Client Targets

The intended client surfaces are:

- iOS
- Android
- Chrome extension
- macOS

The first design goal should be client agnostic behavior. Clients are capture and interaction surfaces. The core agent should not be tightly coupled to one UI, OS, or runtime.

## Agent Interoperability

CoBuddy should define an open protocol for other agents and clients to talk to it. The protocol should support realtime context sharing, user intent messages, tool or agent delegation, capabilities discovery, permissions, and streaming responses.

Candidate interoperability directions:

- ACP-compatible agent communication
- A2A-compatible agent communication
- OpenClaw Gateway integration for device and local agent actions
- A CoBuddy protocol layer for realtime multimodal context where existing protocols are insufficient

The protocol should not exist for elegance. It should exist so any capable client or agent can join the user's context safely and predictably.

## Design Principles

### Live Context Is The Product Core

Static context tells the agent who the user is and what has mattered before. Live context tells the agent what is happening now. CoBuddy's product identity should come from combining both intelligently.

### Long-Term Context Makes Live Context Useful

Live context lets CoBuddy understand the current moment. Long-term context lets it understand why the moment matters to this user. CoBuddy should be able to use long-term memory through its own memory, user import/export, external systems, and connected agent ecosystems.

### Openness Is A Product Requirement

CoBuddy should be open because the user's context and agents will not all live inside CoBuddy. The product should support interoperable protocols, context portability, and external agent/tool access. A closed CoBuddy would be less useful and easier for platform assistants to absorb.

### Presence Must Be User-Controlled

CoBuddy should not be framed as permanently always-on. The user opens, pauses, resumes, and closes live sessions. During an open session, CoBuddy may feel "always there" in that context, but the boundary is explicit and controlled by the user.

### Customization Should Happen Through Conversation

CoBuddy should be deeply personal without requiring users to configure a large dashboard. Users should be able to say things like "be more concise," "only remember things I explicitly save," "answer in text during meetings," or "ask my research agent when I say look deeper." CoBuddy can expose inspectable rules and presets, but the primary customization interface should be conversation.

### Start With Shared Context

The agent should know what moment the user is in: meeting, podcast, video, live stream, work session, or general ambient session. Every feature should improve the agent's ability to understand, preserve, and act within that context.

### Make Privacy A Product Surface

Controls for capture, storage, sharing, and delegation should be visible and simple. Privacy should not be hidden in settings or documentation.

### Keep The Core Agent Decoupled

The core agent should expose protocol-level capabilities and not depend on a specific client. Clients should adapt device-specific capture, permissions, and UX into the core protocol.

### Treat Other Agents As First-Class Collaborators

CoBuddy should route work to specialized agents when useful. It should track what was delegated, what context was shared, and what came back.

### Optimize For Real-Time Use Before Perfect Recall

The first magic moment is answering the user while the situation is happening. Long-term memory matters, but realtime grounding is the wedge.

### Do Not Treat Productivity As The Whole Product

Productivity is important, but it is only one environment where live context matters. CoBuddy's broader ambition is to live alongside the user across media, work, entertainment, movement, play, and thought. In meetings, the agent may be mostly silent and text-based. In media, solo, and play contexts, the agent can be more conversational and voice-native.

## Open Questions

- What is the smallest first user scenario that proves the product is real?
- Should the first prototype focus on podcasts, browser video, solo media, gaming, workouts, or another live context scenario?
- What context should be ephemeral by default, and what should be saved only by explicit user action?
- What long-term context should CoBuddy own, import, request from other agents, or leave external?
- How should CoBuddy expose "what I know right now" to the user?
- How should CoBuddy decide whether to speak, answer in text, stay quiet, or delegate work?
- How should users customize behavior through conversation, and how should CoBuddy show the rules it learned?
- What does the session lifecycle look like: open, pause, resume, close, save, forget?
- What should the import/export model be for memories, notes, transcripts, and project context?
- What is the minimum protocol surface for client-to-agent communication?
- What is the minimum protocol surface for agent-to-agent delegation?
- Which existing protocols can we adopt directly, and where do we need a CoBuddy-specific realtime extension?
- What should "connected to existing agents" mean in v0: discovery, invocation, shared memory, streaming collaboration, or all of them?

## Proposed Discussion Path

1. Pick the first real user scenario.
2. Define the user's exact moment-by-moment workflow.
3. Identify required context inputs for that workflow.
4. Define the minimum agent responses and actions.
5. Define privacy and control requirements.
6. Derive the first protocol shape from the workflow.
7. Only then choose client implementation order.

## Candidate First Scenarios

The first scenario should prove that CoBuddy is not just another task agent. It should prove live shared context.

### Podcast Or Video Companion

The user listens to a podcast or watches a video. CoBuddy follows along, lets the user ask voice or text questions, discusses ideas, remembers selected moments, and can ask other agents to research, summarize, or turn ideas into artifacts.

Why this is strong:

- Voice interaction feels natural.
- The agent can be conversational without social risk.
- The media-rich distinction is immediately visible.
- Live context is essential: the current timestamp, audio, visual scene, and recent discussion all matter.

### Solo Activity Companion

The user opens CoBuddy while exercising, cooking, gaming, watching a movie or series, or following a livestream. CoBuddy adapts to the activity: it can motivate, react, explain, strategize, answer questions, or simply stay quietly present.

Why this is strong:

- It proves CoBuddy is not only a productivity agent.
- It makes companionship emotionally legible.
- Voice interaction feels natural in many solo activities.
- It creates a different relationship than task-first agents.

### Browser Media Companion

The user watches a video, livestream, course, or recorded talk in the browser. CoBuddy observes the media stream and page context, then answers questions or coordinates with other agents.

Why this is strong:

- A Chrome extension can be a focused first client.
- Browser media gives a bounded surface for live context.
- The agent can combine media, page metadata, transcript if available, and user questions.

### Desktop Live Session Companion

The user opens CoBuddy during a desktop session while working, watching, listening, or talking on macOS. CoBuddy is available as a live context layer and companion until the user pauses or closes the session. It can observe selected streams, answer through voice or text, and delegate background work.

Why this is strong:

- It best matches the long-term ambition.
- It shows continuity across activities.
- It may be harder to constrain for a first proof.

### Meeting Companion

The user is in a live meeting. CoBuddy listens privately, answers text prompts, tracks important context, and can delegate follow-up work. This is valuable, but likely not the first expression of the full CoBuddy vision because the user is primarily interacting with other people rather than the agent.
