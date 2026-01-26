# Multi-Objective-Peptide-Discovery-Platform
A unified, one-file peptide discovery platform that uses active learning to optimise affinity, stability, and solubility, and exports CRO-ready synthesis and assay artifacts.
A single, end-to-end **peptide discovery platform demo** that runs iterative **Design → Predict → Propose → Measure → Learn** cycles for **multi-objective peptide optimisation** across **Affinity**, **Stability**, and **Solubility**, and automatically exports **CRO-ready assay artifacts** (candidate lists, synthesis order sheets, 96-well plate maps, and a ranked screening decision report).

## Summary

Peptide discovery often stalls at the interface between computational design and experimental execution, where candidate prioritisation, manufacturability constraints, and assay logistics must be integrated into a single, decision-ready workflow. Here we present a unified, one-file proof-of-concept peptide design platform that performs closed-loop Design, Predict, Propose, Measure, and Learn cycles using multi-objective optimisation across affinity, stability, and solubility. The platform combines sequence featurisation, ensemble RandomForest surrogates with uncertainty, Bayesian-style acquisition (UCB, EI, and Pareto-weighted EHVI-lite), diversity constraints, and explicit manufacturability liabilities including deamidation hotspots, Asp–Pro cleavage risk, oxidation-sensitive residues, cysteine liabilities, extreme charge, and aggregation proxies. In simulation mode, an oracle generates assay-like measurements across iterative rounds, producing learning curves and evolving Pareto landscapes. In real-assay mode, the platform ingests primary dose–response or single-point screening data, computes replicate quality flags, estimates plate performance using the Z′-factor, fits 4-parameter logistic curves without SciPy, and proposes the next synthesis batch. Outputs include CRO-ready candidate lists, synthesis order sheets, and automated 96-well plate maps, enabling immediate translation from computational prioritisation to experimental execution. This work demonstrates an integrated template for peptide discovery pipelines that treat model predictions as laboratory decisions, rather than standalone scores, supporting practical multi-parameter optimisation and reproducible assay planning.

## Introduction

Peptides occupy a practical middle ground between small molecules and biologics, offering tunable binding surfaces, comparatively rapid synthesis, and increasingly diverse therapeutic and diagnostic applications. However, peptide optimisation remains constrained by a familiar reality, improving one property often degrades another. For example, increasing hydrophobicity can strengthen binding and membrane engagement, but frequently reduces solubility and increases aggregation propensity. Likewise, sequence motifs that enhance target interaction can create chemical liabilities such as deamidation hotspots, oxidation-sensitive residues, or cleavage-prone junctions, leading to instability during synthesis, storage, and assay handling.

Modern discovery workflows therefore require multi-parameter optimisation, where candidates are judged simultaneously across potency-related metrics and developability constraints. This demands a decision framework that can integrate predictive modelling, uncertainty, diversity management, and explicit manufacturability penalties into a repeatable “candidate-to-assay” loop. Active learning strategies are increasingly used for discovery because they focus experimental effort on candidates expected to produce the largest improvement in outcomes per assay cycle, rather than exhaustive screening. In practical terms, this means selecting a small batch of candidates that are predicted to be strong, uncertain in informative regions of sequence space, and chemically viable for synthesis and assay deployment.

To address this need, we developed a unified, one-file peptide discovery platform that executes a closed-loop workflow end-to-end, producing both computational analyses and CRO-ready experimental artifacts. The platform supports two modes. In simulation mode, an oracle generates realistic assay-like measurements for iterative development and benchmarking. In real-assay mode, the pipeline ingests experimental assay tables, performs quality control, fits dose–response curves, assembles a canonical measured dataset, and proposes the next experimental batch along with synthesis and plate-planning outputs.

## What this platform does
Implements a closed-loop peptide discovery workflow with:

- **Active learning loop**, proposes improved batches each round using Bayesian-style acquisition.
- **Multi-objective optimisation**, balances **Affinity + Stability + Solubility**.
- **Ensemble uncertainty**, predicts mean and standard deviation using bootstrap RandomForest surrogates.
- **Acquisition strategies**, supports **UCB**, **EI**, and **EHVI-lite** (Pareto-weighted UCB proxy).
- **Pareto front detection**, identifies non-dominated candidates in 3-objective space.
- **Diversity constraints**, greedy selection using Levenshtein distance or a fast proxy mode.
- **Manufacturability and liabilities**, penalises sequences with common developability risks:
  - Deamidation hotspots (NG, NS, QG, QN)
  - Asp–Pro cleavage motif (DP)
  - Oxidation risk (high M/W)
  - Cysteine liability (excess C)
  - Extreme net charge
  - Aggregation proxy (high hydrophobicity/aromaticity)
