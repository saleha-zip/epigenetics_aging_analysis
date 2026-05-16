# Epigenetics Aging Analysis

**Course:** Special Topics in Bioinformatics — Assignment 7
**Author:** Saleha

This repository covers two complementary approaches to studying DNA methylation: whole-genome bisulfite sequencing (WGBS) for base-resolution methylome mapping, and EPIC array-based epigenetic clock benchmarking for biological age estimation. Together they span the two major experimental paradigms in the methylation field — sequencing-based and array-based — and demonstrate how the same underlying biology (CpG methylation) is captured at different resolutions, costs, and scales.

---

## Repository Structure

```
epigenetics_aging_analysis/
├── 01_wgbs/                          # WGBS conceptual component
│   └── README.md                     # Theory, pipeline, and paper summary
├── 02_epic_array_analysis/           # EPIC array clock benchmarking
│   ├── aging_clock_benchmarking.ipynb
│   ├── results/
│   │   ├── eda_age_distributions.png
│   │   ├── eda_sex_breakdown.png
│   │   ├── correlation_matrix_GSE120307.png
│   │   ├── correlation_matrix_GSE41169.png
│   │   ├── age_prediction_GSE120307.png
│   │   ├── age_prediction_GSE41169.png
│   │   ├── deviation_heatmap_GSE120307.png
│   │   ├── deviation_heatmap_GSE41169.png
│   │   ├── mae_comparison.png
│   │   └── predicted_age_distributions.png
│   └── README.md
└── README.md                         ← you are here
```

---

## Part 1 — WGBS: Whole-Genome Bisulfite Sequencing

**Location:** `01_wgbs/README.md`

A conceptual and methodological deep-dive covering:

- What DNA methylation is at the chemical level (cytosine → 5-methylcytosine at CpG dinucleotides)
- How bisulfite conversion enables sequencing-based methylation detection
- How the Galaxy WGBS tutorial pipeline works step by step (QC → trimming → alignment → extraction)
- Summary and findings from the reference paper: *Hierarchical Clustering of Breast Cancer Methylomes Revealed Differentially Methylated and Expressed Breast Cancer Genes* (Lin et al., 2015)
- How WGBS compares to array-based methods like EPIC arrays

**Key tools covered:** FastQC / Falco, Trim Galore, bwameth, MethylDackel, DeepTools, metilene

---

## Part 2 — EPIC Array: Epigenetic Clock Benchmarking

**Location:** `02_epic_array_analysis/README.md`

A computational benchmarking analysis using the [Biolearn](https://github.com/bio-learn/biolearn) library covering:

- Two blood DNA methylation datasets from GEO (GSE120307, GSE41169)
- Eight aging clocks spanning three generations of development (Horvathv1, Hannum, PhenoAge, DunedinPACE, Lin, Zhang_10, YingCausAge, YingDamAge)
- Correlation matrix, deviation heatmap, age-prediction scatter plots, MAE benchmarking, and predicted age distributions — all evaluated on both datasets independently

**Key library:** `biolearn` (Ying et al., 2023)

---

## References

Lin I-H et al. (2015). *Hierarchical Clustering of Breast Cancer Methylomes Revealed Differentially Methylated and Expressed Breast Cancer Genes.* PLOS ONE. https://doi.org/10.1371/journal.pone.0118453

Ying et al. (2023). *Biolearn: An open-source library for biomarkers of aging.* bioRxiv. https://doi.org/10.1101/2023.12.02.569722

Galaxy Training Network. *DNA Methylation data analysis.* https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html

