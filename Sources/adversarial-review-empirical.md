---
name: adversarial-review-empirical
description: Refute-first adversarial review of EMPIRICAL results / script outputs (any claim, finding, or figure/regression headed for a doc). Fresh-context Claude lens-reviewers try to BREAK each claim (composition, weighting, confound, power, threshold-robustness, trace≠construct, data-artifact, measurement-spec), mechanically settleable flags get EXECUTED on the local data, an external GPT/Codex pass adds a heterogeneous outside view, a Lead-Judgment pass filters false positives, and an AND-gate (any fatal lens kills; lens disagreement routes through a transient CONTESTED tie-break) merges to one of three terminal per-claim verdicts (survives / needs-robustness / dies) + the robustness checklist + a "what survives" list + hedged rewrites. Triggers: "adversarial review", "stress-test this finding/result", "review this result before I lock it", "is this finding robust".
argument-hint: "[script or result to review, e.g. 'analysis_output' or a claim]"
allowed-tools: Bash(*), Read, Grep, Glob, Write, Edit, Agent, Workflow, mcp__codex__codex, mcp__codex__codex-reply
---

# Empirical Adversarial Review — refute-first, hybrid, ground-truth-settled

Stress-test an **empirical** result before it gets locked or written into a doc.

**Design principle (why hybrid, not "cross-model only"):** the binding constraint here is
**file access, not model identity**. In-project Claude lenses can read the actual Python + data files +
memory and *recompute* — that file-grounded refutation IS the point and an external model can't do it.
So: **file-grounded refutation in-project; the external model = a heterogeneous outside view on
design/identification only.** Most of the gain comes from **fresh context + a refute-first prompt, not
from the engine** (that heterogeneity adds the last increment is a working hypothesis, not a proven
number). The Claude lenses are therefore **skeptic lenses, NOT independent reviewers** — they share a
model prior with the analysis's own generation (Panickssery et al.: LLM evaluators favor their own
generations), so their *agreement* is weak corroboration; only their *refutations* carry weight.

## Context: $ARGUMENTS

## Pipeline at a glance
**Step 0** scope & gate claims → **Step 1** skeptic lenses attack (in-project) → **Step 2** external GPT
adds threats (file-blind) → **Step 3** execute the decisive checks on real data → **Step 4** Lead-Judgment
filters false positives → **Step 5** AND-gate merge + deliver. Only the two LLM-call steps (1 and 2) have
verbatim prompt templates (bottom).

## When to use — and HOW MUCH (stakes × verifiability tier)
Reviews **results & script outputs**, not prose framing (theory/contribution → `research-review`;
iterate-until-passes → `auto-review-loop`). **Pick a tier; don't always run the full battery:**

| Tier | When | What runs |
|---|---|---|
| **LIGHT** | one figure, low stakes, not headed to a doc | 2–3 named applicable lenses, **no GPT**, no Lead-Judgment |
| **STANDARD** | a number entering the shared project doc | all applicable lenses + execute-checks (Step 3) + 1 GPT pass + Lead-Judgment |
| **HEADLINE** | a headline claim about to be **LOCKED**, or a multi-claim result | all applicable lenses per claim + file-grounded execute-checks + **≥2** GPT passes + mandatory Lead-Judgment |

**Refuse the full battery on a cheaply re-runnable result** — if a 5-minute re-run settles it (weighting
flip, group-FE, threshold sweep, cell-count, ID normalization), just RUN it (the Execute step, Step 3), don't
convene a tribunal. The analog of "crashes-on-run" (where adversarial review is useless) = **anything one re-run settles**.

## Inputs you must gather first
1. The **script(s)** that produced the result.
2. Its **console output / derived CSV / figure** (the actual numbers).
3. The **data caveats** from memory (Guardrails). If $ARGUMENTS doesn't name the script/result, ask which one.

## Prerequisite — Codex MCP (powers the external-GPT leg, Step 2)
The cross-model leg uses GPT via the Codex MCP server. One-time setup (same as `research-review` / `auto-review-loop`):
```bash
claude mcp add codex -s user -- codex mcp-server
```
This exposes `mcp__codex__codex` (a fresh, independent pass) and `mcp__codex__codex-reply` (a threaded follow-up on a saved `threadId`).
**Graceful degradation:** if Codex MCP is NOT configured, skip Step 2 and run **in-project-only** — the lenses,
execute-checks, and Lead-Judgment still function, but you LOSE the cross-model heterogeneity, so a HEADLINE claim's
final survival can no longer be gated on an outside view; say so explicitly in the deliverable. (LIGHT tier never calls GPT, so it is unaffected.)

---

## Step 0 — Extract claims, confirm them, gate the lenses
- Decompose the result into **atomic, falsifiable claims** — each one sentence with a direction/number AND
  its implied sample / conditioning / estimand.
