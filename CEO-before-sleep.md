---
name: CEO-before-sleep
version: 3.0.0
description: Deeply analyze all materials in a project folder from a CEO's perspective. Uses the Agent tool to spawn parallel dimension workers — 14 dimensions run concurrently, 6 roles run sequentially within each worker. Triggers on: "CEO review of /path", "due diligence on /path", "before sleep analysis of /path", "startup analysis", "investment analysis", "business diagnostics", "investor memo for /path", "competitor radar on /path". Reads every document (.md, .docx, .pdf, .xlsx, .pptx, .txt, .csv, etc.), uses claude-opus-4-6, and produces a structured markdown report.
changelog: "v3.0: Full parallel orchestration — Agent tool spawns 14 dimension workers (parallel waves) + Synthesis Agent. Each worker runs 6 roles sequentially. | v2.3: +Phase 6 Investor Memo, +Phase 7 Competitor Radar | v2.2: +Competitor Shadow, +Execution Retro, +Scoring, +Interactive, +Browse"
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

### Phase 2: Parallel Dimension Analysis (Agent Tool Orchestration)

**Model**: `claude-opus-4-6`
**Structure**: 14 dimension workers run in parallel waves. Within each worker, 6 roles run sequentially.
**Parallelism**: Dimensions are parallel. Roles within a dimension are sequential.
**Interactive mode**: If `--interactive`, the Consultant pauses after each dimension (see Agent 5 below).

```
Orchestrator
 ├─ WAVE 1 (parallel, Agent tool) ── Dims 1–10 ── all spawn simultaneously
 │   Each worker: Reviewer→Finder→Solver→Researcher→Consultant→Shadow
 │   Each writes: dim_01_*.md … dim_10_*.md
 │   [wait for all 10]
 │   Orchestrator builds wave1_summaries.md
 │
 ├─ WAVE 2 (parallel, Agent tool) ── Dims 11–14 ── all spawn simultaneously
 │   Receive wave1_summaries.md as additional context
 │   Each writes: dim_11_*.md … dim_14_*.md
 │   [wait for all 4]
 │
 └─ SYNTHESIS AGENT (Agent tool, single)
     reads all 14 dim_*.md → assembles report + action plan
```

**Shared context file:** Before spawning any workers, write all materials + Phase 0 answers to:
`{output_path}/context_bundle.md` — every worker reads this file first.

**Worker spawn pattern:** Dispatch all workers in a wave as a single message with multiple parallel Agent tool calls. Do not spawn them one by one.

**Wave 1 dispatch table:**

| Worker | Dim | Name | Output file |
|--------|-----|------|-------------|
| Agent call 1 | 01 | Item Profitability | `dim_01_item_profitability.md` |
| Agent call 2 | 02 | Talent Acquisition | `dim_02_talent_acquisition.md` |
| Agent call 3 | 03 | Team Composition | `dim_03_team_composition.md` |
| Agent call 4 | 04 | Technology | `dim_04_technology.md` |
| Agent call 5 | 05 | Legal | `dim_05_legal.md` |
| Agent call 6 | 06 | Corporate Structure | `dim_06_corporate_structure.md` |
| Agent call 7 | 07 | Finance | `dim_07_finance.md` |
| Agent call 8 | 08 | Execution | `dim_08_execution.md` |
| Agent call 9 | 09 | GTM | `dim_09_gtm.md` |
| Agent call 10 | 10 | Sales | `dim_10_sales.md` |

After Wave 1 completes: extract each Consultant's score + one-line diagnosis → write `wave1_summaries.md`.

**Wave 2 dispatch table:**

| Worker | Dim | Name | Special instructions |
|--------|-----|------|---------------------|
| Agent call 1 | 11 | Most Urgent Task | Synthesize Dims 1–10. One #1 task: "If you don't do X, Y happens in Z days." |
| Agent call 2 | 12 | Biggest Risk | Risk matrix + Base/Bull/Bear scenarios |
| Agent call 3 | 13 | Crazy Founder's Advice | Single serial-founder voice, 3 exits. Path to $100B. |
| Agent call 4 | 14 | Execution Retro | Skip if pre-product. Quantify slip rate vs. YC benchmark. |

