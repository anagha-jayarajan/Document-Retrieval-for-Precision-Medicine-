# Biomedical Information Retrieval for the TREC 2019 Precision Medicine Track

## Overview

This project implements a biomedical information retrieval pipeline inspired by the multi-stage retrieval and reranking approach described by Liu et al. for the **TREC 2019 Precision Medicine Track**.

The system combines traditional information retrieval techniques with deep learning models to improve the ranking of biomedical documents for precision medicine queries. It progresses from keyword-based retrieval using BM25 to semantic reranking using neural models and a logistic regression ensemble.

The pipeline consists of two major phases:

1. **Initial Retrieval:** Text preprocessing, biomedical entity normalization, query expansion, weighted query boosting, and BM25-based document retrieval.
2. **Deep Learning Reranking:** Treatment-focus classification and disease-related semantic matching, combined with BM25 scores through a logistic regression ensemble.

## Pipeline Architecture

```text
Structured Patient Queries
          |
          v
    Text Preprocessing
          |
          v
 Biomedical Entity Normalization
          |
          v
 Query Expansion and Weighting
          |
          v
      BM25 Retrieval
          |
          v
   Candidate Documents
          |
          +-----------------------+
          |                       |
          v                       v
    BiGRU-Att Model         MatchPyramid Model
          |                       |
          v                       v
 Treatment-Focus Score     Disease-Matching Score
          |                       |
          +-----------+-----------+
                      |
                      v
             Feature Combination
                      |
                      v
          Logistic Regression Ensemble
                      |
                      v
              Final Ranking
                      |
                      v
             Evaluation Metrics
```

## Phase 1: Initial Retrieval

### 1. Text Preprocessing

The pipeline applies a sequence of text normalization operations to patient queries and biomedical documents:

- Lowercasing text for consistent matching.
- Numeric display-token substitution to normalize numeric expressions.
- Stop-word removal to reduce unnecessary terms.
- Porter stemming to normalize word variations.
- Biomedical entity normalization using a local CTD/NCBI gene synonym database.

These operations help standardize terminology and improve matching between patient queries and biomedical literature.

### 2. Query Expansion

Query expansion enriches the original patient queries with additional biomedical terms, including relevant disease and gene synonyms. The goal is to improve retrieval by accounting for variations in biomedical terminology.

### 3. Weighted Query Boosting

Query terms are assigned different weights according to their importance:

| Query component | Weight |
|---|---:|
| Disease and gene terms | 3× |
| Treatment terms | 2× |
| Demographic terms | 1× |

This weighting strategy prioritizes disease- and gene-related information while retaining treatment and patient demographic context.

### 4. BM25 Retrieval

The initial candidate documents are retrieved using **BM25Okapi**, a probabilistic ranking algorithm implemented in the `rank_bm25` library.

BM25 scores documents based on query-term frequency, inverse document frequency, and document length normalization. The resulting ranked candidates are passed to the deep learning reranking stage.

## Phase 2: Deep Learning Reranking

Two complementary neural models capture different aspects of query-document relevance.

### 1. BiGRU-Att: Treatment-Focus Classification

The **Bidirectional Gated Recurrent Unit with Attention (BiGRU-Att)** model captures contextual information from text in both forward and backward directions.

The attention mechanism helps the model focus on relevant words and phrases associated with treatment information.

**Objective:** Estimate the relevance of treatment-related information in candidate biomedical documents.

### 2. MatchPyramid: Disease Semantic Matching

The **MatchPyramid** model performs query-document relevance matching using a two-dimensional interaction matrix and a convolutional neural network (CNN).

The interaction matrix is constructed using dot-product similarities between query and document embeddings. The CNN learns local and higher-level matching patterns from this matrix.

**Objective:** Capture semantic matching patterns between disease-related query information and candidate documents.

### 3. Logistic Regression Ensemble

The outputs from BM25, BiGRU-Att, and MatchPyramid are combined using a logistic regression ensemble.

The ensemble is trained using relevance labels derived from TREC qrels (query relevance judgments).

The combined features are used to estimate document relevance and produce the final ranked list.

## Evaluation

The system is evaluated on three structured patient queries using a small PubMed-style corpus containing 20 documents with relevance annotations.

The evaluation uses standard information retrieval metrics:

