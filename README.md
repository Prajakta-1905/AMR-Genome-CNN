# AMR-Genome-CNN
# Predicting Ciprofloxacin Resistance in *Staphylococcus aureus* Using Machine Learning and DNA Language Models

### A comparative genomic machine learning study using Random Forest, DNABERT-2, and CNN with Attention-based Multiple Instance Learning

---

## Overview

Antimicrobial resistance (AMR) is a major global health challenge, creating an urgent need for computational approaches that can identify and characterize bacterial resistance from genomic data.

This project investigates the prediction of ciprofloxacin resistance in *Staphylococcus aureus* using whole-genome sequences. It explores how different genomic representation strategies and machine learning architectures perform in distinguishing ciprofloxacin-resistant and susceptible isolates.

Three computational approaches are implemented and compared:

1. **Random Forest:** A conventional machine learning baseline using 5-mer nucleotide frequency features.
2. **DNABERT-2:** A pretrained DNA language model used to generate genome-level embeddings, followed by Logistic Regression.
3. **CNN + Attention-MIL:** A deep learning architecture that processes genomic windows using a 1D Convolutional Neural Network and aggregates their representations using attention-based Multiple Instance Learning.

The central objective is to investigate whether pretrained DNA representations offer an advantage over conventional genomic feature engineering and a custom deep learning architecture for antimicrobial resistance prediction.

## Research Objective

**Can whole-genome sequence representations be used to predict ciprofloxacin resistance in *Staphylococcus aureus*, and how do conventional machine learning, pretrained DNA language models, and deep learning approaches compare?**

The project focuses on ciprofloxacin, a fluoroquinolone antibiotic, and explores the predictive potential of genome-wide sequence information.

## Dataset

The dataset was obtained from the **BV-BRC (Bacterial and Viral Bioinformatics Resource Center)**.

| Parameter | Description |
|---|---|
| Organism | *Staphylococcus aureus* |
| Taxon ID | 1280 |
| Antibiotic | Ciprofloxacin |
| Phenotypic labels | Resistant / Susceptible |
| Initial binary-labelled records | 1,207 |
| Genomes used in modelling | 1,138 |
| Susceptible genomes | 664 |
| Resistant genomes | 474 |

Intermediate susceptibility records were excluded from the binary classification task.

### Dataset Splitting

The dataset was divided at the genome level using stratified sampling:

| Partition | Percentage | Genomes |
|---|---:|---:|
| Training | 70% | 796 |
| Validation | 15% | 171 |
| Testing | 15% | 171 |
| **Total** | **100%** | **1,138** |

The split used `random_state=42` and stratification based on resistance phenotype.

The test set was reserved for final model evaluation.

## Methodology

### 1. Random Forest — 5-mer Frequency Representation

The first approach establishes a conventional machine learning baseline.

Each bacterial genome is represented using the frequency distribution of all possible 5-nucleotide sequences (5-mers).

With four possible nucleotides, this produces: 4^5= 1024 features

These features represent the frequency of different short DNA sequence patterns across each genome.

The resulting feature vectors are used to train a Random Forest classifier.

**Model configuration:**

- Estimator: Random Forest
- Number of trees: 300
- Class weighting: `balanced_subsample`
- Maximum features: `sqrt`
- Minimum samples per leaf: 2
- Random state: 42

The classification threshold was selected using validation-set balanced accuracy.

### 2. DNABERT-2 — Pretrained DNA Language Model

The second approach explores pretrained DNA language model representations.

The pretrained **DNABERT-2 (117M)** model is used as a feature extractor to generate numerical representations of genomic sequences.

Rather than training the language model from scratch, its pretrained parameters are kept frozen.

The resulting genome-level embeddings have 768 dimensions and are used as input to a Logistic Regression classifier.

This approach investigates whether pretrained representations of DNA sequence information can provide useful features for antimicrobial resistance prediction.

