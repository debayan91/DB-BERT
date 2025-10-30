
# DB-BERT: Predicting SQL Query Difficulty from Natural Language

This repository contains the code and paper for **DB-BERT**, a Transformer-based model for predicting SQL query difficulty directly from a user's natural language question, prior to query generation.

The full experiment is detailed in `DB_BERT.ipynb` and the accompanying paper, `db_bert_paper_article.pdf`.

## The Problem

Natural Language Interfaces (NLIs) to databases are powerful, but they treat all user questions as equal. A system has no way to distinguish between a simple request and a highly complex analytical query *before* execution.

* **Simple Query:** "List all employees in department X"

* **Complex Query:** "For each department, find the monthly average salary, sorted by departments with the highest variance"

In a production environment, this is dangerous. A single complex query can monopolize resources, create bottlenecks, and degrade system performance for all users.

## Our Approach

This work frames the problem as a **4-class text classification task**. The model reads the user's question (e.g., "How many heads of the departments are older than 56?") and predicts one of four difficulty labels (Easy, Medium, Hard, Extra Hard).

This prediction can then be used to:

* **Route queries:** Send "Easy" queries to a live OLTP database but route "Hard" queries to an OLAP replica.

* **Manage resources:** Proactively allocate more memory or time to complex queries.

* **Improve UX:** Instantly warn the user, "This is a complex query and may take a few moments."

### Data and Labeling

We use the **Spider 1.0 dataset** (`train_spider.json`, `train_others.json`, and `dev.json`). Since the dataset does not include difficulty labels, we generate them using a systematic set of rules based on the syntax of the ground-truth SQL query.

* **Easy (0):** Simple `SELECT-FROM-WHERE` queries.

* **Medium (1):** Contains `JOIN`, `GROUP BY`, `ORDER BY`, or `LIMIT`.

* **Hard (2):** Contains a subquery (2 `SELECT`s) or a `HAVING` clause.

* **Extra Hard (3):** Contains >2 `SELECT`s or a set operation (`UNION`, `INTERSECT`, etc.).

### Models

We compare two models:

1. **Baseline (TF-IDF + SVM):** A classical machine learning model using TF-IDF features (unigrams and bigrams) fed into a Support Vector Machine.

2. **DB-BERT (Ours):** A `bert-base-uncased` model, fine-tuned on the natural language questions and our generated difficulty labels.

## Results

The fine-tuned Transformer model significantly outperforms the classical baseline, proving that deep semantic understanding is more effective at capturing query complexity than simple keyword matching.

All results are from the `dev.json` (1,034 samples) test set.

| Model | Accuracy | Macro F1-Score | 
 | ----- | ----- | ----- | 
| TF-IDF + SVM | 76.89% | 74.48% | 
| **DB-BERT (Ours)** | **81.62%** | **81.02%** | 
| **Improvement** | **+4.73%** | **+6.54%** | 

## How to Run This Experiment

This entire experiment is contained in the `DB_BERT.ipynb` Jupyter Notebook and can be reproduced in Google Colab.

### 1. Environment

* A Google Colab environment with a **T4 GPU** is recommended.

### 2. Install Dependencies

Run the first cell in the notebook to install all required libraries:

```

\!pip install gdown --quiet
\!pip install --upgrade transformers datasets scikit-learn pandas pyarrow --quiet

```

**Important:** After running the install cell, you **must restart the Colab runtime** (`Runtime -> Restart runtime`) for the new libraries to load correctly.

### 3. Run All Cells

After restarting, run all cells in the notebook from top to bottom. The script will:

1. Download the Spider dataset from Google Drive.

2. Load and prepare the `train_spider.json`, `train_others.json`, and `dev.json` files.

3. Generate the 4-class difficulty labels.

4. Train and evaluate the **TF-IDF + SVM** baseline.

5. Train and evaluate the **DB-BERT** model (this will take \~10-15 minutes on a T4 GPU).

6. Print the final comparison metrics.

## Paper and Citation

A full write-up of this experiment, including methodology and discussion, is available in `db_bert_paper_article.pdf`.

If you use this work in your research, please cite it. Once you have created your Zenodo DOI, replace the `YOUR_DOI_HERE` placeholders.

```

@misc{dutta\_2025\_dbbert,
author       = {Dutta, Debayan},
title        = {{DB-BERT: A Transformer-Based Approach for Predicting SQL Query Difficulty from Natural Language Questions}},
month        = oct,
year         = 2025,
publisher    = {Zenodo},
version      = {v1.0.0},
doi          = {10.5281/zenodo.YOUR\_DOI\_HERE},
url          = {[https://doi.org/10.5281/zenodo.YOUR\_DOI\_HERE](https://www.google.com/search?q=https://doi.org/10.5281/zenodo.YOUR_DOI_HERE)}
}

```

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
