# CoBuddy Product Shape

This document captures the current product alignment.

## Primary Identity

CoBuddy should feel more like a companion than a coordinator or operating layer.

It can coordinate tools and agents. It can become an interface to external systems. But the user's primary relationship should be:

> I open CoBuddy when I want an intelligent companion to share this live moment with me.

The agent should not feel like a dashboard, command center, or productivity console. It should feel like a capable companion that can also get things done.

## First Target User

The first target user is not primarily a developer or coding-agent user. It is a multimedia consumer who wants to think, discuss, remember, and act while consuming media.

The seed scenario:

> The user is listening to a podcast. The speakers are discussing something interesting. The user cannot join the conversation, but they want a highly intelligent friend who can listen with them, discuss the topic, challenge ideas, connect it to memory, and note down instinctive thoughts into documents or other tools.

This user may consume:

- Podcasts
- Videos
- Courses
- Movies and series
- Livestreams
- Gaming content
- News and commentary
- Long-form interviews
- Educational media

This is a different persona from coding agents. Coding agents optimize for implementation. CoBuddy should optimize for shared context, thought, companionship, memory, and smooth handoff to tools or agents.

## First Unforgettable Moment

The first unforgettable moment should combine live discussion with background action.

Example:

1. The user is listening to a podcast.
2. CoBuddy is listening in the same session.
3. The user says: "Wait, I have a thought. Is this similar to what we discussed last week?"
4. CoBuddy answers conversationally using the live podcast context and relevant long-term memory.
5. The user says: "Save this as a note and ask my research agent to look deeper."
6. CoBuddy continues the conversation naturally while the research task runs in the background.
7. Later, CoBuddy reports back fluently: "The research agent finished. It found three related papers and one disagreement with the speaker's claim. Want the short version?"

This is different from today's voice agents because the conversation does not stop at realtime chat. CoBuddy can launch background asynchronous tasks, track their state, and report back naturally when done.

## Proactivity

Default behavior should be conservative.

CoBuddy should mostly wait for the user to invite interaction, especially at the beginning. Over time, it can adapt through conversation.

Good default:

- Quietly follow live context during an open session.
- Answer when asked.
- Ask permission before saving memory or starting external tasks.
- Report completed background tasks at natural moments.
- Learn user preferences through lightweight conversation.

Bad default:

- Interrupt too often.
- Save too much.
- Assume every moment is productive.
- Turn media consumption into homework.
- Trigger tools or agents without clear user intent.

## Memory Direction

Memory is central, but it should be designed carefully.

CoBuddy should distinguish:

- Live session context: what is happening now.
- Short-term working context: what matters inside the current session.
- User preferences: stable interaction style and boundaries.
- Selected memories: user-approved notes, facts, ideas, and moments.
- External memories: context from OpenClaw, files, notes, other agents, databases, and imports.
- Consolidated memories: summaries or abstractions created from repeated patterns over time.

CoBuddy should not assume it owns all memory. It should support its own memory, user import/export, and external memory providers or ecosystems.

Relevant product patterns to study include Hermes-style persistent memory, curated memory files, session search, memory providers, and memory injection. The lesson is not that CoBuddy should copy Hermes. The lesson is that memory should be modular, inspectable, searchable, and improvable over time.

## Main Agent And Specialist Agents

Users should not have to manage many companion agents with separate long-term memories.

CoBuddy should be the main interface agent that holds the broadest user context. Other agents can be specialists:

- Coding agents
- Research agents
- Browser agents
- Local device agents
- Workflow agents
- Media analysis agents
- Knowledge-base agents

The user relationship stays with CoBuddy. CoBuddy decides when to ask a specialist agent, what context to share, how to track progress, and how to explain the result back to the user.

This creates a clean mental model:

> One companion, many specialists behind it.

## Personality

CoBuddy should ship with a strong default personality:

- Warm
- Helpful
- Intelligent
- Lightly humorous
- Respectful of boundaries
- Calm under uncertainty
- Able to be quiet

Users should be able to alter the personality through conversation:

- "Be more concise."
- "Be more playful."
- "Don't joke during serious topics."
- "Challenge me more."
- "Be gentle when I am working out."
- "For movies, only speak when I ask."

Live APIs and TTS now make emotional tone more controllable, so personality should be part of the product surface. But it should serve the user, not trap the user into an artificial relationship.

## Openness

CoBuddy should be open in the broad sense:

- Open protocols
- Self-deploy option
- Different client implementations
- Bring-your-own API keys
- External memory providers
- Import/export for context and memory
- Integration with local systems such as OpenClaw
- Integration with agent protocols such as ACP or A2A where appropriate

At the same time, the default experience should be excellent without configuration.

The product should offer:

- A great preset for users who do not want to change anything.
- Conversational customization for users who want personal behavior.
- Deep configuration and self-hosting for advanced users.

This is how CoBuddy can be open without becoming complicated.

## Product Boundary

CoBuddy is broad in vision, but it should not start as "an agent for everything."

The best early expression is:

> A companion for live media sessions that can discuss, remember selected moments, use long-term context, and delegate background work.

From there, it can expand into:

- Solo activity companion
- Browser media companion
- Gaming companion
- Workout companion
- Meeting side-channel companion
- Desktop live session companion
- Cross-device personal context companion

The final ambition is broad personal agency. The first product should make that ambition feel real in one narrow, emotionally obvious moment.

