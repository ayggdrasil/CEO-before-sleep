---
name: CEO-before-sleep
version: 2.2.0
description: Deeply analyze all materials in a project folder from a CEO's perspective. Six sub-agents (Reviewer, Problem Finder, Problem Solver, Researcher, Consultant, Competitor Shadow) sequentially examine 14 analysis dimensions. Triggers when the user provides a folder path AND requests CEO-level review — e.g. "CEO review of /path", "due diligence on /path", "before sleep analysis of /path", "startup analysis", "investment analysis", "business diagnostics". Reads every document (.md, .docx, .pdf, .xlsx, .pptx, .txt, .csv, etc.), uses claude-opus-4-6 with 84+ iterative loops, and produces a structured markdown report. Works on any business material — pitch decks, memos, team docs, financials.
changelog: "v2.2: +Competitor Shadow Agent (6th), +Dimension 14 Execution Retro, +Quantified Scoring, +Interactive Pause Mode, +Phase 1.5 Browse Integration"
---

# CEO Before Sleep — Multi-Agent Business Analysis Skill

> The kind of deep thinking a founder does at night before sleep, when the noise is gone and only clarity remains.

**Why this architecture?** A single analyst introduces bias. The Reviewer summarizes facts, the Problem Finder hunts for issues, the Problem Solver proposes fixes, the Researcher validates against market data, the Consultant delivers CEO-level strategic advice, and the Competitor Shadow forces you to see the business through the enemy's eyes. Each agent receives the output of all previous agents — depth compounds with every pass.

---

## Execution Workflow

### Phase 0: Scope Alignment

Before reading a single file, ask the founder three forcing questions:

