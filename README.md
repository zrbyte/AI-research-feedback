# Using AI to get feedback on your research

A collection of [Claude Code](https://claude.ai/code) skills for condensed matter physics paper review, adapted for STM, Raman spectroscopy, and computational work (DFT, tight-binding, molecular dynamics). Originally developed by [Claes Bäckman](https://claesbackman.com) for economics; adapted for condensed matter physics by Péter Nemes-Incze.

## Skills

### `review-paper` — Pre-Submission Referee Report

Runs a rigorous pre-submission review of an academic paper. Six specialized review agents run in parallel and their findings are consolidated into a single structured report.

**What it reviews:**

| Agent | Focus |
|---|---|
| 1 | Spelling, grammar, and academic style |
| 2 | Internal consistency and cross-reference verification |
| 3 | Unsupported claims and methodological integrity |
| 4 | Mathematics, equations, notation, and computational methods |
| 5 | Figures, tables, and their documentation |
| 6 | Contribution evaluation (adversarial journal-specific referee) |

**Usage:**

From within the paper's directory, run:

```
/review-paper                        # auto-detect paper, Tier 1 general standards
/review-paper PRB                    # PRB referee persona
/review-paper NatPhys path/to/main.tex    # Nature Physics persona + explicit file path
```

Recognized journal names (case-insensitive):

| Tier | Journals |
|---|---|
| Tier 1 — High impact | `Nature`, `Science`, `NatMat`, `NatPhys`, `NatNanotech`, `PRL`, `NatChem`, `NatEner`, `NatRevPhys` |
| Tier 2 — APS / npj / broad | `PRB`, `PRX`, `PRRes`, `npjQM`, `npj2D`, `npjComp`, `CommPhys`, `CommMater`, `SciPost`, `SciAdv`, `PNAS`, `NatComm` |
| Tier 3 — ACS / Wiley / Elsevier | `NanoLett`, `ACSNano`, `Small`, `AdvMater`, `AdvSci`, `AdvFuncMater`, `AdvPhysRes`, `SciRep`, `Carbon`, `NatSciRev`, `ACSEnLett` |

If no journal is specified, Tier 1 general standards are applied. The file path is optional — the skill auto-detects the main `.tex` file if not provided.

The skill also works with PDF input: if no `.tex` files are found, it reads the PDF directly and proceeds with all six agents (cross-reference checking will be approximate).

The consolidated report is saved to `PRE_SUBMISSION_REVIEW_[YYYY-MM-DD].md` in the current directory.

**Domain-specific checks:**

The agents are calibrated for condensed matter experiment and theory. Highlights:

- **Agent 3** checks for tip-artifact vs intrinsic feature attribution in STM/STS, topological invariant computation (not just band gap closure arguments), Raman peak assignment without polarimetry, "intrinsic" surface property claims on potentially contaminated vdW materials, and DFT functional sensitivity.
- **Agent 4** verifies completeness of DFT parameters (code, functional, pseudopotential, k-mesh, cutoff, vdW correction, SOC, convergence criteria), tight-binding parameter definitions, and MD force field and ensemble settings.
- **Agent 5** has separate checklists for STM topography images (scale bar, T, V_b, I_t, processing), STS/dI/dV spectra (stabilisation setpoint, lock-in parameters, raw vs smoothed), Raman figures (excitation wavelength, polarisation, integration time, normalisation), and band structure plots (Fermi level, k-path labels, spin resolution).
- **Agent 6** applies a three-tier referee persona: Tier 1 asks whether the result changes how the community thinks about the material; Tier 2 demands quantitative theory-experiment consistency and complete methods reporting; Tier 3 evaluates novelty against existing literature.

**Customization:**

*Adding journals:* Open the skill file and add the abbreviation to the recognized names list in Phase 1, assigning it a tier. Agent 6 will adopt the appropriate persona automatically.

*Adding project-specific context:* Put paper-specific instructions in a `CLAUDE.md` file in your project directory — Claude Code reads this automatically and all agents will have access to it when reviewing your paper.

*Changing output path:* Edit the save path in Phase 3 of `review-paper.md`.

*Changing figure/table search paths:* Edit the Glob patterns in Phase 1 steps 5–6.

**Requirements:**

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with access to the `general-purpose` subagent (i.e., the Agent tool must be available).
- Papers in LaTeX format (preferred) or PDF.

**Installation:**

Copy [Paper-review/review-paper.md](Paper-review/review-paper.md) into your Claude Code skills directory:

```
~/.claude/commands/review-paper.md
```

or into a project-level skills directory (`.claude/commands/review-paper.md` relative to the project root) if you prefer per-project installation.

## License

MIT — free to use, adapt, and share.
