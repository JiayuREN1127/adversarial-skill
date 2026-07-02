---
name: adversarial-review
description: End-to-end adversarial review pipeline with three stages — Stage 1 (theory diagnosis) → Stage 2 (empirical stress test) → Stage 3 (whole-paper iteration). Implements "refute-first" philosophy throughout, with external reviewers prioritized (heterogeneous perspective) followed by internal reviewers (precise details), fully file-grounded. Can run the full pipeline, or only Stage 1 (theory) or Stage 2 (empirical). Triggers: "adversarial review", "对抗性审查", "end-to-end review", "stress-test this paper", "review my research end-to-end".
argument-hint: "[project/paper/finding, or specify stage: theory/empirical/full]"
allowed-tools: Bash(*), Read, Grep, Glob, Write, Edit, Agent, Skill, Workflow, mcp__codex__codex, mcp__codex__codex-reply
---

# Adversarial Review — End-to-End Adversarial Scrutiny

**Refute-first, file-grounded, nested iteration, user checkpoints at key decisions.**

Treat the research work as something to be overturned — until it survives, or you explicitly decide to abandon a claim.

## Context: $ARGUMENTS

## Constants

- MAX_ROUNDS_STAGE3 = 5
- LENSES_DEFAULT: 9 universal empirical lenses (see Step 2)
- USER_CHECKPOINTS: After Stage 1, when Stage 2 produces dies, each Stage 3 round, any backtracking

## Design Principles

### Philosophy: Refute-first Across All Three Stages

- **Stage 1 — refute theory**: What are the holes in this theoretical framework?
- **Stage 2 — refute empirics**: Does this number/chart/claim hold up?
- **Stage 3 — refute the whole paper**: What's wrong with this manuscript?

"Agreement" is weak corroboration (reviewers share model priors with the analysis, Panickssery et al.); only **verified refutations** carry weight.

### Reviewer Composition: External-first + Internal Follow-up

| Reviewer | Role | File Access | Model |
|---|---|---|---|
| **External** | Heterogeneous perspective (design/identification/theory level) | File-blind (briefing only) | External LLM (Codex or other, **model-agnostic**) |
| **Internal** | Precise details (file-grounded verification/refutation) | Scripts + data + memory | In-project Claude |

Priority: **External first** (candidate threats), **then internal** (verification/refutation/execution). Switch to internal perspective when precise details are needed.

### Model Agnosticism

External reviewers are **not bound to Codex**. Implementation prioritizes `mcp__codex__codex`, but degrades gracefully if unavailable:
- Priority: `mcp__codex__codex` / `mcp__codex__codex-reply` (Codex MCP)
- Fallback: Any available external LLM MCP tool (e.g., `mcp__openai__*`, `mcp__anthropic__*`, etc.)
- Full degradation: Skip external review, internal only (explicitly note in report: "No external perspective this run")

### File-grounded Throughout

All stages can read scripts/data/memory. Rationale: File access is the binding constraint, not model identity (in-project Claude can read files + recompute — this is the core value that external models cannot provide).

## Pipeline Overview

```
User invokes /adversarial-review
   ↓
[Step 0] Mode selection + context gathering
   ↓
[Stage 1] Theory diagnosis (file-grounded)
   ├─ External: External theory expert diagnosis (file-blind briefing)
   ├─ Internal: Theory-empirical consistency, construct-measurement alignment, implicit assumptions
   ├─ Output: theory_diagnosis.md + claims_matrix.md
   └─ **User confirms theoretical framework** (key checkpoint)
   ↓
[Stage 2] Empirical stress test (multi-lens + AND-gate)
   ├─ External: External LLM provides candidate threats (file-blind)
   ├─ Internal: 9 universal lenses + project-specific lenses attack
   ├─ Execute: Run decisive checks on local data
   ├─ Lead-Judgment: False positive filtering
   ├─ Output: empirical_verdicts.md
   ├─ **User decides whether to save dies claims** (key checkpoint)
   └─ If new failure modes → learn → update lens_catalog.md
   ↓
[Stage 3] Whole-paper iteration (refute-first + actual revision)
   ├─ Each round: External review → Internal verification → Auto-fix → User confirmation
   ├─ After fix: Trigger Stage 2 mini re-check (ensure fix didn't introduce new problems)
   ├─ Any stage needs rerun → trigger nested backtracking
   └─ Stop: Surviving claims unrefuted / User says stop / MAX_ROUNDS / 2 rounds no change
   ↓
[Step 4] Termination + final report
   ├─ Output: final_report.md
   └─ Outcome scenario analysis
```

