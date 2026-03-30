# Uncertainty-Aware LLM Framework for Pancreatic Cancer Prediction

This repository contains the experimental outputs, prompt templates,
and supplementary materials accompanying the paper:

> **A Systems Engineering Framework for Uncertainty-Aware LLMs in
> Medical Prediction**
>
> Panagiotis Tirchas, Nicholas Christakis, Dimitris Drikakis
>
> Institute for Advanced Modelling and Simulation, University of
> Nicosia, Nicosia CY-2417, Cyprus
>
> Submitted to *Expert Systems with Applications* (Elsevier)

---

## Overview

This study introduces a systems engineering framework for
uncertainty-aware medical prediction using large language models
(LLMs). The LLM is treated as a modular, replaceable inference
component within an auditable expert-system pipeline. The framework
converts structured patient biomarker data into standardised prompts,
optionally augments inference with retrieved clinical cases via
retrieval-augmented generation (RAG), and quantifies uncertainty along
three complementary dimensions: aleatoric, epistemic, and semantic.

**Important:** The LLM used in all experiments (Azure-hosted GPT-5.2)
was not fine-tuned or retrained on any portion of the pancreatic
cancer dataset. The 146-patient partition described below serves
exclusively as a retrieval lookup table for the RAG module, not as
training data.

---

## Repository Structure
```
.
├── LICENCE                           # Licence
├── README.md                         # Project description
├── results_with_context.xlsx         # Results using context
└── results_without_context.xlsx      # Results without context
```
---

## Experimental Conditions

The experiments evaluate two conditions across 63 hold-out test
patients. Each condition generates 10 independent LLM outputs per
patient: 5 from structurally varied prompt templates and 5 from
repeated stochastic runs using a fixed template.

### With Context (Retrieval-Augmented)

**File:** `results_with_context.xlsx`

The RAG module is active. For each patient, the system retrieves
similar cases from a database of 146 previously characterised patients
(the 70% partition of the full 209-patient cohort). The retrieved
cases, including their confirmed diagnoses, are formatted as text and
appended to the prompt. The LLM receives three components:

1. The instruction prompt (one of five templates)
2. The patient's quantitative biomarker record
3. A summary of retrieved similar cases and their outcomes

### Without Context (Baseline)

**File:** `results_without_context.xlsx`

The RAG module is disabled. The LLM receives only two components:

1. The instruction prompt (one of five templates)
2. The patient's quantitative biomarker record

No retrieved cases or external evidence are provided. The model must
classify based entirely on its pre-trained medical knowledge.

---

## Spreadsheet Structure

Both `.xlsx` files share an identical sheet structure. Each file
contains 12 sheets organised into four groups.

### Group 1: Ground Truth

| Sheet | Contents |
|-------|----------|
| **Actual** | The 63 hold-out test patients with their confirmed diagnoses. Columns: patient identifier and ground-truth binary label (0 = benign or control; 1 = PDAC). This sheet serves as the reference standard for computing all accuracy and uncertainty metrics. |

### Group 2: Aggregated Results

| Sheet | Contents |
|-------|----------|
| **Results** | Aggregated predictions across all runs for each patient. This sheet contains two tables. The first table (**Prompt Variations**) aggregates results from the five structurally varied prompt templates (Prompts 1 through 5). The second table (**Same Prompt**) aggregates results from the five stochastic runs using Prompt 1 as a fixed template (Classify 1 through Classify 5). For each patient, the sheet reports the actual diagnosis, the binary prediction from each run, row accuracy (fraction of runs matching ground truth), and consensus rate (fraction of runs agreeing with the majority-vote label). |

#### Metrics Defined in the Results Sheet

| Metric | Definition |
|--------|------------|
| **Row accuracy** | The percentage of the five predictions for a given patient that match the ground-truth diagnosis. A value of 100% means all five runs were correct; 0% means all five were incorrect. |
| **Consensus rate** | The percentage of the five predictions for a given patient that agree with the majority-vote label (regardless of whether the majority vote is correct). A value of 100% means all five runs produced the same label; lower values indicate label disagreement across runs. |

### Group 3: Individual Prompt Variant Outputs (Aleatoric)

These five sheets record the output when each of the five different
instruction templates was sent to the LLM. The patient biomarker data
are identical
 across all five; only the instruction wording varies.
These outputs are used to compute **aleatoric uncertainty** (sensitivity
to prompt phrasing).

| Sheet | Prompt Template Used |
|-------|---------------------|
| **Prompt 1** | Template 1 (fixed template, also used in the Same Prompt condition) |
| **Prompt 2** | Template 2 |
| **Prompt 3** | Template 3 |
| **Prompt 4** | Template 4 |
| **Prompt 5** | Template 5 |

