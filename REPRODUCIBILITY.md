# Reproducibility Guide

## Project

**Emotion Trajectories in Online Discourse During Drinking Water Contamination Crises: A Comparative Social-Sensing Study**

This document describes the computational workflow corresponding to the
reported manuscript results. It intentionally distinguishes the final
reported workflow from exploratory and superseded notebook runs.

---

## 1. Final reproducibility files

Use these files for the reported analysis:

### Code

1. `01_standard_finetuning.ipynb`
   - renamed from `Fine_tuning_Model-new.ipynb`
   - reported standard DistilRoBERTa fine-tuning
   - pretrained and fine-tuned evaluation
   - generation of the final case-level standard-model predictions

2. `02_lora_comparison.ipynb`
   - renamed from `2Optimized_Fine_Tuning-Copy1.ipynb`
   - reported LoRA configuration and evaluation

3. `03_temporal_emotion_analysis.ipynb`
   - cleaned from `emotionspaperlatest2 (1).ipynb`
   - emotion composition
   - temporal aggregation
   - EmoEle
   - YAKE keyword extraction
   - reported plots/tables

### Data

- `reference_development.csv`
- `heldout_evaluation.csv`
- `standard_evaluation_predictions.csv` if the archived `eval_data.csv` is available
- `lora_evaluation_predictions.csv`
- `nokia_emotion_predictions.csv`
- `flint_emotion_predictions.csv`
- `iqaluit_emotion_predictions.csv`

---

## 2. Reference-data design

The retained online-discourse corpus contains:

- Nokia: 4,461 records
- Flint: 22,724 records
- Iqaluit: 2,052 records
- Total: 29,237 records

Approximately 30% of the full corpus was expert-reviewed for reference
annotation.

### Model-development pool

`reference_development.csv` contains **5,836** valid seven-class reference
records, approximately 20% of the full corpus.

The reported standard fine-tuning notebook performs an internal 80/20 split:

- internal training: **4,668**
- internal validation: **1,168**

The internal validation subset is used for epoch-wise evaluation/checkpoint
selection.

### Held-out evaluation pool

`heldout_evaluation.csv` contains **2,980** records. One record has a label
outside the predefined seven-category scheme and is excluded from classifier
scoring.

Final held-out evaluation size:

**n = 2,979**

This separate evaluation set is used for the reported comparison among:

- pretrained classifier
- standard fine-tuned classifier
- LoRA-adapted classifier

---

## 3. Environment setup

Install dependencies with:

```bash
python -m pip install -r requirements.txt
```

The archived notebooks record slightly different Transformers versions for
the final standard and LoRA runs:

- standard fine-tuning environment: Transformers 4.57.0
- reported LoRA environment: Transformers 4.56.2

The supplied `requirements.txt` therefore allows the narrow range
`>=4.56.2,<4.58`.

The notebooks recorded Torch 2.8.0, Accelerate 1.10.1, and the reported LoRA
environment used PEFT 0.17.1.

---

## 4. Standard fine-tuning

Open:

`notebooks/01_standard_finetuning.ipynb`

The reported training settings are:

| Setting | Value |
|---|---|
| evaluation strategy | epoch |
| save strategy | epoch |
| train batch size | 16 |
| evaluation batch size | 16 |
| epochs | 3 |
| weight decay | 0.01 |
| load best model at end | True |

Base model:

`michellejieli/emotion_text_classifier`

The notebook tokenizes the `body` field from the model-development data.

### Important split caveat

The historical `train_test_split(test_size=0.2)` call did **not** explicitly
record a random seed. Therefore, retraining from the pooled development CSV
may not recreate the identical 4,668/1,168 row membership or the exact
checkpoint.

For exact reproduction of the published predictions, preserve and archive
the final trained checkpoint directory (`emotion-model/`) if it is still
available. The archived final prediction CSVs also preserve the outputs used
in the manuscript.

Do not add a new seed and present the resulting rerun as if it were the
historical reported experiment.

---

## 5. Reported standard-model evaluation

The separate seven-class held-out evaluation set has the following support:

| Emotion | n |
|---|---:|
| anger | 1,552 |
| disgust | 224 |
| fear | 67 |
| joy | 10 |
| neutral | 785 |
| sadness | 143 |
| surprise | 198 |
| Total | 2,979 |

Expected pretrained performance:

- Accuracy: **0.359**
- Macro F1: **0.243**
- Weighted F1: **0.305**

Expected standard fine-tuned performance:

- Accuracy: **0.692**
- Macro F1: **0.459**
- Weighted F1: **0.674**

The standard fine-tuned classifier is the model used for the downstream
case-level emotion trajectories.

---

## 6. Reported LoRA comparison

Open:

`notebooks/02_lora_comparison.ipynb`

The reported LoRA configuration is:

| Setting | Value |
|---|---|
| rank `r` | 8 |
| `lora_alpha` | 32 |
| target modules | `query`, `value` |
| `lora_dropout` | 0.1 |
| bias | `none` |
| task type | `SEQ_CLS` |
| train batch size | 16 |
| evaluation batch size | 16 |
| epochs | 3 |
| weight decay | 0.01 |

Expected held-out evaluation performance:

- Accuracy: **0.075**
- Macro F1: **0.101**
- Weighted F1: **0.060**

### Exploratory cell warning

The historical LoRA notebook also contains a later exploratory training
argument block using five epochs, batch size 8, learning rate `2e-5`, and
weight decay 0.05. That block does **not** correspond to the reported
three-epoch LoRA result and should be removed or clearly marked as
exploratory in the public cleaned notebook.

---

## 7. Full-case prediction files used downstream

The selected standard fine-tuned classifier generated the following final
outputs:

### Nokia

`nokia_emotion_predictions.csv`

Expected classified records: **4,421**

Expected label counts:

| Emotion | Count |
|---|---:|
| disgust | 1,421 |
| neutral | 1,282 |
| anger | 1,262 |
| sadness | 194 |
| fear | 140 |
| surprise | 116 |
| joy | 6 |

### Flint

`flint_emotion_predictions.csv`

Expected classified records: **22,724**

Expected label counts:

| Emotion | Count |
|---|---:|
| anger | 14,615 |
| neutral | 4,933 |
| disgust | 1,402 |
| surprise | 1,049 |
| sadness | 540 |
| fear | 140 |
| joy | 45 |

### Iqaluit

`iqaluit_emotion_predictions.csv`

Expected classified records: **1,977**

Expected label counts:

| Emotion | Count |
|---|---:|
| anger | 828 |
| neutral | 740 |
| disgust | 182 |
| surprise | 129 |
| sadness | 69 |
| fear | 19 |
| joy | 10 |

These counts can be used as a simple integrity check that the correct final
prediction files are being used.

---

## 8. Temporal analysis

Open:

`notebooks/03_temporal_emotion_analysis.ipynb`

Use:

- weekly aggregation for Nokia
- yearly aggregation for Flint
- monthly aggregation for Iqaluit

For each temporal bin:

1. count the seven predicted emotion categories;
2. insert zero counts for any absent category;
3. convert counts to within-bin proportions;
4. calculate descriptive EmoEle;
5. apply case-specific min-max normalization to `[0, 1]`;
6. apply a centered three-bin rolling mean;
7. complete rolling-window boundary values using backward and forward filling;
8. plot the resulting trajectories.

### EmoEle

`EmoEle = (joy + neutral) - (anger + fear + disgust + sadness + surprise)`

Neutral is grouped with joy only to represent movement away from negatively
dominated discourse. It is **not** interpreted as inherently positive
affect, trust, optimism, satisfaction, or social cohesion.

---

## 9. Keyword extraction

The reported manuscript uses **YAKE**, with:

- language: English
- maximum n-gram size: 2
- number of keywords: 20

YAKE outputs are qualitative lexical context only and are not an independent
validation of emotion-classifier performance.

KeyBERT code found in exploratory analysis notebooks was not used for the
reported keyword results. Remove it from the cleaned final notebook or mark
it explicitly as exploratory.

---

## 10. Files not part of the final reported workflow

Do not use the following as the primary reproducibility files:

- `Fine_tuning_Model (1).ipynb`
- `Fine_tuning_Model (2).ipynb`
- `Optimized_Fine_Tuning.ipynb`
- `emotionrq2.ipynb`
- `researchque1 (1).ipynb`
- duplicate/older `emotionspaper*.ipynb`
- `filteredemotion_dataset.csv`
- `filteredemotion_dataset (1).csv`
- `predictedemotion.csv`
- older `predicted_output.csv`
- older `flint_predicted_output.csv`
- older `canada_predicted_output.csv`
- `opt_eval_data.csv`
- case-level `opt_*` prediction files

They represent superseded, exploratory, or non-reported runs.

---

## 11. Data-sharing and privacy

The public reproducibility package should remove unnecessary:

- `Unnamed:*` index columns
- author/user names
- direct user/profile URLs
- other personal metadata not needed for reproduction

Raw or processed platform text should only be redistributed when permitted
by the relevant source/platform terms, licensing conditions, copyright
requirements, and ethical constraints.

The MIT license in this repository applies to repository code unless
otherwise stated; it does not automatically grant rights to third-party
social-media/forum content.

---

## 12. Archival recommendation

For the submission-associated release:

1. tag the cleaned GitHub repository, e.g. `v1.0.0-jem-submission`;
2. archive that release in the code Zenodo record;
3. deposit the cleaned processed data in the data Zenodo record;
4. if available, archive the exact final standard checkpoint and reported
   LoRA adapter/checkpoint in Zenodo;
5. cite both persistent Zenodo DOIs in the manuscript data-and-code statement.

Current project archive identifiers:

- Code: `10.5281/zenodo.18480875`
- Processed data: `10.5281/zenodo.17748814`
