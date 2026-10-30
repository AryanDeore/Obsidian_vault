## Problem

Build Mafia (social deduction game) as a decentralized multi-agent system on Google ADK. Players are independent LLM agents with hidden roles, private memory, and conflicting goals — no central agent dictates their behavior. User configures player count/roles via frontend, then watches the game play out live.

Goal: demonstrate peer-to-peer agent reasoning, hidden state, and deception — not a linear orchestrator-executor pipeline.

## Core Challenges & Approach

|Challenge|Problem|Approach|
|---|---|---|
|Dynamic agent construction|Player count/roles set at runtime; ADK agent hierarchies are normally fixed at construction|Custom `GameMaster(BaseAgent)` builds player agents as a `ParallelAgent` inside `_run_async_impl` at session start, so they stay inside ADK's managed hierarchy instead of being orphaned objects|
|Phase control|Night → reveal → day discussion → vote → elimination must stay deterministic regardless of LLM output|GameMaster owns phase transitions as deterministic logic (ADK 2.0 graph workflow). LLM reasoning is scoped inside each node, never controls phase order|
|Decentralized architecture|No central agent should decide what players say/do|Each player is a peer `LlmAgent` with its own persona/goals. GameMaster only enforces rules + timing, not player behavior|
|Information asymmetry|Hidden roles break if private info leaks into shared context|Public log and private per-agent memory live in separate `session.state` namespaces; each agent's prompt is templated only with public log + its own private slice|
|Believable deception|LLMs default to cooperative/transparent behavior|Persona prompts explicitly encode deception incentives per role; success measured by whether mafia agents survive votes|
|Observability vs. spoilers|Full transparency kills suspense|Stream public events by default; gate full agent-reasoning view behind a debug toggle|

## V1 Scope

**In:** 5–10 configurable players, roles (Mafia, Doctor, Detective, Villager), full day/night loop with voting + win detection, streamed frontend transcript, optional debug view of private agent reasoning.

**Out (later):** human players as a role, cross-game memory/learning, extra roles (Jester, Mayor).

## Milestones

1. Static 5-player game running end-to-end (CLI/playground)
2. Dynamic agent construction from user-submitted config
3. Frontend: setup screen, live transcript, voting UI
4. Debug view + game replay/export
5. Deploy (Agent Runtime / Cloud Run)

## Success Criteria

- Visitor configures + launches a match in <30s, no instructions needed
- Mafia agents win/evade detection meaningfully often (real deception, not random)
- Agent hierarchy stays visible to ADK tracing despite dynamic construction
- Scales 5→10 players with no architecture changes