# MatchLens AI — Design Doc (draft v2)

**Hackathon:** Microsoft × Premier League — *Inside the Game: Developer Hackathon*
**Category:** Best Multi-Agent Orchestration
**Author:** Helena Pedro · **Status:** DRAFT — topology aligned to the Oct 4 decision outline
**Window:** hacking Oct 6–27, 2026 · submission by Oct 27, 11:59 PM PT

Two decisions are deliberately **not** made in this draft, pending ratification:
the orchestration route (Section 6) and the data model, which waits for the
October 6 gate (Section 7).

## 1. Problem

A football match is a firehose of events — passes, shots, tackles, possession
changes, pressure shifts. Fans, studios, and streamers want that stream turned
into *understanding*: what is happening, why it matters, and what pattern it
reveals — live, in plain language, backed by evidence. A single LLM call does
this shallowly: it reads everything at once, blurs the lenses together, and
cannot show its work.

## 2. Solution (one line)

**One match, read three ways at once, then explained:** three specialized agents
read the synthetic event stream in parallel through different lenses; a fourth
agent names the tactical pattern those reads agree on; a final agent turns the
pattern into an explainable broadcast recap that cites the event evidence.

## 3. Ratified scope (from the Oct 4 outline)

- One match, end to end. Category: Best Multi-Agent Orchestration only.
- Demo under two minutes, showing the parallel reads and the evidence-citing recap.
- **Fallback if hours get tight:** two parallel readers + the two sequential
  steps (pattern, recap). Keep the shape, trim the cast.

## 4. Agent topology

```
                 ┌─► Possession & Transitions Agent ─┐
Event stream ────┼─► Pressing Shape Agent ─────────────┼─► Pattern Agent ─► Recap Composer
 (synthetic)     └─► Player Involvement Agent ────────┘        │
        ▲                                                      │
        └────────── Coordinator (shared state, handoffs, failure recovery)
```

| Agent | Lens / role | Reads → Writes |
|---|---|---|
| **Possession & Transitions Agent** | Who has the ball, how possession changes hands, where attacks are born and die | Event stream → possession chains, transition map, tempo reads |
| **Pressing Shape Agent** | Defensive structure: press height, compactness, triggers | Event stream → pressing reads, shape shifts |
| **Player Involvement Agent** | Who is actually influencing play: touches, involvements, quiet influencers | Event stream → involvement rankings with evidence |
| **Pattern Agent** | Names the tactical pattern the three reads converge on (e.g., "high press forcing long balls into a dominant midfield") | Three reads → one named pattern + supporting evidence |
| **Recap Composer** | Explainable broadcast recap: tells the story of the pattern, cites event evidence, in fan language | Pattern + evidence → recap (EN/PT) |
| **Coordinator** | Owns the match timeline and shared state; dispatches the parallel reads, collects them, hands off to Pattern and Recap; health-checks agents and runs failure recovery | Task queue, `MatchState`, handoff log |

Why parallel reads win this category: the rubric rewards *distinct roles
coordinating through shared state and effective handoffs toward an outcome a
single agent could not achieve as effectively*. Three simultaneous lenses that
must be reconciled is orchestration a judge can see; a linear pipeline looks
like one agent in a trench coat.

## 5. Orchestration design

- **Shared state:** a versioned `MatchState` (match clock, score, the three
  agents' current reads, named pattern, recap draft) owned by the Coordinator.
  Agents read snapshots, write structured reads — never free-text blobs.
  *(Fields finalized only after the Oct 6 gate — Section 7.)*
- **Handoffs:** explicit contracts between Coordinator → readers → Pattern →
  Recap; every handoff logged with input/output hashes so the collaboration is
  auditable in the demo and the README.
- **Failure recovery:** if a reader agent times out or returns an invalid read,
  the Coordinator retries once, then proceeds with the remaining lenses and
  *flags the gap* in the recap instead of hallucinating. Degraded mode is a
  rubric feature, not a bug.

## 6. PENDING RATIFICATION — orchestration route

Microsoft offers two routes; one paragraph must choose between them:

- **Route A — custom code in Azure Container Apps + Foundry model.** Full
  control over the Coordinator, shared state, and handoff logging; more
  infrastructure to own during a short window that also contains the Oct 8
  CS529 midterm.
- **Route B — Foundry Agent Service (managed runtime).** Agents defined in
  Foundry, runtime managed; faster to stand up, less surface for custom
  orchestration mechanics (shared state and handoff logs need care to stay
  visible to judges).

*Decision and one-paragraph rationale to be added here after ratification.*

## 7. October 6 gate — read before designing data

On Oct 6, time-boxed: open the real synthetic dataset, note its fields, format,
and delivery mechanism, and only then design event tables or search indexing.
No data model before this gate. This section gets the dataset notes; Sections 4–5
get adjusted to reality if the data disagrees with them.

## 8. MVP scope

**In:** one synthetic match end-to-end; three parallel lens reads; named tactical
pattern with cited evidence; explainable recap (EN/PT); a UI that makes the
orchestration visible (agents working, handoffs, degraded mode); <2 min demo.
**Out (stretch only):** multi-match support, accounts, live streaming beyond the
provided data cadence, voice commentary.

## 9. Timeline

| Dates | Work |
|---|---|
| Oct 4–5 | This doc ratified; check Azure + Foundry access (no provisioning); create Innovation Studio project; public repo scaffolded |
| Oct 6 | **Gate:** dataset inspection, time-boxed (midterm Oct 8, 10:00–12:00) |
| Oct 7–8 | Midterm first; no build |
| Oct 9–15 | Ingestor + Coordinator + three readers; handoff logging |
| Oct 16–21 | Pattern + Recap; UI; failure-recovery paths |
| Oct 22–25 | Hardening, README, pitch, demo script |
| Oct 26 | Record demo video; submission dry-run |
| Oct 27 | Submit — well before 23:59 PT |

## 10. Demo video plan (<2 min, hard limit)

0:00 the problem in one sentence → 0:20 the three readers working the same events
in parallel → 0:50 the Pattern Agent naming the pattern with evidence → 1:20 the
recap, in two languages → 1:45 architecture in one frame + category fit.
Screen-record the real app; no stock footage, no copyrighted music.

## 11. Submission checklist

- [ ] Project on Innovation Studio, category = Best Multi-Agent Orchestration
- [ ] Public GitHub repo: README (setup, architecture, tech list) + `docs/design.md`
- [ ] Pitch/description on the project page (tech used, problem solved)
- [ ] Demo video <2 min, public URL (YouTube/Vimeo)
- [ ] Synthetic data only — no real Premier League data in repo or video

## 12. Open questions for the Challenge brief

1. Dataset delivery (download/API/stream) and schema? *(Oct 6 gate)*
2. Required or prohibited models/services? Provided Azure credits?
3. Exact submission fields on the project page?
4. Single fixture or multiple matches in the dataset?