After Wave 2 completes: spawn Synthesis Agent.

**Synthesis Agent prompt:**
```
Read all 14 dim_*.md files from {output_path}/.
Extract Consultant dimension scores (line: "Dimension score: [X / 100]").
Compute Composite Investability Score (weighted formula).
Assemble full report using templates/report_template.md.
Write: {output_path}/CEO_BeforeSleep_Report.md
Write: {output_path}/CEO_ActionPlan.md
```

For the full Dimension Worker Prompt Template, see `~/.claude/skills/CEO-before-sleep/SKILL.md`.

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

### Phase 6: Investor Memo

After Full or Core Analysis completes, auto-draft a structured investor memo using the analysis as source material.

**When to run:**
- After Full or Core Analysis (sufficient data needed)
- Can also run standalone: `/investor-memo <folder>` if an analysis report already exists
- If Investability Score < 40: still generate the memo, but prepend a warning — sending it now may close doors

**Three sub-agents:**

| Agent | Role |
|-------|------|
| **Memo Writer** | Drafts clean, investor-readable narrative from analysis data |
| **Devil's Advocate** | Flags the 3 questions every investor will ask that the memo doesn't answer well |
| **Positioning Expert** | Sharpens the narrative: what's the one thing this company should be known for? |

**Output:** `<output_path>/CEO_InvestorMemo.md`

**Memo structure:**

1. Header — Company, stage, ask, date
2. TL;DR — One paragraph that earns the next 5 minutes of an investor's time
3. Problem & Solution — What breaks without this, what changes with it
4. Market Opportunity — TAM/SAM/SOM with numbers from Researcher's output
5. Business Model — How it makes money, unit economics summary
6. Traction — Key metrics pulled from Reviewer's confirmed facts
7. Team — Signal-heavy, achievement-oriented (not just titles)
8. The Ask — Amount, use of funds in 3 buckets, runway extension
9. Risks & Mitigations — Top 3 risks from Dimension 12 + mitigation summary
10. Appendix — Investability Score, key KPIs to watch in 3 months

#### Memo Writer

```
You are an investor memo writer who has helped 30+ startups raise from Tier 1 VCs.

Using the CEO Before Sleep analysis as your source, draft an investor memo that:
1. Opens with TL;DR — one paragraph that earns the next 5 minutes of an investor's time
2. States the problem as a pain the investor can feel, not an abstraction
3. Presents the solution with one memorable differentiator
4. Shows market size with specific numbers (from Researcher output — if absent, flag with [NEEDS DATA])
5. Puts the strongest traction metric in bold in the first 100 words
6. Writes the team section like a story — not a LinkedIn bio dump
7. Closes with a clear ask: "$X to achieve Y by Z date"

Rules:
- Never use the words "disruptive", "innovative", "game-changing", or "paradigm"
- If a section lacks data, write [DATA GAP: X needed] rather than bluffing
- Keep the full memo under 1,500 words (body only, excluding appendix)
- Match the language of the source materials
```

#### Devil's Advocate

```
You are a partner at a top-tier VC who has seen 10,000 pitches.

Read the investor memo draft and identify:
1. The 3 questions every serious investor will ask that this memo cannot answer well
2. The #1 red flag a skeptical reader will fixate on
3. The claim that is weakest — where data is missing or logic doesn't hold
4. One sentence the founder should never say in a pitch meeting based on this memo

For each issue: state the investor's actual objection, rate severity (Killer / Serious / Minor),
and suggest the specific fix.
```

#### Positioning Expert

```
You are a brand strategist who has positioned Series A through IPO companies.

Read the memo and answer:
1. What is the one thing this company should be famous for — in one sentence?
2. Does the current memo communicate that thing? (Yes / Partially / No)
3. Who is the target investor persona — and does this memo speak their language?
4. Rewrite the TL;DR to be 30% shorter and 2x more compelling.
5. Suggest the ONE metric the company should lead every conversation with.
```

**Scoring gate:**

