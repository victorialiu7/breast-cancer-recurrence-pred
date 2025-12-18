# HMM-Based Gene Expression Risk Classifier for Breast Cancer Recurrence

A Hidden Markov Model (HMM) implementation for predicting patient risk levels of breast cancer recurrence based on gene expression data from biological pathway gene sets.

## Overview

This project uses HMMs to model gene expression patterns and classify patients into high-risk and low-risk categories. The approach:

1. Computes module scores from gene sets (biological pathways)
2. Ranks modules by discriminative power
3. Discretizes continuous expression scores into observation states
4. Trains separate HMMs for high-risk and low-risk patient groups
5. Predicts risk by comparing likelihoods from both models

## Requirements

- Python 3.8+
- See `requirements.txt` for dependencies

## Setup Instructions

1. **Clone the repository:**
```bash
git clone victorialiu7/cs4775-proj
cd cs4775-proj
```

2. **Create virtual environment:**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies:**
```bash
pip install -r requirements.txt
```

4. **Download and prepare the data:**

   **Step 4a: Download METABRIC data from cBioPortal**
   
   1. Go to [cBioPortal for Cancer Genomics](https://www.cbioportal.org/)
   2. Search for and select the **METABRIC** (Molecular Taxonomy of Breast Cancer International Consortium) study
   3. Download the complete dataset
   4. Unzip the downloaded file (`brca_metabric.tar.gz` or similar)
   5. From the extracted folder, copy these two files to your project directory:
      - `data_clinical_patient.txt` - Clinical patient data including risk labels
      - `data_mrna_illumina_microarray.txt` - Gene expression microarray data
   
   **Step 4b: Create merged dataset**
   
   You'll need to merge the clinical and expression data to create `merged.csv`. The merged file should have:
    - Gene expression columns (one per gene)
    - `PATIENT_ID` - Patient identifier
    - `high_risk` - Binary label (True/False) derived from clinical data
   
   **Step 4c: Gene sets**
   
    You need to create enrich-input1-GO_BP.tsv - Gene sets from GO Biological Process enrichment:

    Get seed gene list:
    - The paper uses a 322-gene list from "Consensus genes of the literature to predict breast cancer recurrence"
    - Alternative: Download the 41-gene CBCG list from CBCG website (https://cbcg.dk/causal.html)

    Perform enrichment analysis:
    - Go to GeneCodis (or similar tool like Enrichr, DAVID)
    - Input your gene list
    - Run GO Biological Process enrichment analysis
    - Download results as TSV

    Required format:
    - Tab-separated file with a genes column
    - Each row contains comma-separated gene names (e.g., "TP53, BRCA1, EGFR")
    - Filter to keep only gene sets with 2+ genes (to avoid zero standard deviation in calculations)

## Usage

This project runs the HMM-based classifier through a Jupyter notebook.

To execute the full pipeline, open and run:

    ranking+hmm+eval.ipynb

Run all cells sequentially from top to bottom. The notebook performs data loading, gene set construction, ranking, HMM training, prediction, and evaluation.

---

## Key Parameters

The following parameters can be modified directly in the notebook:

- **Input data**
  - Gene expression data file:
  
        data_file = "merged.csv"

- **Gene set definition**
  - GO biological process gene sets:
  
        cbcg41_pd = pd.read_csv("enrich-input1-GO_BP.tsv", sep="\t")
  
  - Only gene sets containing more than one gene are used.

- **Number of hidden states**
  - Defined during HMM training:
  
        eh = hmm_rank(train_data_h, 6)
        el = hmm_rank(train_data_l, 6)
  
  - Here, `6` specifies the number of hidden states.

- **Cross-validation**
  - Number of folds:
  
        k_fold = 10
  
  - Samples are randomly shuffled and split into training and validation sets according to this value.

---

## Method Overview

The notebook implements the following pipeline:

1. **Gene set construction**  
   Genes are grouped into biological process gene sets derived from GO enrichment results. Each gene set is summarized per patient using a standardized statistic based on the mean and standard deviation of its member genes.

2. **Gene set ranking**  
   For each patient, gene sets are ranked according to their standardized scores. These ranked gene sets form the observation sequences used by the HMM.

3. **HMM training**  
   Two separate HMMs are trained:
   - One using high-risk samples
   - One using low-risk samples  

   Emission probabilities are estimated from empirical rank frequencies. A left-to-right transition structure is assumed.

4. **Prediction**  
   Each validation sample is evaluated under both HMMs. Classification is based on which model produces the higher log-likelihood. A posterior probability of high risk (`prob_high`) is also computed.

5. **Evaluation**  
   Predictions are compared against ground-truth labels to compute classification performance metrics.

---

## Output

The notebook reports:

- **Confusion matrix**
  - True positives (TP), true negatives (TN), false positives (FP), and false negatives (FN)

- **Performance metrics**
  - Area Under the ROC Curve (AUC)
  - Matthews Correlation Coefficient (MCC)

### Example Output

    AUC: 0.6206
    MCC: 0.2536

A confusion matrix is also displayed using `pd.crosstab`, showing predicted versus true risk labels.


## Project Structure

```
.
├── ranking+hmm+eval.ipynb              # Main HMM classifier notebook
├── data_clinical_patient.txt           # METABRIC clinical data (not in repo)
├── data_mrna_illumina_microarray.txt   # METABRIC expression data (not in repo)
├── merged.csv                          # Processed merged dataset (not in repo)
├── enrich-input1-GO_BP.tsv            # Gene sets (not in repo)
├── requirements.txt                    # Python dependencies
├── .gitignore                         # Git ignore rules
└── README.md                          # This file
```

**Note:** Data files are excluded from the repository per `.gitignore`. Users must download the METABRIC dataset from cBioPortal as described in Setup Instructions.