## Step 0 — Mode Selection + Context Gathering

### 0.1 Mode Selection

Check if $ARGUMENTS specifies stage:
- `stage:theory` or `only theory` → Run Stage 1 only
- `stage:empirical` or `only empirical` → Run Stage 2 only
- Other / unspecified → Full pipeline (Stage 1 + 2 + 3)

Use `AskUserQuestion` to confirm user intent (if unclear).

### 0.2 Context Gathering

Check if `adversarial-review/` directory exists:
- **Exists** → Ask: "Continue previous review? Or start fresh?"
  - Continue: Load existing `context.md` + stage reports
  - Start fresh: Clear directory, re-gather
- **Doesn't exist** → Create directory + gather project context

Gather the following (guide with `AskUserQuestion`):
1. **Core project claims** (1-3 sentences)
2. **Target journal** (for journal-heuristic matching)
3. **Script/data locations** (for file-grounded review)
4. **Project memory file locations** (if any)
5. **Known weaknesses** (issues user already knows about)
6. **Focus of this review** (theory/empirical/full paper / all)

Write to `adversarial-review/context.md`.

---

## Stage 1 — Theory Diagnosis (file-grounded)

> **Goal**: Diagnose theoretical framework vulnerabilities, generate claims matrix for Stage 2 stress testing.
> **Philosophy**: Refute theory — "What are the holes in this theoretical framework?"

### 1.1 External Perspective (file-blind)

Call external LLM (Codex or other), send narrative briefing:

```
[External theory review prompt]

Please diagnose the theoretical framework of the following research as a senior OT/Strategy reviewer (ASQ/AMR/SMJ/Org Science level):

[Project context: core claims, theoretical framework, method narrative, key results, known weaknesses]

Please identify:
1. Theoretical gaps — Is the mechanism well-specified? Are boundary conditions clear?
2. Identification strategy — Endogeneity, alternative explanations, causal hygiene
3. Narrative/positioning — Is the contribution correctly positioned in the literature?
4. Overclaiming — Is it anchored on classics when it should engage with recent "middle" papers?
5. Theoretical tension type classification (Scope violation / Mechanism substitution /
   Overlooked heterogeneity / Level-of-analysis gap / Boundary condition)

Be brutally honest.
```

### 1.2 Internal Perspective (file-grounded)

In-project Claude **reads actual scripts/data/memory**, performs:

1. **Theory-empirical consistency**: Can the data/code actually identify the theoretically claimed mechanism?
2. **Construct-measurement alignment**: How are theoretical constructs operationalized in code? Any gaps?
3. **Implicit assumption exposure**: Are hardcoded thresholds/assumptions in scripts theoretically justified?
4. **Alternative theory generation**: Based on actual data patterns, are there other theories that could explain?

### 1.3 Synthesized Output

Write to `adversarial-review/theory_diagnosis.md`:

```markdown
# Theory Diagnosis Report

## Theoretical Framework Assessment
- List of theoretical claims (itemized)
- Data/code support for each claim (specific file + line number)
- Theory-empirical consistency rating (strong/medium/weak)

## Theoretical Gaps
- Where mechanisms are not well-specified
- Where boundary conditions are undeclared
- Implicit assumptions (mined from code/data)

## Construct-Measurement Alignment
- Theoretical construct → actual measurement mapping
- Alignment rating + gap analysis

## Alternative Theories
- Which other theories could explain the observed data patterns
- Differentiation points from current theory

## Overclaiming Check
- Anchored on classics vs engaging recent literature
- True novelty of contribution
```

Write to `adversarial-review/claims_matrix.md`:

```markdown
# Claims Matrix

| Theoretical Claim | Empirical Test (which script/regression) | Expected Pattern | Supporting Condition | Refuting Condition |
|---|---|---|---|---|
| Claim 1 | script.py:line | β>0, p<.05 | ... | ... |
```

### 1.4 User Confirmation (key checkpoint)

Use `AskUserQuestion` to present theory diagnosis summary, ask user to:
- Confirm theoretical framework
- Mark which claims are "headline claims that must be defended"
- Mark which claims are "secondary claims that can be weakened"

---

## Stage 2 — Empirical Stress Test (multi-lens + AND-gate)

> **Goal**: Adversarial stress test each claim, produce verdicts.
> **Philosophy**: Refute empirics — "Does this number/chart/claim hold up?"

### 2.1 Extract Claims + Gate Lenses

**Step 2.0a: Extract claims**
- Extract atomic, falsifiable claims from claims_matrix.md
- Each claim: one sentence + direction/number + implied sample/condition/estimand

