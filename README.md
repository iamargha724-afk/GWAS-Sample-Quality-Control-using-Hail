# 🧬 Genome-Wide Association Studies (GWAS) Workshop

<p align="center">
  

<p align="center">
  <img src="https://img.shields.io/badge/Organised%20By-Dr.%20Omics%20Labs-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Duration-5%20Days-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Date-4th%20March%202026-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Completed%20✓-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Tool-Hail-blueviolet?style=for-the-badge" />
</p>

---

## 📜 About This Repository

This repository documents my hands-on work and learnings from the **5-Day Workshop on Genome-Wide Association Studies (GWAS)**, organised by **Dr. Omics Labs** on **4th March 2026**.

The workshop covered end-to-end GWAS analysis using the **Hail** framework — from loading raw VCF data into a MatrixTable, performing quality control, running association tests, to visualising results.

---

## 🔬 What is GWAS?

**Genome-Wide Association Studies (GWAS)** scan the entire genome across thousands of individuals to find genetic variants — primarily **Single Nucleotide Polymorphisms (SNPs)** — statistically associated with traits or diseases.

GWAS is foundational to:
- Identifying genetic risk loci for complex diseases (diabetes, cancer, cardiovascular disease)
- Understanding heritability and polygenic architecture
- Building **Polygenic Risk Scores (PRS)** for personalised medicine

---

## 🛠️ Tools & Framework

### Hail (Primary Tool)

[Hail](https://hail.is) is an open-source, scalable framework for genomic data analysis built on Apache Spark. In this workshop, we used Hail's **MatrixTable** data structure to represent multi-sample genomic data efficiently.

```python
import hail as hl
hl.init()
```

### MatrixTable Schema

The core data structure used in this workshop was a Hail **MatrixTable** loaded from VCF data (GRCh37 reference genome):

```
Type:
    struct {
        locus: locus<GRCh37>,
        alleles: array<str>,
        rsid: str,
        qual: float64,
        filters: set<str>,
        info: struct {
            AC: array<int32>,       # Allele Count
            AF: array<float64>,     # Allele Frequency
            AN: int32,              # Total Allele Number
            BaseQRankSum: float64,
            ClippingRankSum: float64,
            DP: int32,              # Total Depth
            DS: bool,               # Downsampled flag
            FS: float64,            # Fisher Strand Bias
            HaplotypeScore: float64,
            InbreedingCoeff: float64,
            MLEAC: array<int32>,    # Maximum Likelihood Allele Count
            MLEAF: array<float64>,  # Maximum Likelihood Allele Frequency
            MQ: float64,            # Mapping Quality
            MQ0: int32,
            MQRankSum: float64,
            QD: float64,            # Quality by Depth
            ReadPosRankSum: float64,
            set: str
        }
    }

Source: <hail.matrixtable.MatrixTable object>
Index: ['row']
```

---

## 📋 Workshop Workflow

### 1. 📥 Data Loading
Loading VCF files into Hail MatrixTable format using GRCh37 reference:

```python
mt = hl.import_vcf('data/sample.vcf.bgz', reference_genome='GRCh37')
mt.describe()
```

### 2. 🔍 Quality Control (QC)

**Variant-level QC** — filtering based on INFO fields:

```python
mt = mt.filter_rows(
    (mt.info.QD >= 2.0) &
    (mt.info.FS <= 60.0) &
    (mt.info.MQ >= 40.0) &
    (mt.info.ReadPosRankSum >= -8.0)
)
```

**Sample-level QC:**

```python
mt = hl.sample_qc(mt)
mt = mt.filter_cols(
    (mt.sample_qc.call_rate >= 0.95) &
    (mt.sample_qc.dp_stats.mean >= 10)
)
```

### 3. 📊 Variant Annotation

Annotating variants with rsIDs, allele frequencies, and functional consequences using Hail's annotation framework.

### 4. 🧮 Association Testing

Running logistic regression for case-control phenotypes:

```python
gwas = hl.logistic_regression_rows(
    test='wald',
    y=mt.pheno.is_case,
    x=mt.GT.n_alt_alleles(),
    covariates=[1.0, mt.pheno.age, mt.pheno.sex, mt.pca_scores[0], mt.pca_scores[1]]
)
```

### 5. 📈 Visualisation

Generating **Manhattan plots** and **QQ plots** to visualise association results and assess genomic inflation.

---

## 📁 Repository Structure

```
GWAS-Workshop-DrOmicsLabs/
│
├── 📄 README.md
├── 📁 certificate/
│   └── certificate.png
│
├── 📁 notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_quality_control.ipynb
│   ├── 03_association_testing.ipynb
│   └── 04_visualisation.ipynb
│
├── 📁 scripts/
│   ├── qc_pipeline.py
│   ├── gwas_analysis.py
│   └── plot_results.py
│
├── 📁 results/
│   ├── manhattan_plot.png
│   └── qq_plot.png
│
└── 📁 data/
    └── README.md           # Data access instructions
```

---

## 🧪 Key Concepts Covered

| Topic | Description |
|-------|-------------|
| **MatrixTable** | Hail's 2D data structure for rows (variants) × columns (samples) |
| **VCF INFO Fields** | AC, AF, AN, DP, QD, FS, MQ — key fields for variant QC |
| **SNP Filtering** | Hard filters using QD, FS, MQ, ReadPosRankSum thresholds |
| **Population Stratification** | PCA-based correction using top principal components |
| **Logistic Regression** | Wald test for binary case/control phenotypes |
| **Manhattan Plot** | Genome-wide -log10(p-value) visualisation |
| **QQ Plot** | Expected vs. observed p-value distribution |
| **LD Clumping** | Identifying independent significant signals |

---

## ⚙️ Environment Setup

```bash
# Create conda environment
conda create -n gwas python=3.9
conda activate gwas

# Install Hail
pip install hail

# Install additional packages
pip install pandas matplotlib seaborn jupyter
```

---

## 🏅 Accreditations

Dr. Omics Labs is recognised under:

| Accreditation | Detail |
|---|---|
| Startup India | DIPP21150 |
| MSME | Certified |
| ISO | 9001:2015 Certified |
| Partners | AWS, Illumina |

---

## 📚 References & Resources

- [Hail Documentation](https://hail.is/docs/0.2/)
- [GWAS Catalog](https://www.ebi.ac.uk/gwas/)
- [PLINK2](https://www.cog-genomics.org/plink/2.0/)
- [Workshop Resources (Google Drive)](https://drive.google.com/drive/folders/1KNh5ZFod81kmaLUsAwFyPMnCGs1WadOg?usp=sharing)

---
