---
name: CEO-before-sleep
description: Deeply analyze all materials in a project folder from a CEO's perspective and deliver expert feedback. Five sub-agents (Reviewer, Problem Finder, Problem Solver, Researcher, Consultant) sequentially examine 13 analysis dimensions. Triggers when the user requests "CEO review", "business analysis", "investment analysis", "business feedback", "expert review", "before sleep analysis", "portfolio review", "startup analysis", "business diagnostics", "due diligence", or similar. Reads every document in the folder (.md, .docx, .pdf, .xlsx, .pptx, .txt, .csv, etc.), uses the strongest model (claude-opus-4-6) with 10+ iterative loops, and produces a structured markdown report. Works on any type of business material — pitch decks, investment memos, team docs, financials, and more.
---

# CEO Before Sleep — Multi-Agent Business Analysis Skill

## Overview

This skill reads every file in a project folder through the eyes of a CEO, runs 5 expert sub-agents across 13 key dimensions sequentially, and produces a structured markdown report — the kind of deep thinking a founder does at night before sleep, when the noise is gone and only clarity remains.

**Why this architecture?** A single analyst introduces bias. The Reviewer summarizes facts, the Problem Finder hunts for issues, the Problem Solver proposes fixes, the Researcher validates against market data, and the Consultant delivers final CEO-level advice. Each agent receives the output of all previous agents, so depth compounds with every pass.

---

## Execution Workflow

### Phase 0: Scope Alignment (New)

Before reading a single file, ask the founder one forcing question per analysis axis:

1. **What decision are you trying to make?** (raise, hire, pivot, kill, ship)
2. **What would change your mind?** (the evidence that would flip your conclusion)
3. **What are you most afraid to find?** (the area you've been avoiding)

These answers are prepended to the context bundle. They act as a north star — every agent references them. This prevents 65-loop runs built on wrong assumptions.

### Phase 1: Material Collection

1. Identify the target folder. If the user does not specify one, ask for the path.
2. Recursively scan all files in the folder and extract content by file type:
   - `.md`, `.txt`, `.csv`, `.json` → read directly with the Read tool
   - `.pdf` → extract text using the Read tool (Claude natively supports PDF)
   - `.docx` → extract with python-docx via Bash
   - `.xlsx` → extract with openpyxl via Bash
   - `.pptx` → extract with python-pptx via Bash
3. Combine all extracted text into a single context bundle. Tag each section with the source file name.

Optionally run `scripts/run_analysis.py --folder <path>` to automate Phase 1 and generate the report skeleton.

### Phase 2: Sequential Sub-Agent Analysis (10+ Loops)

**Model**: `claude-opus-4-6` (strongest available)
**Loop structure**: 13 analysis dimensions × 5 agents = 65 sequential analyses

For each dimension, the 5 agents analyze in order. Each agent's output is appended to the cumulative context so the next agent can build on it.

#### The 5 Sub-Agents

| Agent | Role | Core Question |
|-------|------|---------------|
| **Reviewer** | Fact-based status assessment | "What do the materials actually tell us?" |
| **Problem Finder** | Hidden issues, risks, and gaps | "What is missing, and what is dangerous?" |
| **Problem Solver** | Concrete, actionable solutions | "How can we fix this?" |
| **Researcher** | Market data, benchmarks, validation | "How does the market view this?" |
| **Consultant** | Final CEO-level strategic advice | "What should the CEO decide right now?" |

#### The 13 Analysis Dimensions (in order)

1. **Item Profitability** — Business model, unit economics, margin structure, scalability
2. **Talent Acquisition** — Key talent pipeline, hiring strategy, retention, compensation structure
3. **Team Composition** — Current team capabilities, gap analysis, org structure, decision-making framework
4. **Technology** — Tech stack, technical moat/differentiation, tech debt, scalability
5. **Legal** — Regulatory risk, IP protection, contract structure, compliance
6. **Corporate Structure** — Entity type, equity structure, governance, global structure
7. **Finance** — Financial health, cash flow, burn rate, funding history/plans
8. **Execution** — Roadmap, milestones, execution capability, operational efficiency
9. **GTM (Go-To-Market)** — Market entry strategy, channels, partnerships, early customers
10. **Sales** — Sales pipeline, conversion rates, sales cycle, pricing
11. **Most Urgent Task** — The #1 thing that must be solved right now
12. **Biggest Risk** — The #1 thing that can kill this business
13. **Crazy Founder's Advice** — From the perspective of a serial founder with multiple exits, the path to becoming a $100B company

### Phase 3: Synthesis and Report Generation

Save the final markdown report to `<output_path>/CEO_BeforeSleep_Report.md`.

---

## Agent Prompt Guide

### Common Principles

All agents follow these principles:

1. **Evidence-based**: Analyze based on facts found in the provided materials, not speculation. If no evidence exists, state "Not found in materials."
2. **CEO perspective**: Analyze at a level useful for CEO/board-level decision-making, not operational detail.
3. **Candor**: State both strengths and weaknesses without sugar-coating. Clarity over diplomacy.
4. **Cumulative reference**: Always read prior agents' analyses, avoid duplication, and add new perspectives.
5. **Scope anchor**: Reference the founder's three forcing questions from Phase 0 in every conclusion.
6. **Language**: Write all analysis in the same language as the source materials. Default to English.

---

### Agent 1: Reviewer

```
You are a Senior Business Reviewer.

Role: Meticulously read the provided materials and produce an objective, fact-based status summary.

Method:
1. List every fact related to this dimension that can be confirmed from the materials.
2. Distinguish between what is explicitly stated and what is missing (information gaps).
3. Rate the current state A–F with supporting evidence.
4. Assess positioning relative to typical industry standards.

Output format:
- Key facts summary (bullet points)
- Confirmed strengths
- Confirmed weaknesses
- Missing information (important items not found in materials)
- Status grade (A–F) + rationale

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
1. Realistically assess the market size, growth rates, and competitive landscape for this dimension.
2. Compare against performance metrics of companies with similar business models.
3. Analyze gaps between industry best practices and the current materials.
4. Assess the impact of recent trends (technology, regulation, consumer behavior) on this dimension.
5. Consider global vs. local market differences.

Output format:
- Market context summary
- Competitive benchmark comparison
- Current level vs. industry best practices
- Relevant trends and their impact
- Data-driven insights

Tone: Data-centric, objective, comparative. Back everything with numbers and examples.

Note: In network-enabled environments, use web search to gather the latest data.
In network-disabled environments, rely on generally known industry knowledge and patterns.
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

Output format:
- One-line diagnosis for this dimension
- CEO key decision point
- Recommended actions (priority + timeline)
- Anti-patterns (what to avoid)
- KPIs to check in 3 months
- Bold advice (even if it goes against conventional wisdom)

Tone: Confident, direct, strategic. Diplomatic yet candid. Start with "If I were the CEO..."
```

---

### Special Dimension Instructions

#### Dimension 11: Most Urgent Task

All agents synthesize Dimensions 1–10 to derive the task that must be solved right now.
The Consultant agent must select exactly one #1 task and explain it in the format: "If you don't do this, X will happen within Y days."

#### Dimension 12: Biggest Risk

The Problem Finder leads, but all agents contribute "the one thing that can kill this business."
The Consultant agent organizes risks in a Probability (High/Medium/Low) × Impact (High/Medium/Low) matrix and assigns a **time horizon** to each: when does this risk become lethal if unaddressed?

#### Dimension 13: Crazy Founder's Advice

For this dimension, the Consultant agent uses a special persona:

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

---

## Proposed Improvements (Roadmap)

These are validated improvements drawn from comparing CEO Before Sleep against the gStack engineering sprint framework. Each item is ready to implement.

### 1. Competitor Shadow Agent (6th Agent)

Add a 6th agent that runs after the Consultant on every dimension:

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

**Why it matters**: Dimensions 1–10 only look inward. The Competitor Shadow forces the CEO to see the business from the outside — the only perspective that actually kills companies.

---

### 2. Quantified Scoring Rubric

Replace letter grades (A–F) with a 0–100 weighted score per dimension. Add a weighted composite to produce a single investability score.

| Dimension | Weight |
|-----------|--------|
| Finance | 20% |
| Item Profitability | 15% |
| Biggest Risk | 15% |
| Team Composition | 12% |
| Execution | 10% |
| Technology | 8% |
| GTM | 8% |
| Sales | 5% |
| Legal | 3% |
| Corporate Structure | 2% |
| Talent Acquisition | 2% |

**Composite score formula**: `Σ (dimension_score × weight)`

Output at the top of the report:
```
Overall Investability Score: 68 / 100  →  B−
Confidence: Medium (7 of 13 dimensions had sufficient data)
```

Scores below 40 → flag for Quick Scan re-run after founder addresses gaps.

---

### 3. 12-Month Scenario Modeling (Dimension 12 Extension)

After the Risk matrix, the Problem Finder runs three scenarios:

| Scenario | Trigger | 12-Month Outcome |
|----------|---------|-----------------|
| **Base Case** | Current trajectory continues | Revenue, headcount, runway |
| **Bull Case** | Top 2 risks resolved, top opportunity captured | Same metrics |
| **Bear Case** | Top risk materializes, one key hire leaves | Same metrics |

For each scenario, state: **what does the P&L look like, and does the company still exist?**

This turns abstract risk language into a concrete survival question.

---

### 4. Execution Retro Dimension (Dimension 14)

Add a 14th dimension that runs only if the company is post-seed (has shipped something):

```
Execution Retro: "What have you actually shipped in the last 90 days?"

Reviewer: List every concrete deliverable from the last quarter found in materials.
Problem Finder: Identify the gap between what was planned and what shipped.
Problem Solver: Propose a system to close the execution gap (OKRs, weekly demos, etc.)
Researcher: Benchmark shipped velocity against YC companies at the same stage.
Consultant: "If this team's shipping pace continues unchanged for 12 months,
             where does the company end up?"
```

**Why it matters**: Vision documents are cheap. The gap between plan and shipped reality is the single most predictive signal of execution capability.

---

### 5. Action Plan Export (Phase 4)

After all 65 analyses complete, add a Phase 4 that auto-generates a sprint-ready action backlog:

```markdown
## This Week (Next 7 Days)
- [ ] [Owner TBD] [Action from Most Urgent Task #1]
- [ ] [Owner TBD] [Quick Win from Dimension X]

## This Month (Next 30 Days)
- [ ] [Action]
- [ ] [Action]

## This Quarter (Next 90 Days)
- [ ] [Action]
```

Format compatible with Linear, Notion, and GitHub Issues. The Consultant generates this by scanning all 65 outputs for items classified "Immediate" or "Do right now" and deduplicating them into a single prioritized list.

---

### 6. Interactive Pause Mode (`--interactive` flag)

When run with `--interactive`, the Consultant pauses after each dimension and presents:

```
[Dimension 3: Team Composition — complete]

Consultant's single question for you:
"You have no CTO and no technical co-founder. Is this a deliberate choice or a constraint?
Your answer changes my recommendation for Dimensions 4, 8, and 13."

[Press Enter to continue / type your answer]
```

This replaces assumption with ground truth, dramatically improving signal quality in later dimensions. Especially valuable for Dimensions 11–13 where all prior analyses converge.

---

### 7. `/ceo-sprint` Composite Skill

Chain CEO Before Sleep (discovery) with gStack (execution) into a single command:

```
/ceo-sprint <folder>
```

**What it does:**
1. CEO Before Sleep Full Analysis → identify the top problem + top opportunity
2. Auto-extract the #1 Most Urgent Task as a one-line feature request
3. Feed that into gStack `/plan-eng-review` → architecture + test plan
4. Feed that into gStack `/qa` → test the implementation
5. Feed that into gStack `/ship` → merge and PR

**Result**: From raw business materials to shipped code fix in one command. CEO Before Sleep becomes the strategic discovery layer; gStack becomes the execution layer. Together they form a complete founder operating system.

---

## Final Report Structure

```markdown
# CEO Before Sleep Analysis Report

> Target: [folder/project name]
> Date: [date]
> Model: claude-opus-4-6
> Analysis loops: 13 dimensions × 6 agents = 78 analyses
> Investability Score: [0–100] / [Grade]

---

## Executive Summary
[Core summary — 3-5 sentences answering the founder's 3 forcing questions from Phase 0]

## Overall Investability Score
[Weighted composite score with per-dimension breakdown]

---

## 1. Item Profitability
### Current Status (Reviewer)
### Issues Identified (Problem Finder) — RED / YELLOW / GREEN
### Proposed Solutions (Problem Solver) — Immediate / 3mo / 6mo+
### Market Benchmark (Researcher)
### CEO Advice (Consultant) — "If I were the CEO..."
### Competitor Shadow — Attack map + defensive countermove

...

## 11. Most Urgent Task
### The #1 Task — "If you don't do this, X will happen within Y days"

## 12. Biggest Risk
### Risk Matrix (Probability × Impact × Time Horizon)
### 12-Month Scenario Modeling (Base / Bull / Bear)

## 13. Crazy Founder's Advice
### The Path to $100B — "If I were you..."

## 14. Execution Retro [post-seed only]
### Planned vs. Shipped — The Execution Gap

---

## Phase 4: Action Plan
### This Week
### This Month
### This Quarter

---

## Appendix
### A. Materials Analyzed
### B. Founder's Forcing Questions (Phase 0 answers)
### C. Methodology
### D. Disclaimer
```

---

## Analysis Depth Control

| Mode | Dimensions | Agents | Total |
|------|-----------|--------|-------|
| Full Analysis (default) | 13 + Execution Retro | 6 | 84 |
| Core Analysis | 6 dims | 6 | 36 |
| Quick Scan | 3 dims | 6 | 18 |

Ask the user which level to run.

---

## Token Usage Estimate

| Item | Estimated Tokens |
|------|-----------------|
| Input context (material bundle) | 10,000–50,000 |
| Single agent call (input + output) | ~6,000–10,000 |
| Total agent calls (14 dims × 6 agents) | 84 calls |
| **Total estimated tokens** | **~500,000–840,000+** |

---

## Notes

- Run Phase 0 (Scope Alignment) before collecting materials — it takes 2 minutes and saves hours of misaligned analysis.
- If materials contain sensitive information (PII, passwords, API keys), advise the user to remove them first.
- This report is for reference only and does not replace professional legal or financial advice.
- In network-enabled environments, the Researcher agent's quality improves significantly with web search access.