Architecture:

                Genome Sequence
                       |
                       v
                  DNABERT-2
               (Pretrained, Frozen)
                       |
                       v
           768-Dimensional Embedding
                       |
                       v
              Logistic Regression
                       |
                       v
              Resistance Prediction
                 /           \
                v             v
           Resistant      Susceptible


### 3. CNN + Attention-based Multiple Instance Learning

The third approach uses a custom deep learning architecture designed to process bacterial genomes as collections of smaller genomic regions.

Instead of representing an entire genome as one feature vector, each genome is divided into overlapping sequence windows.

**Window configuration:**

- Window size: 500 base pairs
- Stride: 250 base pairs

Each window is converted into a numerical DNA representation and processed by a 1D Convolutional Neural Network.

The CNN acts as a feature extractor, producing window-level representations. An attention-based Multiple Instance Learning (MIL) aggregator then combines these representations into a genome-level embedding for classification.

The attention mechanism is intended to learn the relative contribution of different genomic windows to the final genome-level prediction.

Architecture:

                 Genome Sequence
                       |
                       v
             Overlapping DNA Windows
                (500 bp, 250 bp stride)
                       |
                       v
                    1D CNN
                       |
                       v
              Window Embeddings
                       |
                       v
                 Attention-MIL
                       |
                       v
           Genome-Level Representation
                       |
                       v
                Genome Classifier
                       |
                       v
              Resistance Prediction
                 /           \
                v             v
           Resistant      Susceptible

## Model Evaluation

All models were evaluated using classification metrics relevant to binary antimicrobial resistance prediction.

The evaluation metrics include:

- **Accuracy:** Overall proportion of correctly classified genomes.
- **Balanced Accuracy:** Average recall across the susceptible and resistant classes.
- **ROC-AUC:** Ability to distinguish between the two phenotypic classes across classification thresholds.
- **PR-AUC:** Precision-recall performance, particularly relevant to class-imbalanced classification.
- **Precision:** Proportion of predicted resistant genomes that were actually resistant.
- **Recall:** Proportion of actual resistant genomes correctly identified.
- **F1-score:** Harmonic mean of precision and recall.

## Results

### Test Set Performance

The following results were obtained from the three modelling experiments.

| Metric | Random Forest | DNABERT-2 | CNN + Attention-MIL |
|---|---:|---:|---:|
| Accuracy | 0.8480 | **0.8947** | 0.5848 |
| Balanced Accuracy | 0.8435 | **0.8855** | 0.5000 |
| ROC-AUC | 0.9097 | **0.9289** | 0.6246 |
| PR-AUC | 0.9061 | **0.9192** | 0.5195 |
| Resistant Precision | 0.8169 | **0.9077** | 0.0000 |
| Resistant Recall | 0.8169 | **0.8310** | 0.0000 |
| Resistant F1-score | 0.8169 | **0.8676** | 0.0000 |

### Key Findings

**DNABERT-2 achieved the strongest overall performance.**

The pretrained DNA language model with Logistic Regression obtained a test accuracy of 89.47% and a balanced accuracy of 88.55%, outperforming the other evaluated approaches.

**Random Forest provided a strong conventional machine learning baseline.**

Using 5-mer frequency features, Random Forest achieved 84.80% accuracy and 84.35% balanced accuracy. This demonstrates that relatively simple genomic feature engineering can provide useful predictive information.

**CNN + Attention-MIL struggled with resistance classification.**

The evaluated CNN+MIL training run achieved 58.48% accuracy but failed to identify resistant genomes at the 0.5 classification threshold. Its predicted probabilities were nearly constant across the test genomes, indicating a collapse in classification behaviour.

This highlights the challenges of learning useful genome-level resistance representations using the implemented CNN+MIL architecture and training configuration.

## Discussion

The comparison demonstrates that the choice of genomic representation and modelling architecture can substantially influence antimicrobial resistance prediction.

The performance of DNABERT-2 suggests that pretrained DNA language model representations may capture useful sequence patterns that support downstream resistance classification.