**Step 2.0b: User confirmation (if needed)**
- Present extracted claims, ask user to confirm/modify

**Step 2.0c: Lens applicability gating (rule-based, not discretionary)**
- Load `adversarial-review/lens_catalog.md` (if exists)
- Overlay default 9 universal lenses
- For each claim, mark each lens: applicable / NA (one-line reason)
- **Mandatory**: Each claim must run at least lens 8 (data artifact) + lens 5 (power)
- List excluded lenses + reasons (visible, auditable)

### 2.2 Default 9 Universal Lenses

| # | Lens | Type | Concrete Check |
|---|---|---|---|
| 1 | **Composition vs within-person** | ✅ Executable | within-person FE / constant-cohort re-estimate |
| 2 | **Weighting / aggregation** | ✅ Executable | volume- vs person- vs un-weighted |
| 3 | **Confound / omitted variable** | ✅ Executable | Add potential confound FEs |
| 4 | **Selection vs influence + causal hygiene** | ✍️ Design-only | immortal-time, censoring, reverse causality |
| 5 | **Power / cell counts** | ✅ Executable | Report n / events / k under the claim |
| 6 | **Definition / threshold robustness** | ✅ Executable | Sweep different thresholds/definitions/time windows |
| 7 | **Trace ≠ construct** | ✍️ Design-only | Is operationalized measure over-read as theoretical construct? |
| 8 | **Data artifact / preprocessing** | ✅ Executable | joins, missing values, ID types, cleaning steps |
| 9 | **Measurement spec** | ✍️/✅ Mixed | Is index built defensibly? Scaling stability |

✅ = Execute on data to settle; ✍️ = Design-level only, emit hedged caveat

### 2.3 External Perspective (file-blind)

Call external LLM, send self-contained briefing:

```
[External empirical review prompt]

Please review the following claims as a senior empirical-methods reviewer. No files attached — everything is below:
[Claim list + design + sample sizes + data caveats + actual numbers/tables + strongest counter-evidence already in hand]

First: What did this briefing likely omit?
Then, per claim, try to break it (alternatives, identification, immortal-time/censoring, composition, weighting)
and name the minimal robustness check that would settle it.
You cannot see the data, so flag concerns as "needs-check", not verdicts.
```

External perspective concerns **can only downgrade to `needs-robustness`** — it never saw the data, so it cannot `dies` a file-grounded claim. Its job is to **generate candidate threats**, which feed into internal execution.

### 2.4 Internal Perspective (file-grounded skeptic lenses)

For each claim, run applicable lenses. Each lens = fresh-context agent that **reads actual script + output + data + memory** and tries to **refute** the claim.

Run in parallel (Workflow/Agent); if no parallel primitive available, loop serially.

**Verdict bounds**:
- `dies`: **Only when refutation is verifiable from existing script/output WITHOUT new computation** (bug visible in code, hardcoded threshold with no sweep, etc.)
- `needs-robustness`: Any refutation requiring **re-estimation** caps here, naming exact spec
- `survives`: Lens finds no verifiable refutation

**Citation discipline**: Every refutation must cite (a) script path:line, (b) data fact, (c) numeric consequence, and self-label **VERIFIED** or **INFERRED**. INFERRED refutations are presumed false-positive until verified.

### 2.5 Execute Decisive Checks (resolve, don't prescribe)

For every **executable** flag (from 2.3 + 2.4) — **run the prescribed re-analysis on local data**, collapse verdict to `survives` / `dies` based on actual numbers.

This is what makes the tool a gate, not a TODO list.

### 2.6 Lead-Judgment (false positive filter)

Main model re-reads every remaining `dies` / `needs-robustness` / disagreement flag, uses actual script + data to classify:
- **CONFIRMED** — artifact really present (cite path:line / cell / number)
- **SPECULATIVE** — plausible but unverified → demote to `needs-robustness` + exact check to run
- **FALSE-POSITIVE** — script/data already rules it out → drop with one-line reason

Only CONFIRMED + SPECULATIVE propagate. Keep auditable "attacks considered and dismissed" list.

### 2.7 Merge (AND-gate) + Verdict

**Aggregation is non-compensatory** — lenses are orthogonal failure detectors, do not average.

Terminal verdicts (three):
- `dies` — any applicable lens returns dies (one fatal hole sinks it)
- `needs-robustness` — any lens returns needs-robustness and none dies
- `survives` — every applicable lens returns survives

**CONTESTED is a transient routing state**, not a terminal verdict. Must collapse to one of three before delivery.