| Metric | Description |
|---|---|
| P@5 | Precision among the top 5 retrieved documents |
| P@10 | Precision among the top 10 retrieved documents |
| NDCG@5 | Normalized Discounted Cumulative Gain at rank 5 |
| NDCG@10 | Normalized Discounted Cumulative Gain at rank 10 |

These metrics assess both the relevance of retrieved documents and the quality of their ranking.

> **Note:** The 20-document corpus and three-query evaluation represent a small-scale implementation. Results should not be interpreted as evidence of full-scale performance on the complete TREC 2019 collection without further evaluation.

## Ablation Study

An ablation study examines the contribution of individual pipeline components by comparing the baseline system with progressively enhanced configurations.

The components investigated include:

1. BM25-only baseline.
2. BM25 with query expansion.
3. BM25 with weighted query boosting.
4. BM25 combined with treatment-focus classification.
5. BM25 combined with disease semantic matching.
6. Full ensemble combining BM25, BiGRU-Att, MatchPyramid, and logistic regression.

The comparison is intended to identify how query enrichment and neural reranking influence retrieval performance.

## Key Findings

The implemented evaluation reports improvements over the BM25-only baseline after adding the pipeline components.

The findings suggest that combining traditional lexical retrieval with complementary deep learning models can improve biomedical document ranking in the experimental setting.

The ensemble approach integrates:

- **Lexical relevance:** BM25-based document matching.
- **Treatment relevance:** BiGRU-Att classification.
- **Disease relevance:** MatchPyramid semantic matching.
- **Combined relevance estimation:** Logistic regression ensemble.

The reported improvements apply to the evaluated sample corpus and should be validated on larger, more representative datasets before drawing broader conclusions.

## Technologies and Libraries

The pipeline uses or is designed around the following technologies:

- **Python** — implementation language.
- **rank_bm25** — BM25-based initial retrieval.
- **NLTK** — text preprocessing and Porter stemming.
- **CTD/NCBI gene synonym resources** — biomedical entity normalization.
- **PyTorch or an equivalent deep learning framework** — neural model implementation.
- **Scikit-learn** — logistic regression and evaluation utilities.
- **TREC qrels** — relevance judgments for training and evaluation.

*The exact dependencies should be adjusted to match the libraries used in the notebook.*

## Project Workflow

1. Load the biomedical document corpus and structured patient queries.
2. Normalize query and document text.
3. Resolve biomedical entities using the local synonym database.
4. Expand queries and apply term-weighting strategies.
5. Build the BM25 index and retrieve candidate documents.
6. Generate treatment-focus scores using BiGRU-Att.
7. Generate disease-matching scores using MatchPyramid.
8. Combine retrieval and neural features using logistic regression.
9. Rank documents using the ensemble scores.
10. Evaluate the results using P@5, P@10, NDCG@5, and NDCG@10.
11. Conduct ablation experiments to compare system configurations.

## Limitations

- The experimental corpus contains only 20 documents and three patient queries.
- Performance may vary with different query formulations, biomedical entities, and document collections.
- Neural reranking quality depends on training data, model configuration, and relevance-label quality.
- Improvements observed in the sample corpus may not generalize to the full TREC collection.
- Reproducibility requires the relevant datasets, synonym resources, qrels, preprocessing configuration, and trained model artifacts.

## Future Enhancements

- Evaluate on a larger portion of the TREC 2019 Precision Medicine Track collection.
- Incorporate biomedical language models such as BioBERT or PubMedBERT.
- Compare additional ranking approaches, including learning-to-rank models.
- Perform statistical significance testing across queries.
- Investigate query-level performance and error cases.
- Add reproducible configuration files, automated evaluation, and experiment tracking.

## References

- TREC 2019 Precision Medicine Track — [TREC](https://trec.nist.gov/)
- Liu et al. — Research describing the multi-stage biomedical retrieval and reranking approach that inspired this implementation.
- BM25 — Robertson and Zaragoza, *The Probabilistic Relevance Framework: BM25 and Beyond*.
- MatchPyramid — Pang et al., *Text Matching as Image Recognition*.

---

**Disclaimer:** This is an experimental biomedical information retrieval implementation for research and educational purposes. It is not a clinically validated decision-support system.
