# Learning EEG Representations for Sleep Deprivation Classification and Sleepiness Prediction

Deep learning project for learning representations from resting-state EEG and testing whether those representations transfer from **sleep deprivation classification** to downstream **subjective sleepiness prediction**.

The project studies two related questions:

1. Can EEG epochs reliably distinguish **normal sleep (NS)** from **sleep deprivation (SD)**?
2. Can representations learned for that classification task support prediction of session-level **Stanford Sleepiness Scale (SSS)** and **Karolinska Sleepiness Scale (KSS)** scores?

## Highlights

- Built an end-to-end EEG preprocessing, training, and evaluation workflow over **8,300 EEG samples**.
- Used **subject-wise train/validation/test splits** to prevent identity leakage across experimental partitions.
- Evaluated **Residual EEG CNN**, **2-Branch EEG CNN**, **DeiT-Tiny**, and **ViT-Small** architectures for NS vs. SD classification.
- Reused learned EEG representations for downstream sleepiness prediction with **Ridge, MLP, GRU, and LSTM** models.
- Compared feature-based transfer, direct ordinal prediction, and multi-task learning approaches.
- Implemented experiments in PyTorch with MNE, scikit-learn, timm, NumPy, Pandas, and SciPy.

## Experimental Design

A central methodological constraint in EEG modeling is avoiding subject leakage. Randomly splitting epochs can place recordings from the same individual in both training and evaluation data, producing overly optimistic performance estimates.

This project therefore uses **subject-wise splitting** so that subjects in the training set do not appear in validation or test data.

The stored classification split contains:

| Split | EEG samples | Subjects |
| --- | ---: | ---: |
| Train | 5,738 | 49 |
| Validation | 1,246 | held-out subjects |
| Test | 1,316 | held-out subjects |
| **Total** | **8,300** | **subject-disjoint across splits** |

This split is reused across the downstream experiments where applicable so that evaluation remains consistent with the original representation-learning setup.

## Phase 1: Sleep Deprivation Classification

The first stage learns EEG representations by classifying each epoch as:

- **NS**: normal sleep condition
- **SD**: sleep-deprived condition

### Models compared

- **Residual EEG CNN**
- **2-Branch EEG CNN**
- **DeiT-Tiny**
- **ViT-Small**

The **Residual EEG CNN** is used as the primary encoder for downstream representation-transfer experiments.

Conceptually, Phase 1 is:

```text
Resting-state EEG
      |
      v
Preprocessing
      |
      v
EEG representation learner
      |
      +----> NS vs. SD classifier
      |
      +----> learned embedding for Phase 2
```

## Phase 2: Sleepiness Prediction

The second stage asks whether representations learned from the NS vs. SD task capture information useful for predicting subjective sleepiness.

Two session-level targets are studied:

- **SSS**: Stanford Sleepiness Scale
- **KSS**: Karolinska Sleepiness Scale

Three modeling strategies are explored.

### 1. Feature-based transfer

Extract representations from the pretrained Residual EEG CNN and use them as inputs to downstream models including:

- Ridge regression
- MLP
- GRU
- LSTM

This tests whether the Phase 1 encoder learns reusable EEG features without requiring the entire network to be retrained for sleepiness prediction.

### 2. Direct ordinal prediction

Train EEG models directly against SSS or KSS targets using ordinal supervision.

This provides an end-to-end alternative to the representation-transfer pipeline.

### 3. Multi-task learning

Train a shared EEG encoder jointly for:

- NS vs. SD classification
- sleepiness prediction

The goal is to study whether the classification task provides useful auxiliary supervision for subjective sleepiness estimation.

## Repository Structure

```text
.
├── code/
│   ├── sleep_deprivation_classification/
│   │   ├── classification experiments
│   │   └── subject-wise split artifacts
│   └── sleepiness_score_prediction/
│       ├── feature-based SSS/KSS prediction
│       ├── direct prediction experiments
│       └── multi-task SSS/KSS experiments
├── requirements.txt
└── README.md
```

The experiments are primarily provided as notebooks under `code/`.

## Tech Stack

**Deep Learning**

- PyTorch
- torchvision
- timm

**EEG and Scientific Computing**

- MNE
- NumPy
- Pandas
- SciPy
- scikit-learn

**Model families**

- Residual CNNs
- Multi-branch CNNs
- Vision Transformers
- GRU / LSTM sequence models
- MLP and linear baselines

## Data

Raw and preprocessed project data are stored separately from the repository:

[Project data folder](https://drive.google.com/drive/folders/1meAulHb0yytaVB1TZRkgO1hG_Lgp4cI_?usp=sharing)

The repository contains the modeling code and experimental notebooks rather than duplicating the full EEG dataset in Git.

## Setup

Clone the repository and install the pinned dependencies:

```bash
git clone https://github.com/chinmayarvind23/eeg-sleep-deprivation-classification-and-sleepiness-prediction.git
cd eeg-sleep-deprivation-classification-and-sleepiness-prediction
pip install -r requirements.txt
```

The main experiments can then be opened from the notebooks under `code/`.

## Why This Project Matters

EEG models can easily appear stronger than they are when evaluation allows information from the same participant to leak across splits. This project treats **subject-independent evaluation** as a first-class constraint and then asks a harder transfer-learning question: whether representations learned from an objective condition label, sleep deprivation, contain useful information for a related subjective outcome, perceived sleepiness.

That makes the project both a classification benchmark and a representation-learning study across related physiological prediction tasks.

## Team

- Rushendra Sidibomma
- **Chinmay Arvind**
- Samarth Kumar Samal
- Arno Benzigar