- **CRO-ready outputs**, exports:
  - Proposed candidates per round (`proposed_candidates_roundXX.csv`)
  - Synthesis order sheet (`roundXX_synthesis_order_sheet.csv`)
  - 96-well plate map (`roundXX_plate_map_96well.csv` + `.png`)
  - Final ranked screening report (`screening_report_top50.csv`)
- **Optional real assay ingestion**, if assay CSVs are provided:
  - Replicate CV QC flags
  - Plate QC Z′ factor (if controls exist)
  - Dose–response 4PL fitting without SciPy (grid + refinement)
  - Proposes the next synthesis batch from measured data

## Repository structure (outputs)

When you run the script, an output folder is created (default: `out_iso_peptides/`) containing:

### Figures
Grouped into five figure sets:

- **Figure 1:** `fig_dist_affinity.png`, `fig_dist_stability.png`, `fig_dist_solubility.png`
- **Figure 2:** `fig_learning_curve.png`
- **Figure 3:** `fig_roundXX_pareto_2d.png` (Round 00 → Round 08)
- **Figure 4:** `fig_roundXX_pareto_3d.png` (Round 00 → Round 08)
- **Figure 5:** `roundXX_plate_map_96well.png` (Round 01 → Round 08)

### Tables (CSVs)
- `learning_history.csv`
- `measured_table_roundXX.csv`
- `proposed_candidates_roundXX.csv`
- `roundXX_synthesis_order_sheet.csv`
- `roundXX_plate_map_96well.csv`
- `screening_report_top50.csv`
- `run_summary.json`

## Quickstart

### 1) Install dependencies

pip install numpy pandas matplotlib scikit-learn

## How selection works (high-level)
Each round:
	1.	Expand candidates by mutating and recombining top-performing sequences.
	2.	Train surrogate models for Affinity, Stability, and Solubility.
	3.	Predict mean and uncertainty on candidate pool.
	4.	Compute acquisition score (UCB, EI, or EHVI-lite).
	5.	Subtract developability penalties and apply hard gating.
	6.	Enforce diversity selection to avoid near-duplicates.
	7.	Export candidates + CRO artifacts.

# Model Card: Unified Peptide Design Platform (One-File PoC)

## Model Overview
**Model name:** Unified Peptide Design Platform (RandomForest Ensemble Surrogates)  
**Version:** v1.0  
**Primary use:** End-to-end peptide candidate prioritisation using **active learning** and **multi-objective optimisation**.  
**Objectives optimised:** **Affinity**, **Stability**, **Solubility** (maximise all three).  
**Core idea:** Predict peptide performance with ML surrogates, propose the next best batch to measure, then learn from assay feedback.

This is a **proof-of-concept platform demo** that can run in:
1) **Simulation mode** (uses an internal “oracle” as a stand-in for real assays).  
2) **Assay ingestion mode** (ingests real experimental CSV results, performs QC, then proposes the next batch).

## Intended Use
### Intended Users
- Computational chemistry and ML scientists working on peptide screening.  
- Discovery scientists designing peptide hit-finding campaigns.  
- Teams preparing **CRO-ready** synthesis and screening batches.

### Intended Use Cases
- Prioritising peptides for **next-round synthesis and testing**.  
- Multi-objective ranking when peptides must balance potency and developability.  
- Producing operational outputs such as:
  - **Next-batch proposal list (CSV)**
  - **Synthesis order sheet (CSV)**
  - **96-well plate maps (CSV + PNG)**
  - **Top-50 decision report (CSV)**
  - **QC tables** (Z’ factor, replicate CV flags)

### Out-of-Scope / Not Intended Use
- Clinical decision-making.  
- Claims of biological mechanism or binding structure validation without experiments.  
- Safety, toxicity, immunogenicity, or PK/PD prediction beyond simple proxies.

