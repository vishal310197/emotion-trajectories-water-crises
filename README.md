# Emotion Trajectories in Online Discourse During Drinking Water Contamination Crises

Reproducibility materials for the manuscript:

**Emotion Trajectories in Online Discourse During Drinking Water Contamination Crises: A Comparative Social-Sensing Study**

Authors: **Vishal Kumar, Frank Hopfgartner, Mourad Oussalah**

This repository contains the code, documentation, and manuscript-supporting
materials for a comparative social-sensing study of emotional expressions in
online discourse during the Nokia, Flint, and Iqaluit drinking-water
contamination crises.

The study treats online discourse as a **partial social-sensing trace**, not as
a representative measure of population-level emotion, trust, compliance,
governance quality, or psychological state.

---

## Repository contents

Recommended public repository structure:

```text
emotion-trajectories-water-crises/
│
├── README.md
├── LICENSE
├── CITATION.cff
├── requirements.txt
├── DATA_DICTIONARY.md
├── REPRODUCIBILITY.md
│
├── notebooks/
│   ├── 01_standard_finetuning.ipynb
│   ├── 02_lora_comparison.ipynb
│   └── 03_temporal_emotion_analysis.ipynb
│
├── manuscript/
│   ├── jem_manuscript.tex
│   ├── supplement.tex
│   └── references.bib
│
├── figures/
│   └── [final manuscript figures]
│
└── data/
    └── README.md
```

Large processed datasets and model files are best archived in Zenodo rather
than duplicated directly in GitHub.

---

## Archived materials

### Code and notebooks

Zenodo DOI:

**10.5281/zenodo.18480875**

### Processed data

Zenodo DOI:

**10.5281/zenodo.17748814**

The Zenodo records should contain the frozen version associated with the
journal submission. GitHub may contain the readable/current repository, while
Zenodo provides the persistent archived version.

---

## Main reproducibility notebooks

### `01_standard_finetuning.ipynb`

Renamed from:

`Fine_tuning_Model-new.ipynb`

This notebook contains the reported standard DistilRoBERTa fine-tuning
workflow and the final standard-model evaluation.

Base model:

```text
michellejieli/emotion_text_classifier
```

Reported training settings:

| Parameter | Value |
|---|---|
| evaluation strategy | epoch |
| save strategy | epoch |
| training batch size | 16 |
| evaluation batch size | 16 |
| epochs | 3 |
| weight decay | 0.01 |
| load best model at end | True |

Reported held-out evaluation performance:

- Accuracy: **0.692**
- Macro F1: **0.459**
- Weighted F1: **0.674**

---

### `02_lora_comparison.ipynb`

Renamed from:

`2Optimized_Fine_Tuning-Copy1.ipynb`

This notebook contains the reported LoRA comparison.

Reported LoRA configuration:

| Parameter | Value |
|---|---|
| `r` | 8 |
| `lora_alpha` | 32 |
| `lora_dropout` | 0.1 |
| target modules | `query`, `value` |
| bias | `none` |
| task type | `SEQ_CLS` |
| training batch size | 16 |
| evaluation batch size | 16 |
| epochs | 3 |
| weight decay | 0.01 |

Reported held-out evaluation performance:

- Accuracy: **0.075**
- Macro F1: **0.101**
- Weighted F1: **0.060**

A later exploratory parameter block using five epochs and batch size 8 is not
part of the reported manuscript workflow and should be removed or clearly
marked as exploratory in the public notebook.

---

### `03_temporal_emotion_analysis.ipynb`

Cleaned from the final downstream analysis notebook.

This notebook contains:

- final emotion-composition summaries;
- weekly Nokia aggregation;
- yearly Flint aggregation;
- monthly Iqaluit aggregation;
- descriptive EmoEle calculation;
- centered three-bin smoothing;
- YAKE keyword extraction;
- figures/tables used in the manuscript.

Legacy KeyBERT and other exploratory analyses are not part of the reported
final workflow and should be removed or clearly marked as exploratory.

---

## Reference-data design

The retained online-discourse corpus contains:

| Case | Retained records |
|---|---:|
| Nokia | 4,461 |
| Flint | 22,724 |
| Iqaluit | 2,052 |
| **Total** | **29,237** |

Approximately 30% of the full corpus was expert-reviewed for reference
annotation.

### Model-development pool

The final development pool contains:

**5,836 records**

It was divided internally into:

- **4,668** internal training records;
- **1,168** internal validation records.

The validation subset was used for epoch-wise evaluation/checkpoint
selection.

### Held-out evaluation

The separate evaluation pool contains 2,980 records. One record carried a
label outside the predefined seven-category emotion scheme and was excluded
from the reported classifier scoring.