- **Step 0.5 — confirm:** present the extracted claims back to the user (with each claim's implied
  sample/estimand) and get a yes/edit before attacking. Garbage claims → garbage verdicts.
- **Lens-applicability gate (rule-based, not discretionary — so the lethal lens can't be quietly dropped):**
  for each claim mark every lens applicable / NA with a one-line reason.
  - **Mandatory regardless of claim:** lens 8 (data-artifact), lens 5 (power/cells).
  - **Rule-triggered:** per-period rate/aggregate → lenses 1 + 2; group/role claim → lens 3;
    survival/exit-timing claim → lens 4; activity-log proxy → lenses 7 + 8; constructed index/measure → lens 9.
  - List the EXCLUDED lenses with justification (visible, auditable).

## Step 1 — Fresh-context skeptic lenses (refute-first)
For each claim run the applicable lenses. Each lens = a fresh-context agent that **reads the actual
script + output + data + memory** and tries to **refute** the claim. Run in **parallel** (Workflow/Agent);
**if no parallel primitive is available, loop them serially with fresh per-lens framing** — independence
comes from the refute prompt + clean context, not concurrency.

**Verdict is bounded by what reading alone can settle:**
- A lens may return **`dies` ONLY when the refutation is verifiable from the existing script/output WITHOUT
  new computation** (ID bug visible in code; k cells printed; threshold hard-coded with no sweep;
  immortal-time visible in the sample construction).
- Any refutation that needs a **re-estimate caps at `needs-robustness`**, naming the exact spec — it does
  NOT get to assert "would likely flatten it" (that fabrication is the confident-but-wrong output we exist
  to catch). It gets settled in the Execute step (Step 3) instead.
- **`default-to-refuted` is the per-lens internal stance, not a license to kill:** a flag escalates to
  `dies` only if the reviewer names the concrete mechanism + the spec/number that shows it. Unsupported
  skepticism caps at `needs-robustness`.

**Citation discipline (makes Lead-Judgment tractable):** every refutation must cite (a) script `path:line` of
the offending step, (b) the data fact (cell count / column / group), (c) the numeric consequence,
and label itself **VERIFIED** (recomputed/inspected) or **INFERRED**. INFERRED refutations are presumed false-positive until verified.

**The failure-mode lens catalog** (✅ = EXECUTABLE, settle on data; ✍️ = DESIGN-ONLY, hedged caveat + route out):

| # | Lens | type | The concrete check |
|---|------|------|--------------------|
| 1 | **Composition vs within-person** | ✅ | within-person FE / constant-cohort re-estimate |
| 2 | **Weighting / aggregation** | ✅ | volume- vs person- vs un-weighted |
| 3 | **Confound / between-group artifact** | ✅ | add group / function / size FE |
| 4 | **Selection vs influence + causal hygiene** | ✍️ | immortal-time, censoring, reverse causality, exit-as-selection |
| 5 | **Power / cell counts** | ✅ | report n / events / k under the claim |
| 6 | **Definition / threshold robustness** | ✅ | sweep different thresholds/time windows |
| 7 | **Trace ≠ construct** | ✍️ | is a behavioral measure over-read as a psychological/work construct? |
| 8 | **Data artifact** | ✅ | ID normalization, data gaps, sparse/echo cases |
| 9 | **Measurement spec** | ✍️/✅ | index built defensibly? (scaling stability) |

DESIGN-ONLY lenses (4, 7, and the design half of 9) are **not settleable on this observational log** →
they emit a hedged caveat + a pointer to `research-review`, and are **exempt from `dies`** (framing lives there, not here).

## Step 2 — External GPT pass (heterogeneous outside view — advisory, not veto)
Compile a **self-contained briefing**: the claim list, design, sample sizes,
the data caveats, the **actual numbers/tables/script text** (not just prose), AND the strongest counter-evidence
already in hand (anti-cherry-pick). First ask GPT to critique the **briefing itself** ("what did this summary
likely omit?"), then to try to break each claim. **MCP calls (see Prerequisite):** each pass is a fresh
`mcp__codex__codex` with `config:{"model_reasoning_effort":"xhigh"}`; save its `threadId`. For HEADLINE tier run
**≥2 INDEPENDENT passes** (separate fresh `codex` calls, NOT a threaded reply — independence is what makes
"replicates across draws" meaningful) and treat only flaws that **replicate** as gate-level. To drill into one
raised threat ("name the minimal check that would settle it"), continue THAT pass with `mcp__codex__codex-reply`
on its saved `threadId`.
**A briefing-only GPT concern can only downgrade to `needs-robustness` — it never saw the data, so it cannot
`die` a file-grounded claim.** Its job is to GENERATE candidate threats, which feed the Execute step (Step 3).

## Step 3 — Execute the decisive checks (resolve, don't just prescribe)
For every EXECUTABLE flag — from the skeptic lenses (Step 1) AND the GPT pass (Step 2) — **run the prescribed
re-analysis on the local data** (a scratch script; **normalize IDs first if needed**), then **collapse the verdict
to `survives` / `dies` on the actual number.** This is what makes the tool a gate and not a TODO list.
(`allowed-tools` grants Bash + Write.) Reuse cached data panels — don't re-load large files per check.

## Step 4 — Lead-Judgment (false-positive filter)  ← the load-bearing adjudication
The main model re-reads every remaining `dies` / `needs-robustness` / disagreement flag (those not already
settled by execution) and, **using the actual script + data**, classifies each:
- **CONFIRMED** — artifact really present (cite path:line / cell / number),
- **SPECULATIVE** — plausible but unverified → demote to `needs-robustness` + the exact check to run,
- **FALSE-POSITIVE** — script/data already rule it out (e.g. IDs already normalized; group-FE already in
  the spec) → drop with a one-line reason. *Must attempt verification before applying this label.*

Only CONFIRMED + SPECULATIVE propagate. Keep an auditable **"attacks considered and dismissed"** list.

## Step 5 — Merge (AND-gate), verdict, deliver
**Aggregation is non-compensatory — lenses are orthogonal failure detectors, NOT voters. Do not average.**
The terminal verdict is one of **three**: `dies` / `needs-robustness` / `survives`.
- **dies** if ANY single applicable lens returns `dies` (one fatal hole sinks it, even 8-1).
- **needs-robustness** if any lens returns needs-robustness and none dies.
- **survives** only if EVERY applicable lens returns survives. A HEADLINE claim's final survival is gated on
  the **executed checks + Lead-Judgment**, not on Claude consensus alone (Claude agreement is necessary, not sufficient).

**CONTESTED is a TRANSIENT routing state, not a terminal verdict.** When applicable lenses split on the *same*
alternative, OR Claude and GPT disagree, OR a `dies` rests on an unverified (INFERRED) assertion → mark CONTESTED
and **run the one decisive tie-breaker re-analysis** (the Execute step, Step 3), never auto-resolve to the
file-blind leg. CONTESTED **must collapse to one of the three terminal verdicts before delivery** — nothing ships as CONTESTED.

Deliver:
1. **Verdict table** — `| claim | lenses (executed result) | GPT | Lead-Judgment | FINAL |`
2. **Prioritized robustness checklist** — remaining checks, highest acceptance-lift-per-hour first (script change + expected table/figure).
3. **"What survives (defensible as-stated)"** — each surviving claim WITH the attacks it withstood → tells you what to LOCK and write un-hedged.
4. **Hedged rewrite** for every `needs-robustness`/CONTESTED claim (e.g., "may suggest …", not "demonstrates …").
5. Save to the project output directory → `_adversarial_review_<topic>_<YYYYMMDD>.md`. If a claim **dies/changes**, update the relevant memory file.

---

## Guardrails
- **Conservative / hedged language** (e.g., "may suggest" not "demonstrates"; match the output doc's own language); **don't over-interpret behavioral measures** as psychological constructs.
- **Normalize IDs** before any recompute touching dyadic/target metrics.
- **Consult project memories first** (if available) — they ARE the lens catalog's evidence base.

## Refute-first reviewer prompt template (Step 1)
```
You are a fresh-context SKEPTIC reviewer (you share a model prior with the analysis — your AGREEMENT is
worthless; only a verified REFUTATION counts). Attack this claim, do not confirm it:
  CLAIM: "<atomic claim + sample/estimand>"
Lens: <name + concrete check + EXECUTABLE or DESIGN-ONLY>.
Read the actual script <path> + output <path/CSV/figure> + data; consult memory <files>.
Find the alternative / artifact / confound / power or timing problem that would OVERTURN it.
Return: verdict (survives | needs-robustness | dies) — but `dies` ONLY if verifiable from the existing
script/output WITHOUT new computation; anything needing a re-estimate caps at needs-robustness with the
exact spec named. Cite script path:line + the data fact + the numeric consequence; label VERIFIED or INFERRED.
```

## External-GPT briefing template (Step 2)
```
Act as a senior empirical-methods reviewer for a top management journal. No files attached — everything is
below (claims + design + sample sizes + data caveats + ACTUAL numbers/tables
+ the strongest counter-evidence already in hand). First: what did this briefing likely OMIT? Then, per claim,
try to break it (alternatives, identification, immortal-time/censoring, composition, weighting) and name the
MINIMAL robustness check that would settle it. You cannot see the data, so flag concerns as "needs-check," not verdicts.
```

## Notes
- This skill **settles what it can on ground truth (the Execute step, Step 3) and hedges only what it genuinely can't** (design-only lenses).
- Complements, doesn't replace: `research-review` (external theory/framing critique), `auto-review-loop` (iterate-until-passes).