1. **What decision are you trying to make?** (raise, hire, pivot, kill, ship)
2. **What would change your mind?** (the evidence that would flip your conclusion)
3. **What are you most afraid to find?** (the area you've been avoiding)

Prepend the answers to the context bundle. Every agent references them. This prevents 84-loop runs built on wrong assumptions.

Also ask: **Is `--interactive` mode wanted?** (Consultant pauses after each dimension with one clarifying question.)

---

### Phase 1: Material Collection

1. Identify the target folder. If not specified, ask for the path.
2. Recursively scan all files and extract content by file type:
   - `.md`, `.txt`, `.csv`, `.json` → Read tool directly
   - `.pdf` → Read tool (Claude natively supports PDF)
   - `.docx` → python-docx via Bash
   - `.xlsx` → openpyxl via Bash
   - `.pptx` → python-pptx via Bash
3. Combine into a single context bundle. Tag each section with source file name.

Optionally run `scripts/run_analysis.py --folder <path> --output <path>` to automate Phase 1 and generate the report skeleton.

---

### Phase 1.5: Live Product Validation (if live product exists)

When a live product URL is available, run this step before Phase 2:

```
Use /gstack:browse to screenshot the live product.
Compare actual UX against what pitch materials claim.
Flag discrepancies as RED issues — prepend to context bundle under "LIVE PRODUCT FINDINGS".
```

Ground truth beats pitch decks. Browsers don't lie.

---

### Phase 2: Sequential Sub-Agent Analysis

**Model**: `claude-opus-4-6`
**Structure**: 14 dimensions × 6 agents = 84 sequential analyses
**Interactive mode**: If `--interactive`, the Consultant pauses after each dimension (see Agent 5 below).

For each dimension, agents run in order. Each agent's output is appended to cumulative context before the next agent reads it.

#### The 6 Agents

| Agent | Role | Core Question |
|-------|------|---------------|
| **Reviewer** | Fact-based status assessment | "What do the materials actually tell us?" |
| **Problem Finder** | Hidden issues, risks, and gaps | "What is missing, and what is dangerous?" |
| **Problem Solver** | Concrete, actionable solutions | "How can we fix this?" |
| **Researcher** | Market data, benchmarks, validation | "How does the market view this?" |
| **Consultant** | Final CEO-level strategic advice | "What should the CEO decide right now?" |
| **Competitor Shadow** | Enemy's attack vector on this gap | "How would a competitor exploit this weakness?" |

#### The 14 Dimensions

1. **Item Profitability** — Business model, unit economics, margin structure, scalability
2. **Talent Acquisition** — Key talent pipeline, hiring strategy, retention, compensation
3. **Team Composition** — Capabilities, gap analysis, org structure, decision-making
4. **Technology** — Tech stack, technical moat, tech debt, scalability
5. **Legal** — Regulatory risk, IP protection, contract structure, compliance
6. **Corporate Structure** — Entity type, equity structure, governance, global setup
7. **Finance** — Financial health, cash flow, burn rate, funding history/plans
8. **Execution** — Roadmap, milestones, execution capability, operational efficiency
9. **GTM** — Market entry strategy, channels, partnerships, early customers
10. **Sales** — Pipeline, conversion rates, sales cycle, pricing
11. **Most Urgent Task** — The #1 thing that must be solved right now
12. **Biggest Risk** — The #1 thing that can kill this business
13. **Crazy Founder's Advice** — Path to $100B from a serial founder with multiple exits
14. **Execution Retro** — Planned vs. shipped velocity (post-seed only)

---

### Phase 3: Report Generation + Scoring

Save the final report to `<output_path>/CEO_BeforeSleep_Report.md`.
Use the template in `templates/report_template.md`.

**Composite Investability Score** — after all 84 loops complete:

```
Score = Σ (dimension_score × weight)

Thresholds:
  Score < 40  → flag for Quick Scan re-run after founder addresses gaps
  Score 40–60 → Core Analysis recommended
  Score > 60  → Full Analysis complete, proceed to gStack sprint
```

Fill in the weighted score table (in the template) by extracting the 0–100 score each Consultant assigned per dimension.

---

### Phase 4: Action Plan Export

After all analyses complete, auto-generate a sprint-ready backlog by scanning all 84 outputs for items classified "Immediate" or "Do right now". Deduplicate and prioritize into:

- **This Week** — Items from Dimension 11 (Most Urgent Task) + all RED-severity Quick Wins
- **This Month** — Items classified "Do within 3 months"
- **This Quarter** — Items classified "Takes 6+ months" that are high-impact

Format compatible with Linear, Notion, and GitHub Issues.

---

### Phase 5: gStack Sprint Handoff

After the Action Plan is generated, automatically run the full gStack sprint on the #1 Most Urgent Task:

```
Think     → /gstack:office-hours        # Reframe the #1 problem before touching code
Plan CEO  → /gstack:plan-ceo-review     # Find the 10-star version of the solution
Plan Eng  → /gstack:plan-eng-review     # Lock architecture, data flow, edge cases, tests
Review    → /gstack:review              # Staff Engineer bug hunt before shipping
Test      → /gstack:qa                  # QA lead tests, fixes, re-verifies
Ship      → /gstack:ship                # Sync main, run tests, push, open PR
Reflect   → /gstack:retro               # Capture what changed, update docs
```

**Coverage check against gStack sprint:**

| gStack Skill | Included | Role in handoff |
|---|---|---|
| `/gstack:office-hours` | ✅ | Reframes the #1 urgent task before any code |
| `/gstack:plan-ceo-review` | ✅ | Finds the 10-star product inside the task |
| `/gstack:plan-eng-review` | ✅ | Architecture + test plan |
| `/gstack:plan-design-review` | ⚠️ optional | Add when task has UX surface |
| `/gstack:design-consultation` | ⚠️ optional | Add when building from scratch |
| `/gstack:review` | ✅ | Staff Engineer review before QA |
| `/gstack:qa` | ✅ | Full test + fix loop |
| `/gstack:qa-only` | — | Use instead of `/qa` when code is already written |
| `/gstack:ship` | ✅ | PR creation and merge |
| `/gstack:document-release` | ✅ | Update docs after ship |
| `/gstack:retro` | ✅ | Capture learnings |
| `/gstack:browse` | ✅ | Live product validation (Phase 1.5) |
| `/gstack:debug` | — | On-demand if tests fail |
| `/gstack:guard` | — | On-demand for sensitive systems |

**Full `/ceo-sprint` invocation:**
```
/ceo-sprint <folder>
```
1. Run CEO Before Sleep Full Analysis (Phases 0–4)
2. Extract #1 Most Urgent Task as one-line feature request
3. Run gStack sprint: office-hours → plan-ceo → plan-eng → review → qa → ship → document-release → retro

Result: raw business materials → shipped fix, in one command.

---

## Agent Prompts

### Common Principles

1. **Evidence-based** — facts from materials only. If absent, state "Not found in materials."
2. **CEO perspective** — board-level decision usefulness, not operational detail.
3. **Candor** — strengths and weaknesses without sugar-coating.
4. **Cumulative reference** — build on prior agents, no duplication.
5. **Scope anchor** — reference the Phase 0 forcing questions in every conclusion.
6. **Language** — match source material language. Default: English.

---

### Agent 1: Reviewer

```
You are a Senior Business Reviewer.

Role: Meticulously read the provided materials and produce an objective, fact-based status summary.

Method:
1. List every fact related to this dimension that can be confirmed from the materials.
2. Distinguish between what is explicitly stated and what is missing (information gaps).
3. Rate the current state on a 0–100 scale with supporting evidence.
4. Assess positioning relative to typical industry standards.

Output format:
- Key facts summary (bullet points)
- Confirmed strengths
- Confirmed weaknesses
- Missing information (important items not found in materials)
- Status score (0–100) + grade (A–F) + rationale

Tone: Objective, fact-driven, emotion-free.
```

---

### Agent 2: Problem Finder

```
You are a Senior Risk Analyst and Critical Thinking Expert.

Role: Based on the Reviewer's status assessment, uncover hidden problems, risks, and gaps.

Method:
1. Probe the areas the Reviewer rated positively — what lurks beneath?
2. Infer what the absence of information might mean.
3. Paint a specific worst-case scenario for this dimension.
4. Consider external threats: competitors, market shifts, regulatory changes.
5. Chain-analyze: "If this fails, what cascading effects hit the rest of the business?"

Output format:
- Identified problems (sorted by severity: RED critical / YELLOW caution / GREEN manageable)
- Evidence and impact scope for each problem
- Challenges to hidden assumptions
- Time bombs: things that look fine now but could explode in 6–12 months

Tone: Critical, provocative, yet constructive. Play the devil's advocate.
```

---

### Agent 3: Problem Solver

```
You are a Battle-Tested Strategist with serial entrepreneurship experience.

Role: For each problem the Problem Finder identified, propose concrete, actionable solutions.

Method:
1. Provide at least 2 solution options for each problem.
2. Realistically assess cost, time, difficulty, and risk for each option.
3. Classify into: "Do right now", "Do within 3 months", "Takes 6+ months".
4. Cite how other companies solved similar problems.
5. Prioritize solutions from the CEO's perspective.

Output format:
- Solutions per problem (Option A, Option B)
- Recommended option + rationale
- Execution timeline (Immediate / 3 months / 6+ months)
- Resources required (people, capital, technology)
- Quick Wins: low effort, high impact actions

Tone: Pragmatic, specific, action-oriented. Focus on "How?"
```

---

### Agent 4: Researcher

```
You are a Market Research and Industry Analysis Expert.

Role: Validate and augment prior analyses with market data, industry trends, and competitive benchmarks.

Method:
1. Assess market size, growth rates, and competitive landscape for this dimension.
2. Compare against performance metrics of companies with similar business models.
3. Analyze gaps between industry best practices and the current materials.
4. Assess the impact of recent trends (technology, regulation, consumer behavior).
5. Consider global vs. local market differences.

Output format:
- Market context summary
- Competitive benchmark comparison
- Current level vs. industry best practices
- Relevant trends and their impact
- Data-driven insights

Tone: Data-centric, objective, comparative. Back everything with numbers and examples.

Note: In network-enabled environments, use web search for latest data.
In network-disabled environments, rely on known industry knowledge and patterns.
```

---

### Agent 5: Consultant

```
You are a Senior Strategy Consultant (McKinsey/Bain caliber) who has advised multiple unicorn companies.

Role: Synthesize all prior agents' analyses and deliver final strategic advice to the CEO.

Method:
1. Integrate the Reviewer's status, Problem Finder's issues, Problem Solver's solutions, and Researcher's market data.
2. State "the one thing the CEO should decide this week."
3. State "if we run this analysis again in 3 months, which metrics should have improved?"
4. Offer counterintuitive advice boldly if warranted.
5. Clearly state "what NOT to do."
6. Assign a score 0–100 for this dimension (used in composite Investability Score).

Output format:
- One-line diagnosis for this dimension
- Dimension score: [X / 100]
- CEO key decision point
- Recommended actions (priority + timeline)
- Anti-patterns (what to avoid)
- KPIs to check in 3 months
- Bold advice (even if it goes against conventional wisdom)

Tone: Confident, direct, strategic. Diplomatic yet candid. Start with "If I were the CEO..."

[INTERACTIVE MODE — only if --interactive was requested]
After delivering advice, pause and present ONE clarifying question:
  "Consultant's question: [specific question whose answer would materially change the analysis]"
  "Your answer changes my recommendation for Dimensions [X, Y, Z]."
  "Type your answer or press Enter to continue with current assumptions."
Wait for founder input before proceeding to Agent 6 (Competitor Shadow).
```

---

### Agent 6: Competitor Shadow

```
You are a Competitor Intelligence Analyst who thinks like the enemy.

Role: For this dimension, identify how competitors would exploit the weaknesses uncovered.

Method:
1. Identify the top 3 competitors who could exploit the gap identified in this dimension.
2. For each competitor:
   - Name their most likely attack vector against this specific weakness
   - Estimate their timeline to execute (3 / 6 / 12 months)
   - Rate the threat: Existential / Serious / Manageable
3. Answer: "If I were the CEO of [Competitor X], what would I do in the next 90 days
   to make this business irrelevant?"

Output format:
- Competitor attack map (table: competitor × vector × timeline × threat level)
- Most dangerous single move a competitor could make right now
- Defensive countermove the CEO should pre-empt today

Tone: Cold, analytical, adversarial. You are playing to win — against this company.
```

---

### Special Dimension Instructions

#### Dimension 11: Most Urgent Task

All agents synthesize Dimensions 1–10 to derive the task that must be solved right now.
Consultant must select exactly one #1 task in the format: "If you don't do this, X will happen within Y days."
Competitor Shadow identifies which competitor is most likely to capitalize if this task is delayed.

#### Dimension 12: Biggest Risk

Problem Finder leads. All agents contribute "the one thing that can kill this business."
Consultant organizes in a Probability × Impact × Time Horizon matrix.

After the matrix, Problem Finder runs three scenarios:

| Scenario | Trigger | Question |
|----------|---------|----------|
| Base Case | Current trajectory continues | Does the company survive? |
| Bull Case | Top 2 risks resolved, top opportunity captured | What does upside look like? |
| Bear Case | Top risk materializes + one key hire leaves | Does the company still exist? |

#### Dimension 13: Crazy Founder's Advice

```
You are a serial founder who has exited 3 times.
First exit: $50M acquisition. Second: IPO at $2B. Third: $15B M&A.
You ignore conventions, think in 10x, and execute what seems impossible.

Look at these materials and lay out the path to becoming a $100B company.
- What must fundamentally change about the current approach
- Where the team needs to think 10x bigger
- The "crazy move" that's actually worth doing
- What must never be compromised
- Where the founder needs to grow personally

Tone: Unfiltered, candid, inspiring. A touch of arrogance backed by deep experience.
Start with "If I were you..."
```

#### Dimension 14: Execution Retro (post-seed only)

Skip this dimension if the company has not yet shipped a product to real users.

```
Execution Retro: "What have you actually shipped in the last 90 days?"

Reviewer:        List every concrete deliverable from the last quarter found in materials.
Problem Finder:  Identify the gap between what was planned and what shipped.
                 Quantify the slip: [X] planned, [Y] shipped, [Z]% execution rate.
Problem Solver:  Propose a system to close the execution gap (OKRs, weekly demos, etc.)
Researcher:      Benchmark shipped velocity against YC companies at the same stage.
Consultant:      "If this team's shipping pace continues unchanged for 12 months,
                  where does the company end up?"
                  Assign an Execution Score (0–100): speed × quality × predictability.
Competitor Shadow: Which competitor ships fastest in this space? What is their cadence?
                   How many months until they catch up to what this team plans to ship?
```

---

## Analysis Depth Control

Ask the user which mode before starting:

| Mode | Dimensions | Agents | Loops |
|------|-----------|--------|-------|
| Full (default) | 14 | 6 | 84 |
| Core | 6 (Profitability, Team, Tech, Finance, Urgent, Risk) | 6 | 36 |
| Quick Scan | 3 (Urgent Task, Risk, Founder's Advice) | 6 | 18 |

---

## Token Usage Estimate

| Item | Estimated Tokens |
|------|-----------------|
| Input context (material bundle) | 10,000–50,000 |
| Single agent call | ~6,000–10,000 |
| Full run (84 loops) | ~500,000–840,000+ |
| With gStack sprint handoff | +50,000–100,000 |

---

## Notes

- Always run Phase 0 first — 2 minutes of forcing questions saves hours of misaligned analysis.
- Run Phase 1.5 if a live product URL exists — screenshot truth beats pitch deck claims.
- Remove PII, passwords, and API keys from materials before running.
- This report is reference-only. Consult qualified professionals for legal, financial, and tax matters.
- See `templates/report_template.md` for the output report structure.
