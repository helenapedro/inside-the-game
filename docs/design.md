# MatchLens AI: Design Doc (draft v2)

**Hackathon:** Microsoft × Premier League, *Inside the Game: Developer Hackathon*
**Category:** Best Multi-Agent Orchestration
**Author:** Helena Pedro · **Status:** DRAFT (topology aligned to the Oct 4 decision outline)
**Window:** hacking Oct 6–27, 2026 · submission by Oct 27, 11:59 PM PT

One decision is deliberately **not** made in this draft: the data model waits
for the October 6 gate (Section 7).

## 1. Problem

A football match is a firehose of events: passes, shots, tackles, possession
changes, pressure shifts. Fans, studios, and streamers want that stream turned
into *understanding*: what is happening, why it matters, and what pattern it
reveals, live, in plain language, backed by evidence. A single LLM call does
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
  Agents read snapshots, write structured reads, never free-text blobs.
  *(Fields finalized only after the Oct 6 gate; see Section 7.)*
- **Handoffs:** explicit contracts between Coordinator → readers → Pattern →
  Recap; every handoff logged with input/output hashes so the collaboration is
  auditable in the demo and the README.
- **Failure recovery:** if a reader agent times out or returns an invalid read,
  the Coordinator retries once, then proceeds with the remaining lenses and
  *flags the gap* in the recap instead of hallucinating. Degraded mode is a
  rubric feature, not a bug.

## 6. Orchestration route: Route C (hosted agents), ratified Oct 5

Microsoft offers three shapes for this. **Route A** is custom code in Azure
Container Apps with a Foundry model: the most infrastructure control, and the
most to provision and operate in a three-week window that also contains a
midterm. **Route B** is prompt-defined Foundry agents: the least to manage and
the weakest fit, because MatchLens's core (a deterministic Coordinator,
versioned MatchState, logged handoffs, failure recovery) is custom
orchestration logic, not prompts. **Route C** is hosted agents in Foundry Agent
Service: the same custom Python code packaged as a container image (or source
Foundry builds into one), with Foundry running it behind a managed endpoint
with automatic scaling, a dedicated Entra identity, session-level state
persistence, and end-to-end observability.

**Decision: Route C, with Route A as the fallback.** One person, three weeks,
and a midterm in the middle argue for spending the hours on the orchestration
the rubric pays for, not on operating Container Apps, a container registry, and
separate hosting. The Coordinator, MatchState, handoff contracts, and failure
recovery stay in code we control; Foundry takes hosting, scaling, identity,
session persistence, and observability. If a hosted limit blocks the design,
we fall back to Route A and the Coordinator code moves unchanged.

Conditions and caveats, kept attached to the decision:

- This stands **only if the access check passes**: active Azure
  subscription; Foundry Project Manager on an existing project (or Owner at
  resource-group scope); Azure Developer CLI 1.27.1+ with the
  microsoft.foundry extension and an authenticated azd session; an
  authenticated Azure CLI session; Python 3.13+; an existing Foundry project
  with a deployed chat model and available quota. The quickstart's example
  model is an example, not a requirement.
- Hosted billing adds container compute on top of inference and scales per
  active session; keep the sandbox small (hosted sandboxes run 0.5 vCPU /
  1 GiB to 2 vCPU / 4 GiB). Versions are immutable once created, so every
  resource or environment change is a new version to retest. Hosted agents
  support Python and C# only; Python is our language, so this does not bind.
- Session state persistence stores application state; it does **not** design
  how three parallel readers share one MatchState. The contract work in
  Sections 4 and 5 remains ours.
- Nothing is provisioned (no Container Apps, Cosmos DB, or registry stack)
  before the dataset is seen on Oct 6.

## 7. Data gate: resolved Oct 9 (no provided dataset; we generate ours)

Original plan was to inspect a provided dataset on Oct 6 before designing any
data model. That gate is now resolved by a negative finding: Helena checked
Innovation Studio on Oct 9 and found no dataset file, and the Challenge brief
supplies none either. The brief's wording points the same way: projects must
use synthetic, football-realistic data, no real Premier League data is
redistributed, and item (v) explicitly invites creating or extending synthetic
datasets. So the dataset is ours to generate, and the ingestor consumes an
event stream we define.

