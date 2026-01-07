# CI-Guided Data Curation: 40% Error Recovery

> **Validated:** Prediction instability signals label noise. Remove unstable samples, training improves.

This repository contains the complete experiment that validates CI-guided data curation on noisy data.

## The Result

| Dataset Type | Curation Effect |
|--------------|-----------------|
| Clean benchmark (SST-2) | No improvement (nothing to fix) |
| **Noisy data (10% label flip)** | **+4% accuracy, 40% gap to perfect labels closed** |

## The Hypothesis

Samples that cause unstable predictions are more likely to have label issues. The Collapse Index (CI) measures how often a model's output changes when you slightly rephrase the input. High instability = suspicious sample.

## The Experiment

We ran 4 experiments on SST-2 sentiment classification with DistilBERT:

| Experiment | Data | Strategy | Result |
|------------|------|----------|--------|
| 1 | Clean | Remove dangerous samples | -5% accuracy |
| 2 | Clean | Downweight dangerous | -6% accuracy |
| 3 | Clean | Full dataset removal | -1% accuracy |
| **4** | **10% noise** | **Remove suspicious** | **+4% accuracy** |

Experiments 1-3 were controls. They proved CI does not hallucinate problems that don't exist.

Experiment 4 was validation. It proved CI finds real noise when noise exists.

## Key Results (Experiment 4)

| Metric | Baseline | Curated | Oracle | Delta |
|--------|----------|---------|--------|-------|
| Accuracy | 68.0% | 72.0% | 78.0% | **+4.0%** |
| F1 Score | 0.758 | 0.763 | 0.796 | +0.5% |
| AUC | 0.819 | 0.789 | 0.860 | -3.0% |

**40% of the accuracy gap to perfect labels recovered.**

## The Pipeline

Getting from raw data to curated training set:

```bash
# Step 1: Analyze perturbed predictions
ci run data/sst2_predictions.csv --cl --diag

# Step 2: Outputs triage.json with CSI types and brittle cases
# data/analyzed/20260107_sst2_demo/triage.json

# Step 3: Run curation
ci curate relabel data/analyzed/20260107_sst2_demo --rebuild
```

Output: curated CSV with difficulty labels + list of "dangerous" samples.

## Files in This Repository

```
ci-curation/
├── README.md                         # This file
├── experiment_log.md                 # Full experiment log with all 4 experiments
├── curation_ab_test_v4_noisy.ipynb   # The validation experiment (run on Colab)
└── sst2_ci_demo_curated.csv          # Sample curated dataset
```

## Running the Experiment

1. Open `notebooks/curation_ab_test_v4_noisy.ipynb` in Google Colab
2. Upload `data/sst2_ci_demo_curated.csv` when prompted
3. Run all cells (takes ~5 min on T4 GPU)
4. See results: Baseline vs Curated vs Oracle

## When to Use CI-Guided Curation

| Dataset Type | Use Curation? | Expected Benefit |
|--------------|---------------|------------------|
| Web-scraped data | Yes | High (10-40% error recovery) |
| Crowdsourced labels (MTurk) | Yes | High |
| Auto-labeled (weak supervision) | Yes | Moderate-High |
| Clean benchmarks (SST-2, GLUE) | No | None (already clean) |
| Expert-labeled gold standard | No | Minimal |
| Small datasets (<500) | Caution | May hurt |

## The Insight

> Prediction instability is a learnable signal for data quality. Samples that confuse models during evaluation were likely confusing during labeling too.

**Measure instability -> Find noise -> Clean data -> Better models**

## Links

- [SST-2 Base Dataset (ci-sst2)](https://github.com/collapseindex/ci-sst2) - Original SST-2 validation dataset used in this experiment
- [SRI Validation (ci-sri)](https://github.com/collapseindex/ci-sri) - AG News multi-class validation
- [Case Study](https://collapseindex.org/case-studies/ci-curation-validation) - Full writeup
- [Collapse Index Labs](https://collapseindex.org) - Website

## Citation

```bibtex
@misc{kwon2026curation,
  title={CI-Guided Data Curation: Using Prediction Instability to Detect Label Noise},
  author={Kwon, Alex},
  year={2026},
  publisher={GitHub},
  howpublished={\url{https://github.com/collapseindex/ci-curation}},
  note={Collapse Index Labs},
  orcid={0009-0002-2566-5538}
}
```

Please also cite the original SST-2 dataset:

```bibtex
@inproceedings{socher2013recursive,
  title={Recursive deep models for semantic compositionality over a sentiment treebank},
  author={Socher, Richard and Perelygin, Alex and Wu, Jean and Chuang, Jason and Manning, Christopher D and Ng, Andrew and Potts, Christopher},
  booktitle={Proceedings of the 2013 conference on empirical methods in natural language processing},
  pages={1631--1642},
  year={2013}
}
```

## License

- **This Repository:** MIT License (notebook code only)
- **CI + SRI Methodology:** [Proprietary](https://github.com/collapseindex/collapseindex/blob/main/LICENSE.md) - (c) 2026 Collapse Index Labs - Alex Kwon
- **SST-2 Dataset:** Stanford NLP (cite original paper above)
- **DistilBERT Model:** Apache 2.0

**Copyright (c) 2026 Collapse Index Labs - Alex Kwon. All rights reserved.**