### 2.8 Learning Mechanism (when new failure modes discovered)

If this review discovers new failure modes not in `lens_catalog.md`:
- Append to `adversarial-review/lens_catalog.md`
- Format: `| Failure mode name | Type | Concrete check | Project precedent |`
- Next run automatically loads as new lenses

### 2.9 Output + User Confirmation

Write to `adversarial-review/empirical_verdicts.md`:

```markdown
# Empirical Verdicts Report

## Verdict Table
| Claim | Applicable lenses (executed result) | External | Lead-Judgment | FINAL |
|---|---|---|---|---|

## Prioritized Robustness Checklist
Remaining checks, sorted by acceptance-lift-per-hour

## "What Survives" List
Each surviving claim + attacks it withstood

## Hedged Rewrite Suggestions
Hedged rewrite for each needs-robustness / CONTESTED claim
```

**User confirmation (key checkpoint)**:
- Present verdict summary
- For each `dies` claim, ask: "Save this claim (change theory/empirics)? Or abandon?"
- After user decision, update claims_matrix.md

---

## Stage 3 — Whole-Paper Iteration (refute-first + actual revision)

> **Goal**: Iteratively revise the whole manuscript until submittable or user says stop.
> **Philosophy**: Refute the whole paper — "What's wrong with this manuscript?"

### 3.1 Per-Round Iteration Structure

```
[3.1a] External review (file-blind, comprehensive review)
   ↓
[3.1b] Internal verification (read scripts/data, confirm changes landed)
   ↓
[3.1c] Parse assessment + action items
   ↓
[3.1d] Implement fixes (auto or ask user)
   ↓
[3.1e] Stage 2 mini re-check (if empirics changed)
   ↓
[3.1f] User confirmation (key checkpoint)
   ↓
[3.1g] Document this round
   ↓
Check stop condition → if not met, back to 3.1a
```

### 3.2 External Review (file-blind)

Call external LLM, send current draft + revision history:

```
[External whole-paper review prompt]

Please review the following draft as a senior OT/Strategy reviewer (Round N/MAX).

[Current draft + changes since last round + Stage 1 diagnosis + Stage 2 verdicts]

Please assess:
1. Are theory/mechanism/boundary conditions in place?
2. Is empirical results narrative accurate, not overclaiming?
3. Is contribution positioning correct?
4. Is writing/structure/narrative flowing?
5. Score 1-10 + Verdict (ready/almost/not ready)
6. Remaining critical weaknesses (ranked)

If ready, please state so clearly.
```

### 3.3 Internal Verification (file-grounded)

In-project Claude reads current draft + scripts/data, verifies:
- Do numbers cited in draft match latest script output?
- Did changes actually land (not just text, code too)?
- Were any new theory-empirical inconsistencies introduced?

### 3.4 Implement Fixes

Prioritized, fix types:

**Auto-fixable** (no user ask needed):
- Wording adjustments ("demonstrates" → "may suggest")
- Literature citation updates
- Format/structure optimization
- Overclaiming wording downgrade

**Need user ask** (key checkpoint):
- Theoretical framework restructuring
- Core claim weakening or deletion
- Empirical results re-interpretation
- Any modification affecting claims_matrix

### 3.5 Stage 2 Mini Re-check (if empirics changed)

If this round's modifications involved empirics (changed scripts/data/analysis):
- Trigger relevant Stage 2 lens rerun (not full battery, only applicable)
- Ensure fix didn't introduce new problems
- If new problems discovered → back to 3.4 to continue fixing

### 3.6 Nested Backtracking (Model C)

If this round discovers earlier stages need rerun:
- **Theoretical framework major restructuring** → trigger Stage 1 rerun
- **Empirical strategy major change** → trigger Stage 2 rerun
- Use `AskUserQuestion` to ask user if cost is worth it

### 3.7 Stop Conditions (OR)

- **A**: All surviving claims unrefuted in latest round
- **B**: User explicitly says "enough"
- **C**: Reached MAX_ROUNDS_STAGE3 = 5
- **D**: 2 consecutive rounds with no substantive change (converged)

### 3.8 Documentation

Each round appends to `adversarial-review/iteration_log.md`:

```markdown
## Round N (timestamp)

### External Review
- Score: X/10
- Verdict: [ready/almost/not ready]
- Key criticisms: [list]

### Internal Verification
- Number consistency: [pass/issues]
- Changes landed: [pass/issues]

### Implemented Changes
- [list]

### Stage 2 Mini Re-check (if applicable)
- Result: [survives/needs-robustness/dies]

### Status
- [continue / backtrack to Stage X / stop]
```

