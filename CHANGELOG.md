# CEO Before Sleep — Changelog

---

## v3.0.0 — 2026-03-23

### Architecture: Full Parallel Orchestration via Agent Tool

**The fundamental change:** Phase 2 is no longer a single-agent sequential loop. The orchestrator now uses the `Agent` tool to spawn dimension workers in parallel. Each worker is an independent subagent that runs the 6 expert roles sequentially in its own context.

**Pattern:** Inspired by Microsoft Magentic-One (orchestrator + specialist agents), OpenAI Swarm (agent handoffs), and Claude Code's own multi-agent orchestration patterns. Key insight: dimensions are independent — there's no reason to wait for Dim 3 to finish before starting Dim 4. Only Dims 11–12 have a data dependency (they need Dims 1–10 to synthesize from).

**Execution model:**

```
Wave 1 (parallel): Dims 1–10 — 10 Agent tool calls in one message
  ↓ [wait for all 10, build wave1_summaries.md]
Wave 2 (parallel): Dims 11–14 — 4 Agent tool calls in one message
  ↓ [wait for all 4]
Synthesis Agent: 1 Agent tool call — reads 14 files, assembles report
```

**Shared memory via file system:** Before spawning, the orchestrator writes `context_bundle.md` to disk. All workers read it. This is the standard pattern for shared context in multi-agent systems — avoids re-passing the full context in every agent prompt string.

**Performance gain:** Wall-clock time for the full analysis drops from ~14x single worker time to ~3x single worker time (Wave 1 + Wave 2 + Synthesis).

**Dimension Worker Prompt Template:** Added to `~/.claude/skills/CEO-before-sleep/SKILL.md`. Reusable for all 14 workers. Fill in dimension name, description, output path. Each worker internally runs Roles 1–6 in sequence, with each role reading all prior roles' output.

**Synthesis Agent:** New dedicated agent that reads all 14 `dim_*.md` files, extracts dimension scores, computes the Composite Investability Score, and assembles the final report + action plan.

**Output files produced:**
- `context_bundle.md` — shared context for all workers
- `wave1_summaries.md` — extracted scores + diagnoses from Dims 1–10
- `dim_01_*.md` through `dim_14_*.md` — per-dimension analysis
- `CEO_BeforeSleep_Report.md` — assembled full report
- `CEO_ActionPlan.md` — sprint-ready backlog

### Updated

- `~/.claude/skills/CEO-before-sleep/SKILL.md` — full rewrite of Phase 2 with parallel orchestration, Dimension Worker Prompt Template, Wave 1/2 dispatch tables, Synthesis Agent prompt, output file manifest. Bumped to v3.0.0.
- `CEO-before-sleep.md` (project copy) — updated Phase 2 section with parallel architecture diagram and dispatch tables. Bumped to v3.0.0.
- Token estimate table updated: wall-clock now ~3 worker turns for full run.

---

## v2.3.0 — 2026-03-23

### Added: Phase 6 — Investor Memo

A three-agent pipeline that auto-drafts a VC-ready investor memo from the full analysis output.

**Why:** The analysis produces everything needed for a fundraise memo — but in analyst format, not investor format. Phase 6 translates CEO depth into investor language without starting from scratch.

**How it works:**

| Agent | Role |
|-------|------|
| Memo Writer | Drafts a clean, narrative memo under 1,500 words. Flags `[DATA GAP: X]` where data is missing rather than bluffing. Banned words: "disruptive", "innovative", "game-changing", "paradigm". |
| Devil's Advocate | Acts as a skeptical VC partner. Surfaces the 3 investor objections the memo can't answer, rates each (Killer / Serious / Minor), and recommends specific fixes. |
| Positioning Expert | Identifies the one thing this company should be famous for. Rewrites the TL;DR 30% shorter and 2x more compelling. Names the lead metric for every conversation. |

**Scoring gate:** If Investability Score < 40, the memo is still generated but prefaced with a warning — sending a memo before addressing critical gaps can close investor doors.

**Invocation:** Runs automatically after Full or Core Analysis. Can also be triggered standalone:
```
investor memo for /path/to/folder
```

**Output file:** `CEO_InvestorMemo.md`

---

### Added: Phase 7 — Competitor Radar

A dedicated competitive intelligence loop that maps named competitors across all 14 analysis dimensions.

**Why:** The Competitor Shadow agent (added in v2.2) analyses each dimension through an adversarial lens inside the main loop. Competitor Radar is different — it's a cross-cutting view: take N named competitors and map them against all dimensions simultaneously. Useful pre-fundraise, pre-launch, and for annual strategy.

**How it works:**

| Agent | Role |
|-------|------|
| Intel Collector | Builds a profile card per competitor: business model, stage, funding, GTM motion, strengths, confidence rating. Extracts any direct mentions from analyzed materials. |
| Threat Mapper | Produces a heat map (competitors × 14 dimensions) with Existential / Serious / Manageable / None ratings. For each red/yellow cell: attack vector, timeline, and trigger condition. |
| Counter-Strategist | For each existential or serious threat: 30-day pre-emptive move, 90-day moat-builder, and the asymmetric bet (what this company can do that the competitor structurally cannot). Concludes with the #1 competitor to fear and the window to neutralize them. |

**Radar modes:**

| Mode | Competitors | Dimensions | Use when |
|------|------------|-----------|----------|
| Full Radar | All identified | 14 | Pre-fundraise, annual strategy |
| Focused Radar | 1–2 specific | All 14 | Known threat emerging |
| Hot Spots Only | All identified | Existential only | Quick check-in |

**Invocation:**
```
/competitor-radar <folder> [--competitors "Company A, Company B, Company C"]
competitor radar on /path
who are we up against
```
If `--competitors` is not provided, competitor names are extracted from the materials and confirmed with the founder before running.

**Output file:** `CEO_CompetitorRadar.md`

---

### Updated

- `CEO-before-sleep.md` — bumped to v2.3.0, updated description and changelog line, added Phase 6 and Phase 7 sections, updated token estimate table
- `templates/report_template.md` — added Phase 6 (Investor Memo) and Phase 7 (Competitor Radar) sections with full output scaffolding, bumped version ref to v2.3.0, updated methodology note in Appendix D
- `README.md` — added Phase 6 and Phase 7 to What's New, updated usage examples with investor memo and competitor radar triggers, updated token budget table

---

## v2.2.0

- Added Competitor Shadow Agent (6th sub-agent) — adversarial lens on each dimension
- Added Dimension 14: Execution Retro — planned vs. shipped velocity vs. YC benchmark
- Added Quantified Scoring — weighted 0–100 Investability Score with threshold rules
- Added Interactive Pause Mode (`--interactive`) — Consultant pauses per dimension for one clarifying question
- Added Phase 1.5 Browse Integration — `/gstack:browse` screenshots the live product before analysis
- Added `/ceo-sprint` composite command — full analysis → gStack sprint in one invocation

## v2.1.0

- Separated skill, template, and roadmap into distinct files
- Added gStack sprint handoff (Phase 5)
- Added Phase 0 forcing questions

## v2.0.0

- Initial multi-agent architecture: 5 agents × 13 dimensions
- Added Phase 1 material collection with multi-format file support
- Added Phase 1.5 live product validation placeholder

## v1.0.0

- Single-agent CEO analysis
- Basic dimension coverage
- Markdown report output