#### Column Schema

| Column | Type | Description |
|--------|------|-------------|
| Patient | int | Patient identifier (1 through 63) |
| Cancer Prediction | int (0 or 1) | Binary diagnostic classification |
| Confidence | int (0 to 100) | Model-reported confidence score |
| Reasoning | string | Free-text clinical reasoning statement |

### Group 4: Stochastic Repeat Outputs (Epistemic)

These five sheets record the output when the same instruction template
(Prompt 1) was sent to the LLM five separate times. Each call uses a
fresh stochastic draw (temperature ~0.5). The instruction and patient
data are identical across all five; only the random seed differs.
These outputs are used to compute **epistemic uncertainty** (the model's
internal generation variability).

| Sheet | Description |
|-------|-------------|
| **Classify 1** | First stochastic run with Prompt 1 |
| **Classify 2** | Second stochastic run with Prompt 1 |
| **Classify 3** | Third stochastic run with Prompt 1 |
| **Classify 4** | Fourth stochastic run with Prompt 1 |
| **Classify 5** | Fifth stochastic run with Prompt 1 |

The column schema is identical to the Prompt sheets (Patient, Cancer
Prediction, Confidence, Reasoning).



### Group 5: Appendix

| Sheet | Contents |
|-------|----------|
| **Appendix** | Reference material including the five prompt templates and definitions of the metrics used in the Results sheet (row accuracy and consensus rate). |


---

## Dataset

The pancreatic cancer biomarker dataset originates from the following
public clinical study:

> Debernardi S, O'Brien H, Algahmdi AS, et al. A combination of
> urinary biomarker panel and PanRisk score for earlier detection of
> pancreatic cancer: A case-control study. *PLOS Medicine*. 2020;
> 17(12):e1003489.
> https://doi.org/10.1371/journal.pmed.1003489

The original study evaluated 590 urine samples. After complete-case
analysis (removing records with missing biomarker measurements), 209
patient records were retained. These were partitioned as follows:

| Partition | n | Purpose |
|-----------|---|---------|
| Retrieval database | 146 (70%) | Populates the RAG module lookup table |
| Hold-out test set | 63 (30%) | Independent evaluation (results in this repository) |

The LLM was not trained on either partition.

---

## Experimental Configuration

| Parameter | Value |
|-----------|-------|
| Model | Azure-hosted GPT-5.2 (chat-completion API) |
| Temperature | ~0.5 |
| Top-
p | 1.0 (default) |
| Semantic embedding model | text-embedding-3-large |
| Cosine similarity threshold | 0.75 |
| Retrieval database size | 146 patients |
| Test set size | 63 patients |
| Outputs per patient per condition | 10 (5 prompt variants + 5 stochastic repeats) |

---

## Prompt Templates

The five instruction templates are reproduced in Appendix A of the
paper. 

All five templates share identical clinical task definitions (binary
PDAC classification), output format constraints (CSV with patient_id,
cancer_prediction, confidence, reasoning), and decisional framing
("Be decisive. Do NOT default to the safer-sounding label"). They
differ only in surface-level instruction wording.

---

## Reproducing the Paper Results

All metrics reported in the paper (accuracy, precision, recall,
confusion matrices, aleatoric entropy, epistemic entropy, semantic
entropy, AUROC, and risk-coverage curves) can be recomputed from the
prediction and reasoning columns in these spreadsheets combined with
the ground-truth labels in the Actual sheet.

1. **Accuracy, precision, recall, confusion matrices:** Compare the
   Cancer Prediction column in each sheet against the Actual sheet.

2. **Aleatoric entropy and variation ratio:** For each patient,
   collect the five binary predictions from Prompt 1 through
   Prompt 5. Compute the empirical positive-class probability and
   apply binary Shannon entropy (Equation 2 in the paper).

3. **Epistemic entropy and variation ratio:** For each patient,
   collect the five binary predictions from Classify 1 through
   Classify 5. Apply the same entropy formula (Equation 4).

4. **Semantic entropy:** Embed the Reasoning texts using
   text-embedding-3-large. Cluster by cosine similarity (threshold
   0.75). Compute entropy over cluster proportions (Equation 8).

5. **AUROC and risk-coverage:** Use the entropy scores to rank
   patients and compute discrimination and selective-classification
   metrics.

---

## Citation

If you use these materials, please cite:

[Full citation to be added upon publication]



---

## License

**GNU General Public Licence v3.0**