## Model Architecture and Components
### Surrogate Predictors
Each objective is modelled with an **ensemble of RandomForestRegressor models**:
- Separate ensemble for **Affinity**
- Separate ensemble for **Stability**
- Separate ensemble for **Solubility**

**Uncertainty estimate:** Standard deviation across bootstrap ensemble predictions.

### Acquisition Functions (BO-like selection)
Candidates are ranked using a composite multi-objective acquisition score:
- **UCB**: `μ + βσ`  
- **EI**: Expected Improvement against current best composite score  
- **EHVI-lite**: Pareto-weighted exploration proxy to encourage balanced candidates

### Feature Engineering
Each peptide sequence is converted into numeric features:
- Amino-acid composition (20 features)
- Physicochemical descriptors, e.g. length, hydropathy stats, charge, aromaticity
- Structural proxies, e.g. helix propensity proxy, hydrophobic autocorrelation proxy
- Motif features (e.g. RGD-like or patterned motifs)
- Optional **2-mer k-mer counts** (400 features) for local sequence context

## Data and Inputs
### Input Format
**Primary input:** peptide amino-acid sequences (strings).  
Examples: `"ACDEFGHIKLMNPQRS"`, `"KKWXXW..."` (letters must be valid amino acids).

### Simulation Mode
- Starts from a randomly generated peptide library.  
- “Oracle” assigns synthetic affinity/stability/solubility with noise to mimic experimental variability.  
- Useful for **pipeline testing** and end-to-end demonstration.

### Real Assay Mode (CSV Ingestion)
Primary assay requires columns:
- `peptide_id`
- `concentration_uM`
- `response`
- `replicate`

Optional metadata:
- `plate_id`, `well`, `assay_name`
- `is_pos_ctrl`, `is_neg_ctrl`

Optional secondary assays:
- Stability CSV with `stability_score`
- Solubility CSV with `solubility_score`

## Outputs and Artifacts
The platform produces CRO-style outputs including:
- `proposed_candidates_roundXX.csv`  
  Candidate list with predictions, uncertainty, liabilities, and acquisition score.
- `roundXX_synthesis_order_sheet.csv`  
  Procurement-ready sheet including peptide IDs, purity specs, scale, and notes.
- `roundXX_plate_map_96well.csv` and `roundXX_plate_map_96well.png`  
  Plate layout for screening.
- `screening_report_top50.csv`  
  Final ranked shortlist integrating multi-objective scoring and Pareto annotation.
- `assay_primary_summary.csv` (assay mode)  
  pIC50 or single-point summary, replicate CV flags, 4PL fit diagnostics.
- `assay_plate_qc.csv` (assay mode)  
  Z’ factor estimation (if positive and negative controls are supplied).
- `run_summary.json`  
  Run metadata and best-performing candidates.

## Liability and Manufacturability Filters
To reduce poor developability candidates, the platform applies penalties/gates for:
- **Deamidation hotspots** (e.g. NG, NS, QG, QN)
- **Asp–Pro cleavage** risk (DP)
- **Oxidation risk** (high Met/Trp burden)
- **Cysteine risk** (too many Cys)
- **Extreme net charge**
- **Aggregation proxy** (excess hydrophobicity/aromaticity)

These filters are implemented as:
- **Soft penalty** in ranking (subtract from acquisition score)  
- **Optional hard gate** (drop candidates beyond max allowed liability flags)

## Evaluation and Performance Notes
### What “good performance” means here
- The model is considered useful if it proposes batches that **improve the best observed peptides** over successive rounds.  
- Key success indicators include:
  - Increasing best affinity across rounds
  - Better multi-objective Pareto front coverage
  - Higher composite score candidates with fewer liabilities

### Known Limitations
- Random forests do not learn true structural binding physics.  
- Feature set is lightweight, it approximates structure using sequence-level proxies only.  
- Liability filters are heuristic, they do not replace developability experiments.  
- In real assay mode, the sequence-to-peptide mapping must be correct, otherwise training will be invalid.

## Risks and Biases
- The model can prefer motifs or patterns over-generalised from early data (selection bias).  
- If assay noise is high, uncertainty may be underestimated or overestimated.  
- The composite weighting scheme can bias proposals toward one objective if the scales differ.

To reduce risk:
- Use replicate CV and plate QC metrics to flag unstable assay datasets.  
- Consider re-scaling objectives (z-score) if assay scales differ strongly.  
- Use diverse selection (Levenshtein distance or fast proxy) to avoid redundancy.

