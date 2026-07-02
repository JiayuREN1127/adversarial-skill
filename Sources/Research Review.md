---
name: research-review
description: Get a deep critical review of research from GPT via Codex MCP. Use when user says "review my research", "help me review", "get external review", or wants critical feedback on research ideas, papers, or experimental results.
argument-hint: "[topic-or-scope]"
allowed-tools: Bash(*), Read, Grep, Glob, Write, Edit, Agent, mcp__codex__codex, mcp__codex__codex-reply
---

# Research Review via Codex MCP (xhigh reasoning)

Get a multi-round critical review of research work from an external LLM with maximum reasoning depth.

## Context: $ARGUMENTS

## Prerequisites

- **Codex MCP Server** configured in Claude Code:
  ```bash
  claude mcp add codex -s user -- codex mcp-server
  ```
- This gives Claude Code access to `mcp__codex__codex` and `mcp__codex__codex-reply` tools

## Workflow

### Step 1: Gather Research Context
Before calling the external reviewer, compile a comprehensive briefing:
1. Read project narrative documents (e.g., STORY.md, README.md, paper drafts)
2. Read any memory/notes files for key findings and experiment history
3. Identify: core claims, methodology, key results, known weaknesses

### Step 2: Initial Review (Round 1)
Send a detailed prompt with xhigh reasoning:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Full research context + specific questions]
    Please act as a senior Organization Theory / Strategy reviewer
    (ASQ, AMR, SMJ, Organization Science level). Identify:
    1. Theoretical gaps — is the mechanism well-specified and boundary conditions clear?
    2. Empirical concerns — identification strategy, endogeneity, alternative explanations
    3. Narrative and framing weaknesses — is the contribution positioned correctly in the literature?
    4. Whether the contribution is sufficient for a top management journal
    5. Missing robustness checks or supplementary analyses
    6. Overclaiming check — is the paper anchoring its contribution on foundational/classic papers
       (e.g., March 1991, Granovetter 1973) when it should be engaging with more recent "middle"
       papers that extended those classics? The contribution should respond to specific recent work,
       not claim to revise a foundational theory.
    Please be brutally honest.
```

### Step 3: Iterative Dialogue (Rounds 2-N)
Use `mcp__codex__codex-reply` with the returned `threadId` to continue the conversation:

For each round:
1. **Respond** to criticisms with evidence/counterarguments
2. **Ask targeted follow-ups** on the most actionable points
3. **Request specific deliverables**: experiment designs, paper outlines, claims matrices

Key follow-up patterns:
- "If we reframe X as Y, does that change your assessment?"
- "What's the minimum analysis/robustness check to satisfy concern Z?"
- "Please design the minimal additional empirical tests (highest acceptance lift per analysis hour)"
- "Please write a mock SMJ/ASQ review with scores"
- "Give me a results-to-claims matrix for possible experimental outcomes"

### Step 4: Convergence
Stop iterating when:
- Both sides agree on the core claims and their evidence requirements
- A concrete experiment plan is established
- The narrative structure is settled

### Step 5: Document Everything
Save the full interaction and conclusions to a review document in the project root:
- Round-by-round summary of criticisms and responses
- Final consensus on claims, narrative, and experiments
- Claims matrix (what claims are allowed under each possible outcome)
- Prioritized TODO list with estimated compute costs
- Paper outline if discussed

Update project memory/notes with key review conclusions.

## Theoretical Tension Types (use to diagnose weaknesses)

When evaluating or diagnosing theoretical contribution, classify the tension:

| Type | Description | Example |
|------|-------------|---------|
| Scope condition violation | Theory assumes X; X doesn't hold here | Theory assumes strong IP protection; setting has none |
| Mechanism substitution | Same outcome, different mechanism | Entry increases not from lower barriers but from subsidy capture |
| Overlooked heterogeneity | Effect varies on unaccounted dimension | Effect differs by founder prior experience |
| Level-of-analysis gap | Theory at one level, dynamics at another | Firm-level theory, but interesting dynamics are market-level |
| Boundary condition | Mechanism works differently under condition | Network benefits reverse when network is too homogeneous |

If the paper's tension doesn't fit any of these cleanly, flag it — it may be underspecified.

## Journal Heuristics

Tailor feedback to the target journal:
- **Administrative Science Quarterly:** Most theory-forward. Loves boundary conditions on established mechanisms. Mechanism must be novel and precisely specified.
- **Academy of Management Review:** Pure theory. Mechanism must be novel and well-specified. No empirics.
- **Strategic Management Journal:** Strategic implications must be clear. What should firms do differently?
- **Organization Science:** Theory-forward. Boundary conditions and mechanism precision.
- **Academy of Management Journal:** Balances theory and evidence. Framed as "what we didn't know before."
- **Management Science:** Clean identification, precise mechanism. Less concerned with broad theoretical contribution.
- **Strategic Entrepreneurship Journal:** Must advance entrepreneurship theory specifically.

## Writing Style

Before writing any memos or review documents, read [references/style.md](../references/style.md) for writing constraints.

## Key Rules

- ALWAYS use `config: {"model_reasoning_effort": "xhigh"}` for reviews
- Send comprehensive context in Round 1 — the external model cannot read your files
- Be honest about weaknesses — hiding them leads to worse feedback
- Push back on criticisms you disagree with, but accept valid ones
- Focus on ACTIONABLE feedback — "what analysis, reframing, or robustness check would fix this?"
- Document the threadId for potential future resumption
- The review document should be self-contained (readable without the conversation)

## Prompt Templates

### For initial review:
"I'm going to present a research paper for your critical review. Please act as a senior OT/Strategy reviewer for ASQ/AMR/SMJ/Organization Science..."

### For empirical robustness:
"Please design the minimal additional analyses or robustness checks that would most strengthen identification and address alternative explanations. Be specific about model specifications."

### For theoretical contribution:
"Please evaluate the theoretical contribution. Is the mechanism well-specified? Are boundary conditions clear? How does this advance theory beyond existing work? Suggest specific improvements to the theoretical argument."

### For paper structure:
"Please turn this into a concrete paper outline with section-by-section claims, theory development structure, and table/figure plan."

### For claims matrix:
"Please give me a results-to-claims matrix: what theoretical claim is supported under each possible pattern of empirical results?"

### For mock review:
"Please write a mock R&R review for SMJ/ASQ with: Summary, Strengths, Weaknesses, Questions for Authors, Recommendation (reject/R&R/accept), and What Would Move Toward Accept."