```
If Investability Score < 40:
  Prepend to memo:
  "⚠️  Investability Score [X]/100 is below the 40-point threshold.
   Sending this memo now may close doors before you're ready.
   Recommend addressing [top 2 critical gaps from Dimension 12] first.
   Re-run analysis after addressing gaps."
```

---

### Phase 7: Competitor Radar

A dedicated competitive intelligence loop. Can run standalone or as a continuation of the full analysis.

**Invocation:**
```
/competitor-radar <folder> [--competitors "Company A, Company B, Company C"]
```

Natural triggers: "competitor analysis of [folder]", "who are we up against", "competitive intelligence on [folder]", "competitive landscape"

If `--competitors` is not provided, extract competitor names from the materials automatically. Confirm with the founder before running.

**Three dedicated agents:**

| Agent | Role |
|-------|------|
| **Intel Collector** | Profiles each competitor: model, stage, funding, strengths, GTM |
| **Threat Mapper** | Maps each competitor's attack vector across all 14 dimensions |
| **Counter-Strategist** | Designs the defensive playbook, moat-builders, and asymmetric bets |

**Output:** `<output_path>/CEO_CompetitorRadar.md`

#### Intel Collector

```
You are a competitive intelligence analyst.

For each named competitor:
1. Summarize their business model, stage, funding, and known strengths
2. Extract any direct mentions of them from the analyzed materials
3. Identify their primary customer segment and GTM motion
4. Rate your confidence in this data: High / Medium / Low (note what is missing)

Output: one profile card per competitor with a confidence rating.
```

#### Threat Mapper

```
You are a competitive strategy analyst who thinks in attack vectors.

For each competitor, map their threat level against each of the 14 analysis dimensions:

| Dimension | Competitor A | Competitor B | Competitor C |
|-----------|-------------|-------------|-------------|
| [dim]     | Existential / Serious / Manageable / None | ... | ... |

Then for each Existential or Serious threat:
- Describe the specific attack vector (how would they execute it?)
- Estimate timeline: 90 days / 6 months / 12 months
- Identify the trigger condition: "They move when [X] happens"

Output: Competitive Heat Map + Attack Vector Detail per threat.
```

#### Counter-Strategist

```
You are a CEO who has competed against each of these players before and won.

For each Existential or Serious threat identified by the Threat Mapper:
1. Pre-emptive move: What should the CEO do in the next 30 days to neutralize this?
2. Moat-builder: What would make this dimension defensible within 12 months?
3. Asymmetric bet: What can this company do that the competitor structurally cannot?

Output: Defensive Playbook with 30-day actions, 90-day moat builders, and asymmetric advantages.

Conclude with:
"The #1 competitor to fear right now is [X] because [one sentence].
 The window to neutralize them is [Y] months."
```

**Radar modes:**

| Mode | Competitors | Dimensions | Use when |
|------|------------|-----------|----------|
| Full Radar | All identified | 14 | Pre-fundraise, annual strategy |
| Focused Radar | 1–2 specific | All 14 | Known threat emerging |
| Hot Spots Only | All identified | Existential threats only | Quick check-in |

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

| Item | Tokens | Notes |
|------|--------|-------|
| context_bundle.md (input per worker) | 10,000–50,000 | shared read by all workers |
| Single dimension worker (6 roles) | ~40,000–60,000 | |
| Wave 1: 10 workers (parallel) | ~400,000–600,000 | wall-clock = 1 worker's time |
| Wave 2: 4 workers (parallel) | ~160,000–240,000 | wall-clock = 1 worker's time |
| Synthesis Agent | ~30,000–50,000 | |
| Phase 6: Investor Memo | +18,000–30,000 | |
| Phase 7: Competitor Radar | +30,000–60,000 | |
| gStack sprint handoff | +50,000–100,000 | |
| **Full run total** | **~640,000–980,000** | **wall-clock ≈ 3 worker turns** |

---

## Notes

- Always run Phase 0 first — 2 minutes of forcing questions saves hours of misaligned analysis.
- Run Phase 1.5 if a live product URL exists — screenshot truth beats pitch deck claims.
- Remove PII, passwords, and API keys from materials before running.
- This report is reference-only. Consult qualified professionals for legal, financial, and tax matters.
- See `templates/report_template.md` for the output report structure.
