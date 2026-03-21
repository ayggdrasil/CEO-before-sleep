# CEO Before Sleep — Improvement Roadmap

> **v2.2.0 status**: All 5 items below are now implemented in `CEO-before-sleep.md`.
> This file is archived for context on why each decision was made.

Validated improvements drawn from comparing CEO Before Sleep against the gStack engineering sprint framework.

---

## 1. Competitor Shadow Agent (6th Agent)

Add a 6th agent after the Consultant on every dimension:

```
You are a Competitor Intelligence Analyst who thinks like the enemy.

For this dimension, identify the top 3 competitors who could exploit this weakness.
For each competitor:
- Name their most likely attack vector against this specific gap
- Estimate their timeline to execute (3 / 6 / 12 months)
- Rate the threat: Existential / Serious / Manageable

Then answer: "If I were the CEO of [Competitor X], what would I do in the next 90 days
to make this business irrelevant?"

Output format:
- Competitor attack map (table: competitor × vector × timeline × threat level)
- Most dangerous single move a competitor could make right now
- Defensive countermove the CEO should pre-empt today
```

**Why**: Dimensions 1–10 only look inward. The Competitor Shadow forces the CEO to see the business from the outside — the only perspective that actually kills companies.

**Impact**: 13 dims × 6 agents = 78 loops. Token cost increases ~20%.

---

## 2. Quantified Scoring Rubric

Replace letter grades (A–F) with weighted 0–100 scores per dimension. Current report template (`templates/report_template.md`) already includes the scoring table — this improvement automates the calculation.

**Composite score formula**: `Σ (dimension_score × weight)`

**Threshold rules:**
- Score < 40 → flag for Quick Scan re-run after founder addresses gaps
- Score 40–60 → Core Analysis recommended
- Score > 60 → Full Analysis complete, proceed to gStack sprint

---

## 3. Execution Retro Dimension (Dimension 14)

Add a 14th dimension — runs only if the company is post-seed (has shipped something):

```
Execution Retro: "What have you actually shipped in the last 90 days?"

Reviewer: List every concrete deliverable from the last quarter found in materials.
Problem Finder: Identify the gap between what was planned and what shipped.
Problem Solver: Propose a system to close the execution gap (OKRs, weekly demos, etc.)
Researcher: Benchmark shipped velocity against YC companies at the same stage.
Consultant: "If this team's shipping pace continues unchanged for 12 months,
             where does the company end up?"
```

**Why**: Vision documents are cheap. The gap between plan and shipped reality is the single most predictive signal of execution capability.

---

## 4. Interactive Pause Mode (`--interactive`)

When run with `--interactive`, the Consultant pauses after each dimension and presents one clarifying question:

```
[Dimension 3: Team Composition — complete]

Consultant's question:
"You have no CTO and no technical co-founder. Is this a deliberate choice or a constraint?
Your answer changes my recommendation for Dimensions 4, 8, and 13."

[Type your answer or press Enter to continue with current assumptions]
```

**Why**: Replaces assumption with ground truth. Especially valuable for Dimensions 11–13 where all prior analyses converge.

---

## 5. `/gstack:browse` Integration

When a live product exists, add a `/gstack:browse` step between Phase 1 (Material Collection) and Phase 2 (Analysis):

```
Phase 1.5: Live Product Validation
- Use /gstack:browse to screenshot the live product
- Compare actual UX against what pitch materials claim
- Flag discrepancies as RED issues in all relevant dimensions
```

**Why**: Ground truth beats pitch decks. Browsers don't lie.

---

## Implementation Status (v2.2.0)

| # | Improvement | Status |
|---|---|---|
| 1 | Quantified Scoring Rubric | ✅ Implemented |
| 2 | Execution Retro (Dim 14) | ✅ Implemented |
| 3 | `/gstack:browse` Integration | ✅ Implemented (Phase 1.5) |
| 4 | Interactive Pause Mode | ✅ Implemented (`--interactive`) |
| 5 | Competitor Shadow Agent | ✅ Implemented (Agent 6) |