Random Forest also performed competitively, indicating that nucleotide composition and short-sequence frequency patterns contain predictive information.

However, the CNN+MIL experiment highlights an important challenge: learning meaningful genome-level representations from large collections of genomic windows does not necessarily translate into effective phenotype classification.

An important biological consideration is that fluoroquinolone resistance in *S. aureus* can involve specific mutations in quinolone-resistance-determining regions (QRDRs), particularly in genes such as `gyrA` and `grlA`. Genome-wide representations may not always adequately emphasize these relatively localized resistance-associated changes.

This provides motivation for future mutation-aware and gene-focused modelling approaches.

## Limitations

The current implementation has several limitations:

- The dataset is limited to *S. aureus* isolates with available ciprofloxacin susceptibility labels and retrieved genome sequences.
- Genome-wide 5-mer frequencies may not explicitly capture the biological significance of individual resistance-associated mutations.
- DNABERT-2 was used as a frozen feature extractor rather than fine-tuned specifically for antimicrobial resistance prediction.
- The CNN+MIL model did not successfully learn resistant-class predictions in the evaluated training run.
- Model performance depends on the available dataset, phenotype labels, genome quality, feature representations and experimental configuration.
- The reported results represent computational experiments and are not intended for clinical diagnostic use.

## Future Directions

Several improvements could extend this work:

1. **Mutation-aware prediction:** Incorporate resistance-associated mutations in genes such as `gyrA` and `grlA`.
2. **Fine-tuning DNA language models:** Investigate task-specific fine-tuning of pretrained DNA language models.
3. **Improved CNN architectures:** Explore alternative window-level feature extractors and MIL aggregation strategies.
4. **Genomic feature integration:** Combine sequence representations with known resistance-associated genetic markers.
5. **Lineage-aware evaluation:** Investigate the effect of bacterial population structure and genomic relatedness on model generalization.
6. **Interpretability:** Identify genomic regions and sequence patterns contributing to resistance predictions.

## Technologies Used

- Python
- NumPy
- Pandas
- Scikit-learn
- PyTorch
- Hugging Face Transformers
- DNABERT-2
- Matplotlib
- Bioinformatics and whole-genome sequence processing

## Project Structure

```text
AMR_Genome_CNN/
│
├── data/
│   ├── raw/
│   └── processed/
│       └── kmer_features/
│
├── notebooks/
│   ├── data_cleaning.ipynb
│   ├── genome_retrieval.ipynb
│   ├── genome_processing.ipynb
│   ├── amr_kmer_randomforest.ipynb
│   ├── amr_randomforest_comparison.ipynb
│   ├── amr-cnn-training.ipynb
│   └── dnabert2-amr-training.ipynb
│
├── models/
│
├── results/
│   └── model_comparison/
│       ├── final_model_comparison.csv
│       ├── final_model_comparison.png
│       ├── three_model_comparison.csv
│       ├── three_model_comparison.png
│       └── three_model_comparison.pdf
│
└── README.md
```

*The structure above describes the main project components; locally generated files and model artefacts may vary.*

## Reproducibility and Data Availability

The project uses genomic and antimicrobial susceptibility data obtained from BV-BRC.

Raw genomic sequences, processed features, pretrained model weights and trained model artefacts are not necessarily included in this repository.

To reproduce the experiments, the relevant input data, model checkpoints, preprocessing configurations and dependencies must be made available.

The pretrained DNABERT-2 checkpoint used in this study is:

`zhihan1996/DNABERT-2-117M`

## Acknowledgements

- **BV-BRC:** Bacterial and Viral Bioinformatics Resource Center, for bacterial genomic and antimicrobial susceptibility data.
- **DNABERT-2:** For the pretrained DNA language model used in the representation-learning experiment.
- The open-source Python and machine learning communities for the computational tools used in this project.

---

**Project focus:** Comparative evaluation of conventional machine learning, pretrained DNA language models and deep learning for genomic antimicrobial resistance prediction.


