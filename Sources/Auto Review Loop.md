---
name: auto-review-loop
description: Autonomous multi-round research review loop. Repeatedly reviews via Codex MCP, implements fixes, and re-reviews until positive assessment or max rounds reached. Use when user says "auto review loop", "review until it passes", or wants autonomous iterative improvement.
argument-hint: "[topic-or-scope]"
allowed-tools: Bash(*), Read, Grep, Glob, Write, Edit, Agent, Skill, mcp__codex__codex, mcp__codex__codex-reply
---

# Auto Review Loop: Autonomous Research Improvement

Autonomously iterate: review → implement fixes → re-review, until the external reviewer gives a positive assessment or MAX_ROUNDS is reached.

## Context: $ARGUMENTS

## Constants

- MAX_ROUNDS = 4
- POSITIVE_THRESHOLD: score >= 6/10, or verdict contains "accept", "sufficient", "ready for submission"
- REVIEW_DOC: `AUTO_REVIEW.md` in project root (cumulative log)

## Workflow

### Initialization

1. Read project narrative documents, memory files, and any prior review documents
2. Read recent experiment results (check output directories, logs)
3. Identify current weaknesses and open TODOs from prior reviews
4. Initialize round counter = 1
5. Create/update `AUTO_REVIEW.md` with header and timestamp

### Loop (repeat up to MAX_ROUNDS)

#### Phase A: Review

Send comprehensive context to the external reviewer:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N/MAX_ROUNDS of autonomous review loop]

    [Full research context: claims, methods, results, known weaknesses]
    [Changes since last round, if any]

    Please act as a senior OT/Strategy reviewer (ASQ/AMR/SMJ/Org Science level).

    1. Score this work 1-10 for a top venue
    2. List remaining critical weaknesses (ranked by severity)
    3. For each weakness, specify the MINIMUM fix (experiment, analysis, or reframing)
    4. Overclaiming check: is the contribution anchored on foundational classics when it
       should engage with more recent "middle" papers? Flag if so.
    5. State clearly: is this READY for submission? Yes/No/Almost

    Be brutally honest. If the work is ready, say so clearly.
```

If this is round 2+, use `mcp__codex__codex-reply` with the saved threadId to maintain conversation context.

#### Phase B: Parse Assessment

Extract from the response:
- **Score** (numeric 1-10)
- **Verdict** ("ready" / "almost" / "not ready")
- **Action items** (ranked list of fixes)

**STOP CONDITION**: If score >= 6 AND verdict contains "ready" or "almost" → stop loop, document final state.

#### Phase C: Implement Fixes (if not stopping)

For each action item (highest priority first):

1. **Theory refinement**: Sharpen mechanism, boundary conditions, or theoretical framing in the paper draft. Classify the theoretical tension type:
   - Scope condition violation, Mechanism substitution, Overlooked heterogeneity, Level-of-analysis gap, or Boundary condition on established mechanism
2. **Contribution threat check**: Before rewriting, search for papers that already make a similar claim. Use Semantic Scholar API + WebSearch with abstract theoretical terms (not setting-specific terms). If a threat is found, adjust the contribution to differentiate.
3. **Empirical analysis**: Write/modify Python scripts (pandas, statsmodels, linearmodels) for robustness checks, subgroup analyses, mechanism tests. Output tables in AER/SMJ format.
4. **Literature integration**: Find and incorporate additional citations to strengthen positioning. Use author neighborhood search (faculty pages, Google Scholar profiles) not just keyword search.
5. **Documentation**: Update project notes and review document

Prioritization rules:
- Prefer reframing/theory development over new data collection (which requires field work)
- Skip fixes requiring new data that isn't already collected (flag for manual follow-up)
- Always implement additional robustness checks on existing data (cheap, high impact)
- Prioritize addressing alternative explanations and identification concerns
- If a contribution threat is found, this takes highest priority — reframe before doing more empirical work

#### Phase D: Collect Results

If new analyses were run:
- Collect regression output from Python output
- Parse and format results into tables

#### Phase E: Document Round

Append to `AUTO_REVIEW.md`:

```markdown
## Round N (timestamp)

### Assessment
- Score: X/10
- Verdict: [ready/almost/not ready]
- Key criticisms: [bullet list]

### Actions Taken
- [what was implemented/changed]

### Results
- [experiment outcomes, if any]

### Status
- [continuing to round N+1 / stopping]
```

Increment round counter → back to Phase A.

### Termination

When loop ends (positive assessment or max rounds):

1. Write final summary to `AUTO_REVIEW.md`
2. **Outcome scenario analysis** — work through what happens under different empirical results:
   - **Scenario 1: Predicted pattern appears clearly.** State contribution (matches current framing).
   - **Scenario 2: Main effect appears but novel claim is weak.** What does the paper become? Which existing papers does this replicate?
   - **Scenario 3: Opposite of prediction appears.** Can the paper reframe as a generalizability test?
   - **Scenario 4: Baseline finding fails.** What can be salvaged? Be honest if the answer is "not much."
   For each: finding in one sentence, contribution, anchor papers engaged, journal fit.
3. Update project notes with conclusions
4. If stopped at max rounds without positive assessment:
   - List remaining blockers
   - Estimate effort needed for each
   - Suggest whether to continue manually or pivot

## Writing Style

Before writing any memos or review documents, read [references/style.md](../references/style.md) for writing constraints.

## Key Rules

- ALWAYS use `config: {"model_reasoning_effort": "xhigh"}` for maximum reasoning depth
- Save threadId from first call, use `mcp__codex__codex-reply` for subsequent rounds
- Be honest — include negative results and failed experiments
- Do NOT hide weaknesses to game a positive score
- Implement fixes BEFORE re-reviewing (don't just promise to fix)
- If a Python analysis takes > 30 minutes, launch it in background and continue with other fixes
- Document EVERYTHING — the review log should be self-contained
- Update project notes after each round, not just at the end

## Prompt Template for Round 2+

```
mcp__codex__codex-reply:
  threadId: [saved from round 1]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N update]

    Since your last review, we have:
    1. [Action 1]: [result]
    2. [Action 2]: [result]
    3. [Action 3]: [result]

    Updated results table:
    [paste metrics]

    Please re-score and re-assess. Are the remaining concerns addressed?
    Same format: Score, Verdict, Remaining Weaknesses, Minimum Fixes.
```
