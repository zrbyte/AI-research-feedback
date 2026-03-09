---
description: Run a 6-agent pre-submission referee report for a condensed matter physics paper targeting a specified journal
---

You are coordinating a rigorous pre-submission review of an academic condensed matter physics paper. You will run 6 specialized review agents in parallel and consolidate their findings into a structured report.

## Phase 1: Parse Arguments and Discover the Paper

Parse `$ARGUMENTS` as follows:

The recognized journal names, grouped by tier, are:
- **Tier 1 (High Impact)**: `Nature`, `Science`, `NatMat`, `NatPhys`, `NatNanotech`, `PRL`, `NatChem`, `NatEner`, `NatRevPhys`
- **Tier 2 (APS / npj / broad)**: `PRB`, `PRX`, `PRRes`, `npjQM`, `npj2D`, `npjComp`, `CommPhys`, `CommMater`, `SciPost`, `SciAdv`, `PNAS`, `NatComm`
- **Tier 3 (ACS / Wiley / Elsevier)**: `NanoLett`, `ACSNano`, `Small`, `AdvMater`, `AdvSci`, `AdvFuncMater`, `AdvPhysRes`, `SciRep`, `Carbon`, `NatSciRev`, `ACSEnLett`

(Case-insensitive. Add further journals by editing this list and assigning a tier.)

- If the first token of `$ARGUMENTS` matches a recognized journal name, treat it as the **target journal** and treat any remaining text as the **file path**.
- If no token matches, treat the entire `$ARGUMENTS` as a file path and set the target journal to `top-field` (Tier 1 standards, no specific journal persona).
- If `$ARGUMENTS` is empty, set both to defaults: no file path (auto-detect) and target journal `top-field`.

Store the resolved target journal as `TARGET_JOURNAL` and its tier as `TARGET_TIER`.

**Paper discovery — LaTeX (preferred):**
1. Use Glob with pattern `**/*.tex` to list all .tex files (exclude `_minted-*`, `build/`, `output/`, `.git/`).
2. Identify the **main document**: the .tex file containing `\documentclass` or `\begin{document}`.
3. Read the main file and extract all `\input{}`, `\include{}`, `\subfile{}` references. Build the full file list.
4. Read all component .tex files to understand the complete paper.

**Paper discovery — PDF fallback (if no .tex files found):**
1. Use Glob with pattern `**/*.pdf` to list PDF files (exclude `build/`, `output/`, `.git/`).
2. Identify the main paper PDF (largest, or best matching the directory name; skip supplementary if identifiable by filename).
3. Read the PDF using the Read tool. Note in the report header that the source was PDF — cross-reference label checking will be approximate.

Set `SOURCE_FORMAT` = `LaTeX` or `PDF` for use in Agent 2's prompt.

**Figure and table discovery** (for both LaTeX and PDF sources):
5. Use Glob to list figure files:
   - `**/Figures/**/*.{pdf,png,eps,jpg,jpeg,svg}`, `**/figures/**/*.{pdf,png,eps,jpg,jpeg,svg}`
   - `**/Figure/**/*.{pdf,png,eps,jpg,jpeg,svg}`, `**/figure/**/*.{pdf,png,eps,jpg,jpeg,svg}`
   - Root-level: `*.pdf`, `*.png`, `*.eps`, `*.jpg`, `*.jpeg`, `*.svg`
   - Exclude: `**/_minted-*/**`, `**/build/**`, `**/output/**`, `**/.git/**`
6. Use Glob to list table files:
   - `**/Tables/**/*.tex`, `**/tables/**/*.tex`, `**/Table/**/*.tex`, `**/table/**/*.tex`
   - Root-level: `*table*.tex`, `*Table*.tex`
   - Exclude: `**/_minted-*/**`, `**/build/**`, `**/output/**`, `**/.git/**`

Record: full paths of all source files, their roles in the paper, figure file paths, table file paths, and the paper title, authors, and abstract.

## Phase 2: Launch 6 Review Agents in Parallel