---

## Step 4 — Termination + Final Report

### 4.1 Final Report

Write to `adversarial-review/final_report.md`:

```markdown
# Final Report

## Surviving Claims List
Each surviving claim + attacks it withstood
→ These can be LOCKED and written unreservedly

## Fixed Issues List
All issues fixed from Round 1 to final round

## Remaining TODO
Issues not fixed + estimated effort

## Hedged Rewrite Suggestions
Hedged rewrite for each needs-robustness claim

## Outcome Scenario Analysis
- Scenario 1: Predicted pattern appears clearly → contribution (matches current framework)
- Scenario 2: Main effect appears but novel claim is weak → what does paper become? Who does it replicate?
- Scenario 3: Opposite of prediction appears → can it be reframed as generalizability test?
- Scenario 4: Baseline finding fails → what can be salvaged? Answer honestly.
```

### 4.2 Update Project Memory

Update project memory/notes with key conclusions (if any).

---

## Guardrails

### General
- **Refute-first throughout all three stages**: Every stage tries to refute first, not confirm
- **File-grounded priority**: Internal review must read actual files, not go by impression
- **External perspective heterogeneous**: External LLM provides design/identification-level external perspective, doesn't repeat internal lenses
- **Honesty**: Include negative results and failed fixes; don't hide weaknesses to game positive scores
- **Citation discipline**: Every refutation cites path:line + data fact + numeric consequence, labels VERIFIED/INFERRED
- **No fabrication**: Any refutation requiring re-estimation caps at `needs-robustness`, cannot assert "would likely flatten"

### Stage 3 Fix Boundaries
- **Auto**: Wording, literature citations, format, section structure
- **Need user ask**: Theoretical framework restructuring, core claim weakening/deletion, empirical results re-interpretation

### User Checkpoints
- After Stage 1 (theoretical framework confirmation)
- Stage 2 dies (whether to save that claim)
- Stage 3 each round (confirm changes)
- Any backtracking (confirm cost is worth it)

## Project Directory Structure

```
adversarial-review/
├── context.md              # Project context
├── claims_matrix.md        # Stage 1 output: claim-evidence mapping
├── lens_catalog.md         # Stage 2 accumulated: project-specific lenses
├── theory_diagnosis.md     # Stage 1 output: theory diagnosis
├── empirical_verdicts.md   # Stage 2 output: empirical verdicts
├── iteration_log.md        # Stage 3 output: iteration log
└── final_report.md         # Final report
```

## Key Rules

- Always prioritize external perspective (heterogeneous), then internal (precise)
- External reviewer **model-agnostic**: prioritize Codex MCP, fallback to any available external LLM, full degradation to internal only
- Internal lenses are skeptics, not independent reviewers — agreement is weak corroboration, only refutations carry weight
- AND-gate is non-compensatory — any fatal lens means dies, do not average
- Implement fixes **before** re-reviewing (don't just promise to fix)
- Stage 2 learning mechanism: append new failure modes to lens_catalog.md when discovered
- Document everything — reports should be self-contained
- Update project directory files at each stage, not just at the end

## Prompt Templates

### External Theory Review (Stage 1)
See Stage 1.1 section

### External Empirical Review (Stage 2)
See Stage 2.3 section

### Internal Skeptic Lens (Stage 2)
```
You are a fresh-context SKEPTIC reviewer (you share a model prior with the analysis — your AGREEMENT is worthless; only a verified REFUTATION counts). Attack this claim, do not confirm it:
  CLAIM: "<atomic claim + sample/estimand>"
Lens: <name + concrete check + EXECUTABLE or DESIGN-ONLY>
Read actual script <path> + output <path/CSV/figure> + data; consult memory <files>
Find the alternative / artifact / confound / power or timing problem that would OVERTURN it
Return: verdict (survives | needs-robustness | dies) — dies ONLY if verifiable from existing script/output WITHOUT new computation; anything needing re-estimation caps at needs-robustness with exact spec named
Cite script path:line + data fact + numeric consequence; label VERIFIED or INFERRED
```

### External Whole-Paper Review (Stage 3)
See Stage 3.2 section

### Round 2+ Follow-up (Stage 3, when using threadId)
```
[Round N update]

Since your last review:
1. [Change 1]: [result]
2. [Change 2]: [result]

Updated status: [paste relevant summary]

Please re-score and re-assess. Are remaining concerns addressed?
Format: Score, Verdict, Remaining Weaknesses, Minimum Fixes
```