## Usage (Examples)
### Simulation Mode
python peptide_platform_unified.py --outdir out_iso_peptides --seed 7 --rounds 8 --acq UCB


# Datasheet for Dataset: Unified Peptide Design Platform (Measured + Proposed Batches)

## Dataset Summary
This dataset supports an end-to-end **peptide discovery workflow** where peptide sequences are iteratively **designed, predicted, selected, measured, and refined** across multiple rounds. It contains both **measured peptide outcomes** (from simulation-oracle or real assays) and **model-proposed candidates** for the next experimental batch, alongside **CRO-ready operational artifacts** such as synthesis order sheets and plate maps.

**Primary task:** Multi-objective peptide optimisation.  
**Optimisation targets:** **Affinity**, **Stability**, **Solubility** (maximise all three).  
**Data modalities:** peptide sequences, numeric assay-like scores, ML predictions, QC metrics, and developability flags.

## Motivation
Peptide hit discovery rarely optimises a single property. A peptide can be potent but unstable, or stable but poorly soluble. This dataset exists to demonstrate a platform workflow that balances these competing constraints while producing outputs that are directly usable for **synthesis ordering** and **screening execution**.

## Composition
### What does each record represent?
Depending on the file, each row represents either:
1) A **measured peptide** with experimentally observed (or simulated) performance outcomes, or  
2) A **proposed peptide** with predicted performance and ranking metadata.

### What is included?
The dataset includes multiple CSV outputs typically grouped into:

#### A) Measured Peptides (Ground Truth Table)
- `measured_table_round00.csv`
- `measured_table_round01.csv` ... `measured_table_roundXX.csv`
- `measured_table_from_assays_round00.csv` *(assay ingestion mode only)*

Each row contains:
- peptide `sequence`
- measured `affinity`, `stability`, `solubility`
- `round` number (iteration index)

#### B) Proposed Peptides (Next-Batch Candidate Tables)
- `proposed_candidates_round01.csv` ... `proposed_candidates_roundXX.csv`
- `proposed_candidates_next_batch.csv` *(assay ingestion mode only)*

Each row contains:
- peptide `sequence`
- predicted outcomes (`pred_affinity`, `pred_stability`, `pred_solubility`)
- uncertainty (`sd_affinity`, `sd_stability`, `sd_solubility`)
- liability gate metrics and flags
- acquisition/ranking score (`acq_score`)

#### C) CRO-Ready Operational Outputs
**Synthesis order sheets**
- `roundXX_synthesis_order_sheet.csv`
- `next_batch_synthesis_order_sheet.csv`

**Plate maps**
- `roundXX_plate_map_96well.csv`
- `roundXX_plate_map_96well.png`
- `next_batch_plate_map_96well.csv`
- `next_batch_plate_map_96well.png`

#### D) Decision Reports
- `screening_report_top50.csv`

This is the final ranked list of the top candidates using:
- multi-objective composite score
- Pareto-front detection
- liability annotations

#### E) Learning/Progress Tracking
- `learning_history.csv`
- `fig_learning_curve.png`
- `run_summary.json`

#### F) Assay QC and Summaries (Real Assay Mode Only)
- `assay_primary_summary.csv`
- `assay_plate_qc.csv`
- `assay_stability_summary.csv` *(optional)*
- `assay_solubility_summary.csv` *(optional)*

## Data Collection Process
### Simulation Mode
Measured values are generated by a synthetic “oracle” function that approximates assay-like behaviour with noise. This mode exists only to validate the workflow logic end-to-end.

### Real Assay Ingestion Mode
Measured values are derived from real experimental CSV files:
- Primary assay can be:
  - **Dose–response** (IC50 and pIC50 derived via 4PL fitting), or
  - **Single-point screening** (robust trimmed mean of responses)
- Replicate consistency is checked using **CV (%)**
- Plate quality is estimated using **Z’ factor** if controls are provided

## Preprocessing and Cleaning
### Sequence Handling
- All peptide sequences are stored as uppercase single-letter amino-acid strings.
- Invalid characters are not permitted and should be removed/cleaned upstream.

