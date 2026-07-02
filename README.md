# Adversarial Review Skills

A collection of Claude Code skills for adversarial review of research work, implementing a "refute-first" philosophy throughout.

## Overview

This repository contains four complementary skills for research quality assurance:

### 1. Adversarial Review (Main Pipeline)
**End-to-end adversarial review pipeline** with three stages:
- **Stage 1**: Theory diagnosis (file-grounded)
- **Stage 2**: Empirical stress test (multi-lens + AND-gate)
- **Stage 3**: Whole-paper iteration (refute-first + actual revision)

**Key features**:
- Refute-first philosophy throughout
- External reviewers (heterogeneous perspective) + internal reviewers (precise details)
- Fully file-grounded
- Nested iteration with user checkpoints
- Model-agnostic (not bound to Codex)

**Trigger**: `adversarial review`, `对抗性审查`, `end-to-end review`, `stress-test this paper`

### 2. Auto Review Loop
**Autonomous multi-round review loop** that iterates: review → implement fixes → re-review, until the external reviewer gives a positive assessment or max rounds reached.

**Use when**: You want autonomous iterative improvement of a document until it passes review.

**Trigger**: `auto review loop`, `review until it passes`

### 3. Research Review
**External theory/framing critique** via Codex MCP. Gets deep critical review of research from GPT with maximum reasoning depth.

**Use when**: You want external feedback on research ideas, papers, or theoretical framing.

**Trigger**: `review my research`, `help me review`, `get external review`

### 4. Adversarial Review Empirical (Generic)
**Empirical stress testing** for any research project (de-identified, generic version). Tests specific empirical results/claims using multiple skeptic lenses.

**Use when**: You want to stress-test specific empirical findings before locking them.

**Trigger**: `adversarial review`, `stress-test this finding`, `is this finding robust`

## Installation

Copy the desired skill to your Claude Code skills directory:

```bash
# For example, to install the main adversarial-review skill:
cp "Adversarial Review.md" ~/.claude/skills/adversarial-review/SKILL.md
```

## Prerequisites

- Claude Code installed
- (Optional) Codex MCP for external reviewer functionality:
  ```bash
  claude mcp add codex -s user -- codex mcp-server
  ```

## Design Philosophy

All skills implement **refute-first** methodology:
- Treat research work as something to be overturned
- Only verified refutations carry weight (agreement is weak corroboration)
- File-grounded verification (not just narrative review)
- AND-gate aggregation (any fatal flaw kills the claim)

## File Structure

```
├── Adversarial Review.md                    # Main end-to-end pipeline
└── Sources/
    ├── Auto Review Loop.md                  # Autonomous iteration loop
    ├── Research Review.md                   # External theory critique
    └── adversarial-review-empirical.md      # Generic empirical stress test
```

## License

MIT

## Author

Created through collaboration between human researcher and Claude.
