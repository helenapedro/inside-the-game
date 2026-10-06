# MatchLens AI: Multi-Agent Match Intelligence

**One match, read three ways at once, then explained.**

MatchLens AI is a team of specialized AI agents built for the Microsoft × Premier League *Inside the Game: Developer Hackathon* (category: **Best Multi-Agent Orchestration**). It turns synthetic football match event data into explainable match intelligence and an evidence-citing broadcast recap.

## The idea

Three agents read the same match event stream **in parallel**, each through a different lens: possession & transitions, pressing shape, and player involvement. A Pattern Agent reconciles those reads and names the tactical pattern; a Recap Composer turns it into an explainable recap that cites the event evidence. A Coordinator owns the shared match state, the handoffs between agents, and failure recovery, so the orchestration itself is visible, auditable, and resilient.

## Status

Design phase. Hacking window: Oct 6–27, 2026. See [docs/design.md](docs/design.md) for the full design.

## Tech

Microsoft Foundry (hosted agents) · Azure AI Services · GitHub Copilot

---

Built by [Helena Pedro](https://github.com/helenapedro)
