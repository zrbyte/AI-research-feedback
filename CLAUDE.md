# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A Claude Code skill repository for academic economics paper review. The primary artifact is a single skill file: `Paper-review/review-paper.md`. There is no build system, test suite, or runtime code — the "code" is the skill prompt itself.

## Installation

Copy the skill file into the Claude Code commands directory:

```
~/.claude/commands/review-paper.md        # user-level (all projects)
.claude/commands/review-paper.md          # project-level (relative to paper's root)
```

## Usage

Run from within the paper's directory:

```
/review-paper                        # auto-detect main .tex, generic standards
/review-paper QJE                    # QJE-specific referee persona
/review-paper JF path/to/main.tex    # Journal of Finance persona + explicit file
```

Recognized journal names (case-insensitive): `AER`, `QJE`, `JPE`, `Econometrica`, `REStud`, `JF`, `JFE`, `RFS`, `JFQA`, `AEJMacro`, `JME`, `RED`.

Output is saved to `PRE_SUBMISSION_REVIEW_[YYYY-MM-DD].md` in the current working directory.

## Skill Architecture (review-paper.md)

The skill runs in three phases:

**Phase 1 — Argument parsing & paper discovery**: Resolves the journal and file path from `$ARGUMENTS`. If no file path is given, it globs for `.tex` files, identifies the main document (the one containing `\documentclass` or `\begin{document}`), follows all `\input{}`/`\include{}`/`\subfile{}` references, and builds lists of figure and table file paths.

**Phase 2 — 6 parallel agents**: All six agents are launched in a single message using the `general-purpose` subagent type. Each receives the full list of `.tex`, figure, and table paths. The agents cover:
- Agent 1: Spelling, grammar, academic style
- Agent 2: Internal consistency and cross-reference verification
- Agent 3: Unsupported claims and identification integrity
- Agent 4: Mathematics, equations, notation
- Agent 5: Tables, figures, and their documentation
- Agent 6: Contribution evaluation as an adversarial journal referee (persona determined by `TARGET_JOURNAL`)

**Phase 3 — Consolidation**: Agent outputs are merged into a single structured report with an Overall Assessment, all six agent sections, and a ranked Priority Action Items list. Triage order: identification failures > missing required analyses > internal inconsistencies > table/figure documentation > math errors > style.

## Extending the Skill

- **Add a journal**: Add its abbreviation to the recognized names list in Phase 1 of `review-paper.md`. Agent 6 adopts the appropriate editorial persona automatically based on its training knowledge.
- **Add project context**: Put paper-specific instructions in a `CLAUDE.md` in the paper's project directory — all agents will inherit it automatically.
- **Change output path**: Edit the save path in Phase 3 of `review-paper.md`.
- **Change figure/table search paths**: Edit the Glob patterns in Phase 1 steps 5–6.