### Feature Generation (inside the pipeline)
Features are derived from sequences using:
- amino acid composition
- physicochemical properties (hydropathy, charge, aromaticity, etc.)
- motif counts
- optional 2-mer k-mer frequencies

No external embeddings are required.

## Recommended Uses
### Suitable Use Cases
- Demonstrating an **active learning** loop for peptide optimisation  
- Benchmarking multi-objective selection strategies (UCB, EI, EHVI-lite)  
- Producing **synthesis and screening-ready batches**  
- Identifying Pareto-optimal trade-offs between potency and developability  

### Not Suitable Use Cases
- Predicting clinical outcomes  
- Making claims of biological mechanism without validation  
- Toxicity, immunogenicity, PK/PD prediction (not included)  
- Generalising across unrelated peptide families without retraining  

## Dataset Splits and Leakage Risks
This is an iterative dataset where proposals depend on prior measured rounds. Standard random splitting can be misleading.

### Best practice evaluation
- Evaluate by **time/round** (train on earlier rounds, test on later rounds).
- Report performance under:
  - in-round prediction
  - next-round generalisation
- Use diversity constraints to reduce redundancy and selection bias

## Data Fields (Key Columns)
### Common Columns
| Column | Meaning |
|---|---|
| `sequence` | Amino-acid sequence of the peptide |
| `round` | Iteration index (0 = initial measurement set) |

### Measured Table Columns
| Column | Meaning |
|---|---|
| `affinity` | Measured binding/functional score (or pIC50 proxy in assay mode) |
| `stability` | Measured stability score (scalar assay or default=1.0 if missing) |
| `solubility` | Measured solubility score (scalar assay or default=1.0 if missing) |

### Proposed Candidate Columns
| Column | Meaning |
|---|---|
| `pred_affinity` | Predicted affinity |
| `pred_stability` | Predicted stability |
| `pred_solubility` | Predicted solubility |
| `sd_affinity` | Ensemble uncertainty (std) for affinity |
| `sd_stability` | Ensemble uncertainty (std) for stability |
| `sd_solubility` | Ensemble uncertainty (std) for solubility |
| `liability_penalty` | Soft penalty based on developability flags |
| `acq_score` | Final acquisition score used for ranking |

### Liability Flag Columns
| Column | Meaning |
|---|---|
| `flag_deamidation_hotspot` | NG/NS/QG/QN present |
| `flag_asp_pro_cleavage` | DP present |
| `flag_oxidation_risk` | high Met/Trp burden |
| `flag_cysteine_risk` | excessive cysteine |
| `flag_extreme_charge` | charge too high/low |
| `flag_aggregation_risk` | hydrophobic/aromatic heavy |
| `liability_count` | total flags triggered |

### Assay QC Columns (Real Assay Mode)
| Column | Meaning |
|---|---|
| `replicate_cv_median_percent` | median CV across concentrations |
| `flag_high_cv` | replicate CV exceeds threshold |
| `zprime` | Z’ factor per plate if controls exist |
| `IC50_uM` | fitted IC50 (dose-response only) |
| `pIC50_uM` | derived potency metric (dose-response only) |
| `fit_rmse` | goodness-of-fit for 4PL fitting |

## Ethical and Safety Considerations
- This dataset contains **no human or patient data**.
- Peptides are synthetic candidates and should be treated as experimental designs, not therapeutics.
- Liability scoring is heuristic, it reduces risk but does not guarantee manufacturability or safety.

## Known Biases and Limitations
- Early rounds can bias the model toward motif-rich sequences, potentially missing novel chemotypes.
- Assay noise may cause unstable ranking, especially if replicate CV is high.
- The surrogate models are sequence-based and do not use structural docking or MD simulation.

## Maintenance and Versioning
- Dataset outputs are generated per run and stored in the run output directory.
- `run_summary.json` captures:
  - mode (simulation vs assay ingestion)
  - best observed peptides
  - key parameters (acq choice, constraints)
  - artifact filenames for auditability

## Cite this work
If you use this code, figures, or ideas, please cite:

**Petalcorin, M. I. R.** (2026). *Unified Proof-of-Concept Peptide Design Platform (Simulation + Assay-Ingestion + CRO Artifacts)*. GitHub repository.

## How to Reproduce the Dataset
### Simulation Mode
```bash
python peptide_platform_unified.py --outdir out_iso_peptides --seed 7 --rounds 8 --acq UCB