Final reported evaluation size:

**n = 2,979**

Seven evaluation classes:

- anger
- disgust
- fear
- joy
- neutral
- sadness
- surprise

See [`REPRODUCIBILITY.md`](REPRODUCIBILITY.md) for full details.

---

## Final case-level prediction files

The selected standard fine-tuned model was used for the downstream temporal
analysis.

Recommended public filenames:

| Case | Public filename | Classified records |
|---|---|---:|
| Nokia | `nokia_emotion_predictions.csv` | 4,421 |
| Flint | `flint_emotion_predictions.csv` | 22,724 |
| Iqaluit | `iqaluit_emotion_predictions.csv` | 1,977 |

Expected final label counts are documented in
[`REPRODUCIBILITY.md`](REPRODUCIBILITY.md).

The difference between retained and classified record counts reflects
exclusion of records without usable text for emotion classification.

---

## EmoEle

The descriptive Emotional Elevation summary is:

```text
EmoEle =
(joy + neutral)
-
(anger + fear + disgust + sadness + surprise)
```

The case-specific EmoEle series is:

1. calculated from within-bin emotion proportions;
2. min-max normalized to `[0, 1]`;
3. smoothed with a centered three-bin rolling mean;
4. completed at rolling-window boundaries using backward and forward filling.

`neutral` is grouped with `joy` only to represent movement away from
negatively dominated discourse. It is **not** interpreted as positive affect,
trust, optimism, satisfaction, social cohesion, or recovery of trust.

EmoEle is a descriptive summary and is not a validated operational indicator.

---

## Keyword extraction

The reported manuscript uses **YAKE** for qualitative lexical context.

Reported settings:

- language: English;
- maximum n-gram size: 2;
- top keywords: 20.

YAKE outputs are not used as independent validation of the emotion
classifier.

---

## Installation

Python dependencies are listed in:

```text
requirements.txt
```

Install with:

```bash
python -m pip install -r requirements.txt
```

The historical final notebooks were run under slightly different
Transformers versions, so `requirements.txt` permits the narrow range
covering both archived environments.

For exact historical replication, use the archived final model checkpoint
where available.

---

## Recommended execution order

Run the notebooks in this order:

```text
1. notebooks/01_standard_finetuning.ipynb
2. notebooks/02_lora_comparison.ipynb
3. notebooks/03_temporal_emotion_analysis.ipynb
```

For reproducing only the reported temporal figures and tables, start from the
archived final prediction CSVs rather than retraining the classifier.

---

## Important reproducibility note

The original internal 80/20 model-development split was created without an
explicitly recorded random seed in the archived notebook.

Therefore, a fresh rerun from the pooled development CSV may not recreate the
identical training/validation row membership or exact trained checkpoint.

For exact reproduction of the manuscript outputs, use:

- the archived final prediction CSVs; and
- the archived final trained standard-model checkpoint, if available.

Do not add a new random seed and treat that rerun as the historical reported
experiment.

---

## Data documentation

See:

- [`DATA_DICTIONARY.md`](DATA_DICTIONARY.md) for file and variable definitions;
- [`REPRODUCIBILITY.md`](REPRODUCIBILITY.md) for the complete reported workflow.

Before public release, processed datasets should be cleaned of unnecessary:

- `Unnamed:*` index columns;
- usernames/authors;
- direct user/profile URLs;
- other personal metadata not required for reproducibility.

Raw or processed platform text should only be redistributed where permitted
by the relevant source/platform access, licensing, copyright, and
redistribution conditions.

---

## Manuscript sources

The final repository may include:

```text
manuscript/jem_manuscript.tex
manuscript/supplement.tex
manuscript/references.bib
```

along with the final figures referenced by the LaTeX source.

---

## Citation

If you use the code or notebooks, please cite the archived software record:

**Kumar, V., Hopfgartner, F., & Oussalah, M. (2026). Emotion trajectories in
online discourse during drinking water contamination crises: analysis code.
Zenodo. https://doi.org/10.5281/zenodo.18480875**

Machine-readable citation metadata are provided in:

```text
CITATION.cff
```

For the processed dataset, cite the corresponding Zenodo data record:

**10.5281/zenodo.17748814**

---

## License

Repository code is released under the **MIT License** unless otherwise stated.

See:

[`LICENSE`](LICENSE)

The repository license applies to the software/code authored for this project.
It does **not** automatically grant redistribution rights for third-party
forum/social-media content, pretrained models, or other externally sourced
materials.

---

## Contact

For questions about the manuscript, data, or reproducibility materials,
please use the corresponding-author contact information provided in the
published article or repository metadata.
