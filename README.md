# CEO Before Sleep

> A multi-agent business analysis skill for Claude Code.

Six expert sub-agents analyze your business materials across 14 dimensions and deliver a CEO-grade report — the kind of deep thinking a founder does at night before sleep, when the noise is gone and only clarity remains. Includes an Investor Memo generator and a Competitor Radar for competitive intelligence.

---

## How It Works

**6 Agents × 14 Dimensions = 84 analyses — 14 parallel workers, 6 sequential roles each**

| Agent | Role |
|-------|------|
| Reviewer | Fact-based status assessment |
| Problem Finder | Hidden issues, risks, and gaps |
| Problem Solver | Concrete, actionable solutions |
| Researcher | Market benchmarks and validation |
| Consultant | Final CEO-level strategic advice |
| Competitor Shadow | How competitors would exploit each weakness |

Each agent reads all prior agents' outputs — depth compounds with every pass.

---

## Installation

Copy `CEO-before-sleep.md` to your Claude Code skills folder:

```bash
cp CEO-before-sleep.md ~/.claude/skills/
```

---

## Usage

Trigger by describing your intent in conversation:

```
CEO review of /path/to/my/project
business analysis of this startup
due diligence on /path/to/pitch/deck/folder
before sleep analysis of my business
investor memo for /path/to/my/project
competitor radar on /path/to/my/project --competitors "Stripe, Square, Brex"
```

Claude will automatically detect the request and run the appropriate analysis.

---

## Phase 0 Automation (Optional)

Use the Python script to collect all materials and generate the report skeleton before Claude begins analysis:

```bash
python scripts/run_analysis.py \
  --folder /path/to/project \
  --output /path/to/output/CEO_BeforeSleep_Report.md
```

**Requirements:**
```bash
pip install python-docx openpyxl python-pptx pymupdf
```

---

## Analysis Depth

| Mode | Dimensions | Agents | Total |
|------|-----------|--------|-------|
| Full Analysis (default) | 14 | 6 | 84 |
| Core Analysis | 6 | 6 | 36 |
| Quick Scan | 3 | 6 | 18 |

---

## The 14 Dimensions

1. Item Profitability
2. Talent Acquisition
3. Team Composition
4. Technology
5. Legal
6. Corporate Structure
7. Finance
8. Execution
9. GTM (Go-To-Market)
10. Sales
11. Most Urgent Task
12. Biggest Risk + 12-Month Scenario Modeling
13. Crazy Founder's Advice (path to $100B)
14. Execution Retro (post-seed only)

---

## What's New in v3.0.0

- **Parallel orchestration via Agent tool** ✅ — Phase 2 is no longer a sequential loop. The orchestrator spawns 14 dimension workers using the `Agent` tool in two parallel waves. Wall-clock time drops from ~14x to ~3x a single worker's time.
- **Wave 1** — 10 workers spawn simultaneously (Dims 1–10), each writing its own `dim_0N_*.md` file.
- **Wave 2** — 4 workers spawn simultaneously (Dims 11–14), receiving `wave1_summaries.md` as additional context.
- **Synthesis Agent** — Reads all 14 `dim_*.md` files, extracts dimension scores, computes the Composite Investability Score, and assembles `CEO_BeforeSleep_Report.md` and `CEO_ActionPlan.md`.
- **Shared context via `context_bundle.md`** — All materials + Phase 0 answers written to disk once; every worker reads the same file. No re-passing giant context strings in every prompt.
- **Dimension Worker Prompt Template** — Reusable template in `SKILL.md` for all 14 workers. Fill dimension name/description/path and dispatch.

## What's New in v2.3.0

- **Phase 6: Investor Memo** ✅ — Three-agent pipeline (Memo Writer, Devil's Advocate, Positioning Expert) auto-drafts a VC-ready memo from the analysis. Flags data gaps. Includes scoring gate for Investability Score < 40.
- **Phase 7: Competitor Radar** ✅ — Dedicated competitive intelligence loop (Intel Collector, Threat Mapper, Counter-Strategist). Outputs a heat map across all 14 dimensions, a 90-day threat calendar, and a defensive playbook. Invokable standalone as `/competitor-radar`.

## What's New in v2.2.0

- **Competitor Shadow Agent** ✅ — 6th agent analyzes how top 3 competitors would exploit each weakness, with a 90-day attack countermove
- **Quantified Scoring** ✅ — weighted 0–100 Investability Score with threshold rules (<40 re-run, 40–60 Core, >60 proceed)
- **Execution Retro** ✅ — Dimension 14 measures planned vs. shipped velocity, benchmarked against YC companies
- **Interactive Pause Mode** ✅ — `--interactive` flag makes Consultant pause after each dimension for one clarifying question
- **Phase 1.5 Browse Integration** ✅ — `/gstack:browse` screenshots the live product before analysis begins
- **`/ceo-sprint` Composite** — chains full CEO analysis with gStack sprint for discovery → execution in one command

---

## Token Budget

| Run | Tokens | Wall-clock |
|-----|--------|-----------|
| Wave 1: 10 workers in parallel | ~400,000–600,000 | 1 worker's time |
| Wave 2: 4 workers in parallel | ~160,000–240,000 | 1 worker's time |
| Synthesis Agent | ~30,000–50,000 | — |
| Full analysis total | ~640,000–980,000 | ~3 worker turns |
| + Phase 6: Investor Memo | +18,000–30,000 | — |
| + Phase 7: Competitor Radar | +30,000–60,000 | — |
| + gStack sprint handoff | +50,000–100,000 | — |

This is intentional — CEO-level depth requires it.

---

## File Types Supported

`.md` `.txt` `.csv` `.json` `.pdf` `.docx` `.xlsx` `.pptx` `.yaml` `.py` `.ts` and more.

---

## Disclaimer

This report is generated by an AI analysis tool for reference purposes only. For legal, financial, and tax matters, consult qualified professionals.