Proposed synthetic event schema (to ratify Oct 12; tuned so every reader lens
and every brief feature has the fields it needs):

- `event_id`, `match_id`, `period`, `match_clock_seconds`
- `event_type`: pass, shot, tackle, possession_change, pressure, foul,
  kickoff, goal
- `team_id`, `player_id` (stable across the match)
- `x`, `y` (0-100 pitch coordinates), `end_x`, `end_y` if applicable
- `outcome` (successful/unsuccessful/intercepted), `possession_id`
- Speed and distance inputs: pass distance derived from coordinates; ball
  and shot speed generated within realistic ranges
- Match metadata: team names, formations, a pre-set tactical script (which team
  presses high, how momentum shifts) so the Pattern Agent has a real pattern
  to find and the recap has something true to say

Generator shape: a deterministic, seeded simulator emitting one full match as
an ordered event log (JSON lines), fast enough to replay in real time or
accelerated for the demo. This doubles as brief item (v), data innovation,
and makes the demo reproducible.

## 8. MVP scope

**In:** one synthetic match end-to-end; three parallel lens reads; named tactical
pattern with cited evidence; explainable recap (EN/PT); a UI that makes the
orchestration visible (agents working, handoffs, degraded mode); <2 min demo.
**Out (stretch only):** multi-match support, accounts, live streaming beyond the
provided data cadence, voice commentary.

### Official pipeline mapping (Challenge brief, received Oct 9)

The brief defines one pipeline in five stages. Our topology covers each stage,
which is worth stating explicitly on the project page and in the demo:

| Brief stage | Our piece |
|---|---|
| (a) Ingest events as they happen | Event Ingestor + Coordinator intake |
| (b) Interpret into stats, patterns, context | The three parallel readers |
| (c) Explain why a moment matters | Pattern Agent (control vs chaos, pressure changes, rhythm) |
| (d) Render insight on screen | UI feed + recap panel |
| (e) Personalize | Two audience modes (analyst vs casual fan), player-focused mode via the Player Involvement lens, EN/PT language |

The brief also lists features to consider: player identification with speed
and distance thresholds, pass quality (distance, accuracy, difficulty rating),
ball and shot speed, auto-eventing from video, narrative generation, and
multi-language storytelling. We take the narrative and multi-language items as
core (already in scope); pass quality and speed metrics are cheap deterministic
computations if the dataset carries the fields; auto-eventing from video is out
of scope unless the dataset turns out to be video.

One gap versus our earlier scope: the brief treats personalization as a
first-class stage, not a stretch goal. Proposal (pending Helena's ratification,
Oct 12): promote the minimal version into the MVP, two rendering modes over
the same MatchState, analyst (dense stats) and casual fan (story first), plus a
player-focused filter, because stage (e) is one of five and the fallback
scope must not quietly drop it.

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
| Oct 27 | Submit well before 23:59 PT |

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
- [ ] Synthetic data only; no real Premier League data in repo or video

## 12. Open questions for the Challenge brief

1. Dataset delivery (download/API/stream) and schema? **Resolved Oct 9: no
   provided dataset found on Innovation Studio; we generate our own synthetic
   event stream (Section 7).**
2. Required or prohibited models/services? Provided Azure credits? **Brief text
   silent on both**; no required services named, no credit offer stated.
3. Exact submission fields on the project page? **Not in the brief text**; the
   Oct 1 overview's submission list (Section 11 checklist) stands.
4. Single fixture or multiple matches in the dataset? **Ours to choose**: the
   generator will produce one canonical demo match, parameterized so more
   matches cost nothing extra.

Answered by the brief (Oct 9): the five-stage pipeline and feature list are
requirements, not suggestions; personalization (stage e) is a core stage;
narrative generation, explainability, and multi-language output are expected.
See Section 8 mapping.
