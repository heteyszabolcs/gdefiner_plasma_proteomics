# Plasma Proteomics Differential Analysis & GSEA Pipeline

## Overview

This repository contains a reproducible workflow for analyzing plasma proteomics data using:

- Linear modeling (**limma**) for differential protein expression  
- Gene Set Enrichment Analysis (**GSEA**, preranked via `gseapy`)  

The pipeline is designed for **multi-cohort clinical proteomics data** with repeated measures and confounding variables.

---

## 🧬 Data Description

### 1. `sample_info.tsv`

Metadata table describing each sample.

| Column | Description |
|------|------------|
| Sample ID | Patient identifier (prefix indicates country: SWE, IRE, ITA, NOR) |
| TP | Timepoint (T0, T1, T2) |
| Sex | Biological sex (M / F) |
| CF ID | Unique sample identifier (used to map quantification data) |
| cancer_type | Cancer type |
| toxicity | Toxicity status (yes / no) |
| country | Country of origin |
| ... | Additional clinical covariates |

---

### 2. `meta.tsv`

Mapping table linking CF IDs to raw file names.

| Column | Description |
|--------|------------|
| Sample ID | CF ID |
| R.FileName | Identifier used in quantification matrix |

---

### 3. `plasma_prot_quant.tsv`

Protein quantification matrix.

| Column | Description |
|--------|------------|
| PG.Genes | Protein / gene annotation |
| Other columns | Sample-specific intensities (mapped via `meta.tsv`) |

---

## ⚙️ Data Processing Workflow

### 1. Column Mapping
- Match quantification columns to **CF IDs** using `meta.tsv`
- Rename columns to CF IDs

### 2. Gene Name Cleaning
- Multi-annotation entries (e.g. `ARHGEF5;ARHGEF5;...`) are simplified to:

### 3. Expression Matrix Construction
- Rows → samples (CF IDs)  
- Columns → proteins  
- Values → normalized intensities  

### 4. Preprocessing
- Remove duplicated samples
- Convert to numeric
- Log2 transform

---

## 📊 Differential Expression Analysis (limma)

We use the **limma** framework for linear modeling with empirical Bayes moderation.

### Key Features

- Supports flexible contrasts:
- `Sex: M vs F`
- `TP: T2 vs T0`
- `toxicity: yes vs no`
- Adjusts for confounders:
- country
- cancer_type
- TP
- toxicity

---

### Example Usage

```r
results <- limma_differential(
expr = expr_matrix,
sample_info = sample_info,
contrast_feature = "Sex",
contrast_direction = "M_vs_F",
confounding_factors = c("country", "cancer_type", "TP", "toxicity")
)