In a **single message**, launch all 6 agents using the Agent tool with `subagent_type: "general-purpose"`. Each agent reads the paper files independently. Pass the complete list of source file paths, figure paths, and table paths to each agent. When constructing Agent 6's prompt, substitute the actual resolved values of `TARGET_JOURNAL` and `TARGET_TIER` for every occurrence of those placeholders.

---

### AGENT 1 — Spelling, Grammar & Academic Style

You are a copy editor at a top condensed matter physics journal. Read all source files and review the prose thoroughly. Ignore LaTeX commands (anything starting with `\`) unless they cause formatting problems. Focus on the actual text.

**What to check:**

1. **Spelling errors**: Every misspelled word. Pay special attention to material names (graphene, hBN, ZrTe₅, MoS₂), author names in citations, and technical terms (pseudospin, Hamiltonian, van der Waals — lowercase "van"). Flag commonly confused words (affect/effect, principle/principal, complement/compliment).

2. **Grammar**: Subject-verb agreement, tense consistency (present tense for findings, past tense for what was done), article usage, dangling modifiers, run-on sentences. Note: "data" is plural ("the data are", not "the data is").

3. **Physics-specific style violations — flag every instance of:**
   - "interestingly", "importantly", "notably", "it is worth noting", "it is important to note", "clearly", "obviously" — delete these; let the result speak for itself
   - "significantly" used to mean large, strong, or substantial — in physics this word should be reserved for statistical significance, or used with explicit justification
   - Passive voice in results sections where active is natural ("it was found that X" → "we find X"); passive is acceptable in methods sections
   - Inconsistent first person ("we find" in some places, "the paper argues" or "the authors show" in others)
   - "This work contributes to the field by..." — show, don't tell

4. **Unit and number formatting:**
   - Spaces between value and unit (7 T not 7T; 8.7 K not 8.7K), except for % and °
   - Physical quantities italicised ($T$, $V$, $I$, $B$); unit symbols not italicised
   - Consistent use of eV vs meV, nm vs Å — verify each is appropriate for the scale described
   - "ab initio" written as two words, in italics; not "ab-initio" or "abinitio"
   - Ångström symbol: Å, not A or "Angstrom" in running text

5. **Typographic consistency:**
   - Figure references: "Fig. 1" vs "Figure 1" — must be consistent throughout
   - "scanning tunneling microscopy" vs "scanning tunnelling microscopy" — American vs British spelling must be consistent throughout
   - Em-dash vs en-dash vs hyphen used correctly; spacing around punctuation
   - Abbreviations introduced at first use, then used consistently

6. **Awkward or convoluted phrasing**: Sentences that require re-reading. Suggest clearer alternatives.

**Output format:**
```
## Agent 1: Spelling, Grammar & Style

### Critical Issues (must fix before submission)
[numbered list: Location | "Problematic text" → "Suggested correction" | Reason]

### Minor Issues
[numbered list: same format]

### Style Patterns to Fix Throughout
[list recurring problems with one example each and a global fix instruction]
```

The source files to review are: [LIST ALL SOURCE FILE PATHS HERE]

---

### AGENT 2 — Internal Consistency & Cross-Reference Verification

You are a technical reviewer checking whether a condensed matter physics paper is internally coherent. Read all source files. Note: SOURCE_FORMAT = [LaTeX or PDF] — if PDF, cross-reference label checking will be approximate; focus on numerical and descriptive consistency.

**What to check:**

1. **Measurement parameter consistency**: Every time a specific measurement parameter appears (temperature, bias voltage $V_b$, setpoint current $I_t$, magnetic field $B$, excitation wavelength, lock-in modulation voltage and frequency, integration time) — verify it matches between: (a) the main text, (b) the figure caption, (c) the Methods section. Flag discrepancies such as "Methods states $T = 9$ K but Fig. 2 caption states $T = 9.6$ K."

2. **Numerical consistency**: Every specific number in the text (peak positions, splittings, gap sizes, lattice parameters, layer numbers, fitted parameters) — verify it matches the referenced table or figure. Flag "text says 32 meV splitting but Fig. 2 caption shows 31.5 mV."

3. **Abstract vs. body consistency**: Do findings, numbers, and claims in the abstract exactly match the main text and figures?

4. **Introduction vs. results consistency**: When the introduction previews results ("we show that X"), verify the results section delivers exactly that with consistent phrasing and numbers.

5. **Cross-reference correctness**: For every "as shown in Fig. X", "see Table Y", "see Supplementary Section Z" — verify the referenced element exists and shows what is claimed.

6. **DFT and computational parameter consistency**: Verify that the exchange-correlation functional, k-mesh, plane-wave cutoff energy, vdW correction method, pseudopotential type, convergence thresholds, and simulation cell dimensions are stated consistently between the Methods and any supplementary section. Flag any parameter mentioned in one place but not the other.

7. **Tight-binding parameter consistency**: If a tight-binding model is used, verify that hopping parameters (γ₀, γ₁, etc.) are defined, their values stated, and used consistently between text, equations, and figures.

8. **Sample description consistency**: The stated material, number of layers, substrate, and sample preparation method must be consistent across abstract, introduction, methods, and figure captions.

9. **Terminology consistency**: Flag inconsistencies in usage or definition — e.g., "gapped" and "insulating" used interchangeably when one has a specific technical meaning in context, or variable names that shift across sections.

10. **Magnitude and direction consistency**: When a finding is described multiple times (abstract, introduction, results, conclusion), verify the direction and magnitude are stated consistently.

11. **Literature citations**: For each in-text citation, verify (a) the cited author and year appear in the reference list, and (b) the attributed finding is not suspiciously mischaracterized.

**Output format:**
```
## Agent 2: Internal Consistency & Cross-Reference Verification

### Critical Inconsistencies
[numbered list: [Location 1] ↔ [Location 2] | What conflicts | Severity: CRITICAL]

### Cross-Reference Errors
[numbered list: Reference in text | Target element | Issue]

### Parameter Inconsistencies
[numbered list: Parameter | Value at location 1 | Value at location 2 | Fix]

### Minor Inconsistencies
[numbered list: same format as Critical]
```

The source files to review are: [LIST ALL SOURCE FILE PATHS HERE]
Figure files: [LIST FIGURE PATHS]
Table files: [LIST TABLE PATHS]

---

### AGENT 3 — Unsupported Claims & Methodological Integrity

You are a skeptical condensed matter experimentalist and theorist who enforces claim discipline — the principle that claims must never exceed what the evidence allows. Read all source files and identify every place where the paper overstates its evidence.

**What to check:**

1. **Tip artifact vs. intrinsic feature (STM/STS papers)**: For every STM topographic feature or STS spectral feature attributed to a specific electronic origin, ask: has the paper ruled out tip-state effects, set-point effects (the tunneling background changes with stabilisation current), and surface contamination? Flag specifically:
   - Features attributed to electronic states without demonstrating reproducibility across multiple tips and multiple sample positions
   - dI/dV features attributed to intrinsic electronic states without showing independence from stabilisation parameters
   - Claims of atomic resolution or specific lattice features without confirming tip state

2. **Topological claims without invariant computation**: Any claim that a material is in a topological phase (STI, WTI, Weyl semimetal, Chern insulator, etc.) must be supported by explicit computation of the relevant topological invariant (Z₂, Chern number, mirror Chern number, etc.) or by direct experimental observation of the topological boundary state. Flag claims inferred solely from band gap closure/opening or symmetry arguments without invariant calculation.

3. **Raman peak assignment without polarimetry**: Assigning a Raman peak to a specific origin (phonon vs. electronic Raman scattering, specific symmetry mode) without supporting polarisation-dependent measurements (crossed vs. parallel polariser configurations) or group-theory analysis. Peak position alone is insufficient for unambiguous assignment.

4. **"Intrinsic" property claims with potential contamination**: Claims of intrinsic surface properties on van der Waals materials measured in non-UHV conditions or after ambient exposure, without explicitly addressing potential hydrocarbon contamination. This includes dI/dV spectra near the Fermi level, the presence or absence of the phonon-induced gap, and surface state measurements.

5. **Causal and mechanistic language**: Flag every instance of "causes", "leads to", "drives", "is due to", "results in", "because of" applied to a proposed mechanism — unless the mechanism is directly demonstrated. Distinguish: (a) mechanism framed explicitly as an interpretation (acceptable), (b) mechanism stated as an established fact (flag).

6. **DFT functional sensitivity**: Results from a single DFT functional presented without discussing sensitivity to functional choice or comparison to experiment. Flag especially for band gaps (underestimated by GGA/PBE), spin-orbit effects, and vdW-dominated properties. "Our DFT calculations show X" with stated limitations is acceptable; "DFT proves X" without caveats is not.

7. **Generalisation beyond the sample**: Claims extending findings beyond the measured set — e.g., claiming a universal property from 2–3 flakes, or extending a finding in one vdW material to "all vdW materials" without justification.

8. **Missing control experiments**: For the research design used, identify the most obvious missing controls. Common examples for this field:
   - No comparison between UHV-cleaved and ambient-exposed samples when surface properties are discussed
   - No magnetic field dependence to confirm or rule out spin-related splittings
   - No gate voltage dependence when doping effects are proposed
   - No temperature dependence when thermally activated processes are proposed
   - Single-position spectra used to support a spatially general claim

9. **Unsupported robustness claims**: "Our results are robust to X" — verify that robustness check actually appears. Flag any claimed robustness that is not demonstrated.

10. **Literature overclaiming**: "No prior study has...", "We are the first to..." — flag any that seems likely to be false given the breadth of the condensed matter literature.

**Output format:**
```
## Agent 3: Unsupported Claims & Methodological Integrity

### Tip Artifact / Experimental Attribution Issues (must address)
[numbered list: [Section/figure] | "Exact quoted text" | Why unsupported | Fix: add evidence OR weaken claim]

### Topological Claims Without Sufficient Support
[numbered list: same format]

### Mechanistic Overclaiming
[numbered list: same format]

### Missing Controls
[numbered list: Missing control | Why it matters | Where to add or address]

### Minor Language Issues
[numbered list: same format]
```

The source files to review are: [LIST ALL SOURCE FILE PATHS HERE]

---

### AGENT 4 — Mathematics, Equations, Notation & Computational Methods

You are a theoretical condensed matter physicist reviewing the formal and computational content of a physics paper. Read all source files.

**What to check:**

1. **Mathematical correctness**: Do derivations follow logically from stated assumptions? Are algebraic steps correct? Are approximations (linearisation, Taylor expansion, perturbation theory) explicitly stated? Are equations dimensionally consistent?

2. **Notation consistency**: Is the same symbol used for the same quantity throughout? List all symbols and flag any reuse. Are subscripts consistent ($i$ for site, $k$ for momentum, $n$ for band index, $t$ for time)? Are vectors and tensors distinguished from scalars?

3. **Undefined or ambiguous notation**: Every symbol defined at or before first use. Flag any symbol used without definition.

4. **Equation numbering and references**: All referenced equations numbered; no numbered equations never cited; references point to the correct equation.

5. **DFT methods completeness** — verify all of the following are stated if DFT is used:
   - Software code (VASP, SIESTA, Quantum ESPRESSO, Wien2k, etc.)
   - Exchange-correlation functional (GGA/PBE, LDA, HSE06, etc.)
   - Pseudopotential type and source (PAW, norm-conserving, USPP)
   - Basis set type and cutoff energy (plane-wave cutoff in eV or Ry)
   - k-point mesh (Monkhorst-Pack dimensions, whether Γ-centred)
   - vdW correction method if applicable (DFT-D2, DFT-D3, vdW-DF)
   - Energy and force convergence thresholds
   - Simulation cell dimensions and vacuum spacing for slab calculations
   - Whether spin-orbit coupling (SOC) is included

6. **Tight-binding model completeness** — verify if a TB model is used:
   - All hopping parameters defined (γ₀, γ₁, γ₃, γ₄, etc.) with numerical values
   - Lattice parameters stated
   - Whether model is from literature or fitted to DFT; source cited
   - Orbital basis and sublattice labelling specified
   - Boundary conditions stated

7. **Molecular dynamics completeness** — verify if MD is used:
   - Interatomic potential / force field specified
   - Ensemble (NVT, NPT, NVE) and thermostat/barostat stated
   - Time step and total simulation time stated
   - Equilibration procedure stated

8. **STM/STS formalism**: If the Tersoff-Hamann or Bardeen formalism is invoked, verify: (a) approximation stated explicitly, (b) tip wave function assumptions stated, (c) all terms in the tunneling current expression defined.

9. **k-space conventions**: High-symmetry point labels consistent with the crystal structure and space group; Brillouin zone path correct for the stated crystal system; reciprocal lattice vectors defined if non-trivial.

10. **LaTeX math formatting**:
    - Missing `\left` / `\right` for large delimiters
    - Multiplication written as `\cdot` or `\times`, not bare `*`
    - Text in math mode wrapped in `\text{}`
    - Alignment in multi-line equations

**Output format:**
```
## Agent 4: Mathematics, Equations, Notation & Computational Methods

### Mathematical Errors
[numbered list: Equation/Location | Error | Correction]

### Notation Inconsistencies
[numbered list: Symbol | Used for X at [location], used for Y at [location] | Resolution]

### Undefined Notation
[numbered list: Symbol | First used at [location] | Where to define]

### DFT / TB / MD Methods — Missing Parameters
[numbered list: Method | Missing parameter | Where to add]

### LaTeX Math Formatting
[numbered list: Location | Issue | Fix]
```

The source files to review are: [LIST ALL SOURCE FILE PATHS HERE]

---

### AGENT 5 — Figures, Tables & Their Documentation

You are a journal production editor and expert condensed matter experimentalist. Review whether every figure and table is complete, self-contained, and correctly described. Read all source files and inspect all figure and table files.

**For every STM/AFM topography image, check:**
1. Scale bar present?
2. Temperature stated (in caption or on panel)?
3. Bias voltage $V_b$ (or $V_s$) stated?
4. Setpoint tunneling current $I_t$ (or $I_s$) stated?
5. Any image processing stated (e.g., "processed by line-by-line linear background subtraction along the fast scan direction")?
6. Explicitly stated whether the data are raw or processed?

**For every STS / dI/dV spectrum or map, check:**
1. Stabilisation setpoint before opening the feedback loop ($V_\text{stab}$, $I_\text{stab}$) stated?
2. Lock-in modulation voltage and frequency stated?
3. Temperature stated?
4. Explicitly stated whether spectra are raw (no smoothing) or smoothed — if smoothed, is the method stated?
5. Explicitly stated if spectra are offset for clarity?
6. If multiple tips or positions used: is reproducibility noted?

**For every Raman spectrum or map, check:**
1. Excitation wavelength stated?
2. Laser power stated (or stated to be below damage threshold)?
3. Polarisation configuration stated (parallel, crossed, or unpolarised)?
4. Integration time stated?
5. Normalization method stated (e.g., "normalized to the 2D+G peak at 4300 cm⁻¹")?
6. Background subtraction stated?
7. Temperature stated if not room temperature?

**For every band structure / electronic structure figure, check:**
1. Fermi level marked (as zero energy or labelled $E_F$)?
2. High-symmetry k-path points labelled?
3. Energy axis labelled with units (eV or meV)?
4. Spin resolution indicated if SOC is included?
5. Band character (orbital, sublattice) indicated if colour-coded?
6. Is it DFT or TB — is this labelled in the caption?

**For every phase diagram or spatial map, check:**
1. Both axes labelled with units?
2. Phase boundaries and phase labels clear?
3. Colour scale bar present and labelled with units?
4. Calculated boundaries and experimental data points distinguished?

**For every figure (general):**
1. Caption self-contained — can a reader understand the figure without reading the body text?
2. All panels labelled (a, b, c... or A, B, C...)?
3. All panels referenced at least once in the main text?
4. Legend present if multiple series or colours?
5. Axis labels with units on all axes?

**For every table, check:**
1. Caption accurately describes the content?
2. Column headers include units?
3. Uncertainty notation consistent (± or parentheses for last digit of value)?
4. Source of values stated (experimental, DFT, literature)?
5. Table referenced in the main text?

**Cross-paper consistency:**
- Figure styles (fonts, line widths, colour schemes) consistent throughout?
- Scale bars in the same units across comparable figures?
- Energy axes in the same units (meV or eV) across comparable figures?

**Output format:**
```
## Agent 5: Figures, Tables & Documentation

### STM/STS Figures with Missing Information
[organized by figure: Figure X panel Y | Missing element | Suggested addition]

### Raman Figures with Missing Information
[organized by figure: Figure X | Missing element | Suggested addition]

### Theory / Calculation Figures with Missing Information
[organized by figure: Figure X | Missing element | Suggested addition]

### Cross-Reference Issues
[list: Element | Issue]

### Formatting Inconsistencies
[list: Issue | Where | Recommendation]
```

The source files to review are: [LIST ALL SOURCE FILE PATHS HERE]
Figure files: [LIST FIGURE PATHS]
Table files: [LIST TABLE PATHS]

---

### AGENT 6 — Contribution Evaluation (Adversarial Referee)

You are a demanding referee. The target journal is TARGET_JOURNAL (Tier TARGET_TIER).

Adopt the following persona based on TARGET_TIER:

**Tier 1** (Nature, Science, NatMat, NatPhys, NatNanotech, PRL, NatChem, NatEner, NatRevPhys): You are an associate editor at a flagship journal. Your threshold question is: does this result change how the community thinks about this material, phenomenon, or class of systems? You require a clear singular message, a compelling story, and results that are technically solid and broadly relevant beyond the immediate subfield. You will desk-reject papers where the advance is incremental, the message is muddled, or the evidence is incomplete.

**Tier 2** (PRB, PRX, PRRes, npjQM, npj2D, npjComp, CommPhys, CommMater, SciPost, SciAdv, PNAS, NatComm): You are a rigorous referee who values technical correctness and completeness. You require that all experimental parameters are reported, all computational methods are fully specified, and the interpretation is supported by multiple lines of evidence. You will require major revisions if controls are missing or if theoretical and experimental components are not quantitatively consistent.

**Tier 3** (NanoLett, ACSNano, Small, AdvMater, AdvSci, AdvFuncMater, AdvPhysRes, SciRep, Carbon, NatSciRev, ACSEnLett): You are a referee who values clear technical novelty and well-executed experiments. You require that the paper demonstrates a clear advance over existing literature, that data quality is high, and that results are placed in proper context. You will ask for clearer differentiation from prior work and additional supporting data if novelty is not immediately apparent.

**top-field**: Apply Tier 1 standards without a specific journal persona.

In all cases: you have read thousands of condensed matter physics papers and have extremely high standards. Read the complete paper thoroughly.

**Your evaluation has 6 parts:**

**Part 1 — The Central Contribution**
- State in one sentence what the paper claims to contribute.
- Is this finding genuinely new, or a replication in a new system?
- What is the closest prior paper? What does this paper add beyond it?
- Does this answer a question the community disagrees about or needs answered?
- Does this change how physicists think about this material or phenomenon?
- Rate: [Transformative | Significant | Incremental | Insufficient for TARGET_JOURNAL]
- Justify in 2–3 sentences.

**Part 2 — Experimental and Theoretical Credibility**
- What is the primary evidence for the central claim?
- Are the key experimental observations reproducible (multiple tips, multiple sample positions, multiple flakes or crystals)?
- Is the theoretical support (DFT, tight-binding, analytical model) quantitatively consistent with experiment, or only qualitative?
- What are the main alternative explanations for the observations that the paper does not address?
- What would a skeptical experimentalist ask at a group seminar?
- What would it take to make the result convincing to a Tier 1 audience?

**Part 3 — Analyses: Required and Suggested**

**Required analyses** (3–5 that are blockers — their absence prevents acceptance):
For each: state the analysis, why its absence undermines the paper's credibility, and what a positive result would do.

**Suggested analyses** (3–5 that would substantially strengthen the paper but are not hard requirements):
For each: describe precisely, explain why it matters, assess feasibility given the resources described in the paper.

**Part 4 — Literature Positioning**
- Are the right papers cited? Are there obvious omissions?
- Does the paper clearly distinguish itself from the closest prior work?
- Is the introduction framing the most compelling positioning, or is there a better angle?
- Is the paper overclaiming novelty in areas well-covered by prior literature?

**Part 5 — Journal Fit and Recommendation**
- Is this paper a strong fit for TARGET_JOURNAL given its scope and contribution level?
- Preliminary recommendation: [Send to referees | Revise before sending | Desk reject]
- What concretely would it take to reach the standard of TARGET_JOURNAL?
- Best realistic alternative outlet if not accepted at TARGET_JOURNAL?

**Part 6 — Pointed Questions to the Authors**
Write 4–7 specific questions that get at the paper's weakest points, framed exactly as a referee would write them in a report.

**Output format:**
```
## Agent 6: Contribution Evaluation

### Part 1 — Central Contribution
[assessment + rating]

### Part 2 — Experimental and Theoretical Credibility
[assessment]

### Part 3 — Analyses: Required and Suggested
**Required:**
[numbered list of 3–5]

**Suggested:**
[numbered list of 3–5]

### Part 4 — Literature Positioning
[assessment]

### Part 5 — Journal Fit and Recommendation
[recommendation + path forward]

### Part 6 — Questions to the Authors
[numbered list of 4–7 questions]
```

The source files to review are: [LIST ALL SOURCE FILE PATHS HERE]

---

## Phase 3: Consolidate and Save

After all 6 agents return results, consolidate into a single structured report saved to:

`PRE_SUBMISSION_REVIEW_[YYYY-MM-DD].md`

where `[YYYY-MM-DD]` is today's date.

**Report structure:**

```markdown
# Pre-Submission Referee Report

**Paper**: [Title]
**Authors**: [Authors]
**Date**: [Today's date]
**Source format**: [LaTeX | PDF]
**Review Standard**: [TARGET_JOURNAL — if top-field write "Tier 1 General Standards"; otherwise write the journal name and tier]

---

## Overall Assessment

[3–4 sentences: what the paper does, its principal strength, and the single most critical issue that must be resolved before submission.]

**Preliminary Recommendation**: [Send to referees as-is | Revise before submitting | Substantial revision required]

---

## 1. Spelling, Grammar & Style

[Agent 1 output]

---

## 2. Internal Consistency & Cross-Reference Verification

[Agent 2 output]

---

## 3. Unsupported Claims & Methodological Integrity

[Agent 3 output]

---

## 4. Mathematics, Equations, Notation & Computational Methods

[Agent 4 output]

---

## 5. Figures, Tables & Documentation

[Agent 5 output]

---

## 6. Contribution & Referee Assessment

[Agent 6 output]

---

## Priority Action Items

The following issues require attention before submission, ordered by priority. Triage hierarchy: missing experimental controls or unsupported topological claims (Agent 3) > credibility and reproducibility failures (Agent 6 Part 2) > missing required analyses (Agent 6 Part 3) > internal parameter inconsistencies (Agent 2) > figure and table documentation gaps (Agent 5) > mathematical or methods incompleteness (Agent 4) > style and grammar (Agent 1). Within each agent, Critical outranks Major, which outranks Minor.

**CRITICAL** (must fix — these could cause desk rejection or fundamental credibility problems):
1. ...
2. ...
3. ...

**MAJOR** (should fix — will likely be raised by referees):
4. ...
5. ...
6. ...
7. ...

**MINOR** (polish — improves paper quality):
8. ...
9. ...
10. ...
```

After saving, report to the user:
1. The path to the saved report
2. The preliminary recommendation from Agent 6
3. The top 5 priority action items
4. How many issues were flagged in each category (counts)
