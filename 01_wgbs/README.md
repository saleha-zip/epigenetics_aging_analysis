# 01 — WGBS: Whole-Genome Bisulfite Sequencing

This section covers the conceptual and methodological foundations of WGBS-based DNA methylation analysis. It is based on the Galaxy training tutorial, the introductory slides on DNA methylation, and the reference paper by Lin et al. (2015). No code is run here — this is the reading and understanding component of the assignment.

---

## Table of Contents

1. [What Is DNA Methylation?](#1-what-is-dna-methylation)
2. [How Bisulfite Conversion Works](#2-how-bisulfite-conversion-works)
3. [CpG Sites and Their Biological Significance](#3-cpg-sites-and-their-biological-significance)
4. [WGBS vs. Array-Based Methods](#4-wgbs-vs-array-based-methods)
5. [The Galaxy WGBS Pipeline — Step by Step](#5-the-galaxy-wgbs-pipeline--step-by-step)
6. [Reference Paper: Lin et al. (2015)](#6-reference-paper-lin-et-al-2015)
7. [Images to Download from the Galaxy Tutorial](#7-images-to-download-from-the-galaxy-tutorial)

---

## 1. What Is DNA Methylation?

DNA methylation is an epigenetic modification — a chemical change to DNA that affects gene expression without altering the underlying nucleotide sequence. In mammals, it almost always occurs at **cytosine** residues in the context of a **CpG dinucleotide** (a cytosine immediately followed by a guanine on the same strand).

A methyl group (–CH₃) is added to the fifth carbon of cytosine, converting it to **5-methylcytosine (5mC)**. This reaction is catalysed by a family of enzymes called **DNA methyltransferases (DNMTs)**:

- **DNMT3A and DNMT3B** establish *de novo* methylation patterns during development
- **DNMT1** maintains existing methylation patterns during cell division by methylating the newly synthesised strand after replication

### Why does it matter?

DNA methylation is not random. It is a regulatory layer that controls which genes are expressed in which cell types, when, and at what level:

- **Promoter methylation** generally silences gene expression by blocking transcription factor binding or recruiting repressor complexes
- **Gene body methylation** is often positively correlated with gene expression — a counterintuitive pattern that reflects active transcription
- **Enhancer methylation** can disrupt long-range regulatory interactions between enhancers and their target genes

Methylation patterns are largely cell-type-specific and are reprogrammed at key developmental stages (fertilisation, gametogenesis). In cancer and aging, these patterns become dysregulated — some regions gain aberrant methylation (hypermethylation), silencing tumour suppressor genes; others lose methylation (hypomethylation), destabilising repeat elements and activating oncogenes.

---

## 2. How Bisulfite Conversion Works

Bisulfite sequencing is the gold-standard method for detecting methylation at single-base resolution. The key chemical step exploits a difference in how methylated and unmethylated cytosines react with sodium bisulfite:

| Cytosine state | Reaction with bisulfite | Read in sequencing |
|---|---|---|
| **Unmethylated C** | Deaminated → uracil (U) | Sequenced as **T** |
| **Methylated C (5mC)** | Protected — no reaction | Sequenced as **C** |

After bisulfite treatment the DNA is amplified and sequenced. When you align the reads to a reference genome and look at any CpG position:

- A **C** in the read → that cytosine was **methylated** in the original DNA
- A **T** in the read → that cytosine was **unmethylated** in the original DNA

The **percent methylation** at a CpG site is simply:

```
methylation % = (reads with C at that position) / (total reads covering that position) × 100
```

### Important caveats

**Bisulfite conversion is destructive.** It degrades approximately 80–95% of input DNA, so library preparation requires special kits and sufficient starting material. Converted DNA is also no longer double-stranded in the traditional sense — the two strands are no longer complementary after conversion, which complicates alignment.

**Incomplete conversion** is a known source of false positives. Any unmethylated C that fails to convert will appear as a spurious methylation call. Conversion efficiency is typically assessed by including a spike-in DNA (often lambda phage, which is grown in methylase-negative bacteria and should have 0% methylation) and measuring how many of its Cs incorrectly appear as methylated. Acceptable conversion efficiency is typically >99%.

**The per-base sequence quality plots in FastQC will look abnormal** for bisulfite-converted data — an unusual T enrichment is expected and is actually a sign of successful conversion, not a quality failure.

---

## 3. CpG Sites and Their Biological Significance

A **CpG site** is a cytosine followed by a guanine in the 5′→3′ direction (the "p" refers to the phosphodiester bond between them). Although CpGs are statistically underrepresented in mammalian genomes (because methylated cytosines are mutation-prone and convert to thymines over evolutionary time), they cluster in regions called **CpG islands**.

**CpG islands** are regions roughly 200–3,000 bp long with elevated CpG density and high GC content. About 70% of human gene promoters are associated with a CpG island. These islands are usually unmethylated in normal cells, allowing the associated genes to be expressed. Silencing by methylation of CpG islands — particularly at tumour suppressor gene promoters — is one of the hallmarks of cancer epigenetics.

**CpG context types:**

- **CpG** — the primary site of methylation in mammals; the focus of most methylation analyses
- **CHG and CHH** (where H = A, T, or C) — methylated in plants and some stem cells; generally absent in somatic mammalian cells. Their presence in WGBS data can be used to assess incomplete bisulfite conversion

---

## 4. WGBS vs. Array-Based Methods

The two approaches in this assignment represent opposite ends of the cost/resolution/coverage trade-off:

| Feature | WGBS | EPIC Array (Illumina) |
|---|---|---|
| **Coverage** | Every cytosine in the genome (~28M CpGs in humans) | ~935,000 pre-selected CpG sites |
| **Resolution** | Single-base, genome-wide | Pre-defined loci only |
| **Cost** | High (~$300–1000+ per sample) | Lower (~$100–200 per sample) |
| **Data size** | Very large (50–100 GB per sample raw) | Manageable (intensity files ~10 MB) |
| **Bias** | Low — captures all CpGs including novel/intergenic | Array design bias — misses unannotated regions |
| **CpG context** | CpG, CHG, CHH all captured | CpG only |
| **Typical use** | Discovery, reference methylomes, structural features (HMRs, PMDs) | Population studies, biomarkers, aging clocks, clinical applications |
| **Alignment complexity** | High — bisulfite conversion complicates alignment | None — probes hybridise directly |
| **Epigenetic clocks** | Not typically used | The basis of all current aging clocks (Horvath, Hannum, PhenoAge, etc.) |

**Why do epigenetic clocks use arrays and not WGBS?**

Aging clocks were trained on array data because arrays have been applied to thousands of samples in large epidemiological cohorts, providing the statistical power needed to build and validate predictive models. WGBS is more expensive and complex, so it has been applied to far fewer samples. Additionally, clock CpGs are pre-specified loci, which arrays capture reliably and reproducibly across labs — WGBS coverage at any individual CpG depends on sequencing depth and can be incomplete in low-coverage experiments.

WGBS, on the other hand, is indispensable for discovering structural methylation features — hypomethylated regions (HMRs), partially methylated domains (PMDs), allele-specific methylation, and methylation in repetitive or novel genomic regions that are entirely absent from arrays.

---

## 5. The Galaxy WGBS Pipeline — Step by Step

The Galaxy tutorial walks through a real WGBS pipeline applied to human breast tissue data. The tools and steps are described below.

### Overview

```
Raw reads (FASTQ)
       │
       ▼
 Step 1: Quality Control (FastQC / Falco)
       │
       ▼
 Step 2: Read Trimming (Trim Galore)
       │
       ▼
 Step 3: Alignment to Reference Genome (bwameth)
       │
       ▼
 Step 4: Methylation Extraction (MethylDackel)
       │
       ▼
 Step 5: Visualisation (DeepTools, bigWig tracks)
       │
       ▼
 Step 6: Differential Methylation Analysis (metilene)
```

---

### Step 1: Quality Control — FastQC / Falco

**Tool:** FastQC or Falco (a faster drop-in replacement)

**Purpose:** Assess the raw read quality before any processing.

**What to look at:**
- **Per-base sequence quality** — should be high (green) across the read length; drop-off at the 3′ end is normal
- **Per-base sequence content** — *will appear as a failure* in bisulfite data because the T:C ratio is heavily skewed due to conversion. This is expected and not a real quality problem
- **Adapter content** — identifies sequencing adapters that must be trimmed
- **Duplication levels** — can be inflated in WGBS if library complexity is low

> **Galaxy tutorial screenshot to save:** the FastQC per-base sequence content plot showing the characteristic T-skew of bisulfite-converted data. This is the most visually distinctive result of the QC step.

---

### Step 2: Read Trimming — Trim Galore

**Tool:** Trim Galore (wraps Cutadapt + FastQC)

**Purpose:** Remove adapter sequences and low-quality bases from the ends of reads.

**What it does:**
- Removes Illumina adapter sequences that appear when the read length exceeds the insert size
- Trims low-quality 3′ bases (default quality cutoff Phred 20)
- Re-runs FastQC on trimmed reads to confirm improvement

**Why this matters for bisulfite data:** The 5′ ends of reads can show methylation bias (m-bias) — artificially inflated or deflated methylation levels due to random priming artefacts in library preparation. Trim Galore's `--clip_R1` and `--clip_R2` options allow trimming of these biased positions.

---

### Step 3: Alignment — bwameth

**Tool:** bwameth (bisulfite-aware aligner using BWA-MEM)

**Purpose:** Map the bisulfite-converted reads back to the reference genome.

**Why alignment is hard for bisulfite data:**

Standard aligners assume that C in the read should match C in the reference. After bisulfite conversion, unmethylated Cs become Ts — so a read that originated from an unmethylated CpG will have a T where the reference has a C. A naive aligner would either fail to map this read or report it as a mismatch.

Bisulfite-aware aligners solve this by converting all Cs in both the reads and the reference to Ts during the alignment step (in silico), then reporting the original sequence. This allows the aligner to place reads correctly while preserving the C/T distinction needed to call methylation.

**What bwameth produces:**
- A BAM file of aligned reads
- An alignment rate (typically 70–85% for WGBS, lower than regular sequencing due to the reduced sequence complexity after conversion)

> **Galaxy tutorial screenshot to save:** the bwameth alignment statistics output showing mapping rate and read counts.

---

### Step 4: Methylation Extraction — MethylDackel

**Tool:** MethylDackel (formerly PileOMeth)

**Purpose:** Extract per-CpG methylation calls from the aligned BAM file.

**What it does:**

For every CpG position in the genome covered by at least one read, MethylDackel counts:
- How many reads show a C at that position (methylated)
- How many reads show a T at that position (unmethylated)

It calculates the methylation fraction: `C / (C + T)`.

**MethylDackel also detects m-bias** — it plots methylation level as a function of position within the read. If the first or last few bases show unusual methylation levels (especially at read 2's 5′ end in paired-end data), those positions can be excluded from the final extraction.

**Output format:** A bedGraph file with one row per covered CpG:
```
chromosome   start   end   methylation_fraction
chr1         10468   10469   0.85
chr1         10470   10471   0.23
```

> **Galaxy tutorial screenshot to save:** the MethylDackel m-bias plot, which shows per-position methylation across the read. Flat is ideal; sloping ends indicate bias that needs to be clipped.

---

### Step 5: Visualisation — DeepTools / bigWig

**Tool:** DeepTools (`bamCoverage`, `computeMatrix`, `plotProfile`)

**Purpose:** Visualise genome-wide methylation patterns, particularly around functionally important features like transcription start sites (TSS).

**What the tutorial does:**

1. Converts the MethylDackel bedGraph output to bigWig format for IGV or genome browser visualisation
2. Uses `computeMatrix` to centre methylation signal on all TSS positions in the genome
3. Uses `plotProfile` to show the average methylation level at TSS across all genes

**Expected result:** A characteristic "valley" of low methylation centred on the TSS, flanked by higher methylation. This is the signature of active promoter CpG islands — they are unmethylated, allowing transcription factor access.

> **Galaxy tutorial screenshot to save:** the `plotProfile` output showing the TSS methylation valley. This is one of the most biologically interpretable figures in the tutorial.

---

### Step 6: Differential Methylation — metilene

**Tool:** metilene

**Purpose:** Identify differentially methylated regions (DMRs) between two groups of samples (e.g., normal vs. tumour).

**What it does:**

Rather than testing individual CpGs, metilene segments the genome into regions and tests whether consecutive CpGs in a region are consistently more or less methylated in one group versus another. This regional approach is preferred because:
- Individual CpG tests produce millions of p-values with severe multiple-testing burden
- Biologically meaningful methylation changes tend to span regions of dozens to hundreds of CpGs, not isolated sites
- Regional tests are more robust to incomplete coverage at individual CpGs

**Output:** A BED file of DMRs with mean methylation difference, q-value, number of CpGs, and genomic coordinates.

> **Galaxy tutorial screenshot to save:** the metilene output summary plot showing DMR methylation differences, DMR lengths, and the scatter of mean group methylation values.

---

## 6. Reference Paper: Lin et al. (2015)

> Lin I-H, Chen D-T, Chang Y-F, Lee Y-L, Su C-H, Cheng C, et al. (2015). *Hierarchical Clustering of Breast Cancer Methylomes Revealed Differentially Methylated and Expressed Breast Cancer Genes.* PLOS ONE 10(2): e0118453. https://doi.org/10.1371/journal.pone.0118453

### Study Design

The authors applied WGBS to seven breast tissue samples representing a progression from normal to cancerous tissue:

| Sample | Type |
|---|---|
| Normal breast tissue (×2) | Healthy control |
| Fibroadenoma | Benign tumour |
| Invasive ductal carcinoma (×2) | Malignant primary tumour |
| MCF7 | ER+ breast cancer cell line |
| HCC1954 | HER2+ breast cancer cell line |

This design allows the methylome to be tracked across a disease continuum, from normal to benign to malignant to established cancer cell lines.

### Key Concepts

**Hypomethylated Regions (HMRs)**

The authors defined HMRs as contiguous stretches of CpGs with consistently low methylation (below a threshold). They found that the distribution of HMRs — their number, size, and location — changes substantially in cancer:

- Normal cells have a stable set of HMRs at active promoters and enhancers
- Cancer cells gain new HMRs in regions that are methylated in normal tissue (hypomethylation)
- Cancer cells also lose HMRs at regions that should be active (hypermethylation)

The size distribution of HMRs ranges from kilobase-scale (corresponding to individual regulatory elements) to megabase-scale.

**Partially Methylated Domains (PMDs)**

PMDs are large genomic regions (typically hundreds of kilobases to megabases) with intermediate, heterogeneous methylation. They are enriched in late-replicating, gene-poor regions of the genome. In cancer, PMDs expand significantly — large regions of the genome that are stably methylated in normal cells become partially demethylated. This is thought to reflect a loss of methylation maintenance fidelity during the rapid cell division of tumour growth.

### Major Findings

**1. Hierarchical clustering of HMRs separates normal from cancer tissue**

When the authors clustered samples by their HMR methylation levels at promoters, intragenic regions, and intergenic regions, normal tissue and cancer samples formed distinct clusters. This demonstrates that the global methylation landscape — not just individual gene promoters — is reorganised in a consistent, tumour-specific pattern.

Eight distinctive HMR clusters were identified at each genomic location. The most biologically important clusters were those showing:
- Consistently low methylation in all samples (constitutive regulatory elements)
- Hypomethylation specific to cancer cell lines (MCF7, HCC1954) relative to normal (potential oncogenic activation)
- Hypermethylation in cancer samples relative to normal (potential tumour suppressor silencing)

**2. Cancer-specific HMRs overlap with known breast cancer regulatory elements**

The cancer-specific hypomethylated clusters were enriched for enhancers active in breast cancer cell lines and for transcription factor binding sites associated with cancer biology (e.g., FOXA1, GATA3 — both known breast cancer transcription factors). This suggests that methylation loss at enhancers may contribute to aberrant activation of breast cancer gene programmes.

**3. Joint methylation–expression analysis identifies candidate genes**

By integrating WGBS methylation data with gene expression data (RNA-seq), the authors identified genes where:
- Hypermethylation at the promoter in cancer correlates with reduced expression (candidate tumour suppressors silenced by methylation)
- Hypomethylation at the promoter in cancer correlates with increased expression (candidate oncogenes activated by methylation loss)

These genes included several known breast cancer and ovarian cancer genes, validating the approach, as well as novel candidates.

**4. Aberrant X-chromosome inactivation in breast cancer**

A striking finding was evidence of disrupted X-chromosome inactivation (XCI) in cancer samples. In normal female cells, one X chromosome is inactivated by coating with XIST RNA; the XIST promoter is unmethylated on the inactive X and the gene is expressed. In cancer:

- The XIST promoter was hypermethylated in cancer cell lines → XIST expression was reduced → the inactive X was no longer properly silenced
- X-linked genes that should be silenced on the inactive X were instead hypomethylated and overexpressed
- High expression of these X-linked genes in TCGA breast cancer data was associated with significantly worse patient survival

### Relevance to the Galaxy Tutorial

The Galaxy WGBS tutorial uses data derived from this study. The pipeline steps (QC → trimming → alignment → methylation extraction → visualisation → DMR calling) are exactly the steps the Lin et al. authors performed to generate their methylomes. The tutorial makes these steps reproducible for educational purposes using a subset of the data.

---

## 7. Images to Download from the Galaxy Tutorial

You do not need to run the Galaxy pipeline to include representative figures in this README. Download the following images directly from the Galaxy tutorial page and save them to `01_wgbs/images/`:

> **Tutorial URL:** https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html

### Images to save and their filenames

| Image to find in tutorial | Suggested filename | What it shows |
|---|---|---|
| FastQC per-base sequence content plot (bisulfite-converted data) | `fastqc_per_base_content.png` | T-skew caused by bisulfite conversion — the visual signature that conversion worked |
| Falco / FastQC quality score plot (pre- or post-trimming) | `fastqc_quality_scores.png` | Per-base quality across the read; shows where 3′ trimming is needed |
| MethylDackel m-bias plot | `methyldackel_mbias.png` | Per-position methylation across read length; flat = good, sloping ends = bias to trim |
| DeepTools `plotProfile` TSS methylation plot | `deeptools_tss_profile.png` | Average methylation centred on all TSS; the promoter valley is the key biological signal |
| metilene DMR output summary plot | `metilene_dmr_summary.png` | Distribution of DMR sizes, methylation differences, and q-values |

### How to save them

On the tutorial page, right-click each figure → "Save image as" → save to `01_wgbs/images/` with the filenames above.

Once downloaded, the images will be referenced in the README like this:

```markdown
![FastQC per-base sequence content](images/fastqc_per_base_content.png)
```

Place this placeholder section in the README wherever the images should appear — or let me know once you have downloaded them and I can insert the references and captions directly.

---

## References

Lin I-H et al. (2015). *Hierarchical Clustering of Breast Cancer Methylomes Revealed Differentially Methylated and Expressed Breast Cancer Genes.* PLOS ONE. https://doi.org/10.1371/journal.pone.0118453

Galaxy Training Network. *DNA Methylation data analysis.* https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html

Galaxy Training Network. *Introduction to DNA Methylation.* https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/introduction-dna-methylation/slides-plain.html

Krueger F, Andrews SR. (2011). *Bismark: a flexible aligner and methylation caller for Bisulfite-Seq applications.* Bioinformatics. https://doi.org/10.1093/bioinformatics/btr167

Yin et al. (2019). *MethylDackel.* https://github.com/dpryan79/MethylDackel

