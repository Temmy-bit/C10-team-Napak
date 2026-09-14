# C10 Team Napak

An agricultural information-retrieval and RAG project built around a farmer-focused retrieval benchmark. The repository uses the provided agricultural extension corpus to perform document retrieval, semantic search, and answer generation for farmer questions related to crops, soil, climate stress, and farm management.

## Project overview

This project tackles a document-ranking task inspired by IR and RAG benchmarks:

- Given a farmer question, retrieve the most relevant agricultural-extension documents.
- Rank candidate documents by relevance using a retrieval system.
- Optionally use the retrieved context to answer the question in natural language.
- Evaluate retrieval quality using a benchmark-style setup and shared submission format.

The dataset is centered on real-world agricultural advice and includes:

- a corpus of extension-style documents,
- training queries with graded relevance labels,
- test queries without labels,
- a baseline retrieval submission,
- a persisted vector database for semantic retrieval.

## Why this project matters

Many farmers and extension workers need fast, reliable answers to practical agronomic questions. This project models that need as a retrieval problem, where the system must identify the most relevant document(s) before generating an answer. The approach is useful for:

- crop disease diagnosis,
- climate resilience guidance,
- soil and nutrient issues,
- general agronomy support.

## Data and benchmark

The project uses the agricultural extension retrieval dataset stored in the `data/` folder.

### Included datasets

- `data/documents.csv` — corpus of knowledge-base documents
- `data/train_queries.csv` — training questions and positive document references
- `data/test_queries.csv` — evaluation queries used for ranking submissions
- `data/qrels_train.csv` — graded relevance labels for training queries
- `data/sample_submission.csv` — expected submission format
- `data/baseline_submission.csv` — baseline ranking output
- `data/dataset-metadata.json` — dataset metadata and schema information
- `data/vector_store/` — persisted Chroma vector database

### Dataset characteristics

The benchmark is designed for ranking and retrieval tasks, with:

- graded relevance values (3, 2, 1, 0),
- multiple relevant documents per query,
- hard negatives,
- a nDCG@5 evaluation target,
- agricultural topics across crops, soils, climate, and plant health.

## Repository structure

```text
C10-team-Napak/
├── data/
│   ├── baseline_submission.csv
│   ├── dataset-metadata.json
│   ├── documents.csv
│   ├── qrels_train.csv
│   ├── sample_submission.csv
│   ├── test_queries.csv
│   ├── train_queries.csv
│   └── vector_store/
├── docs/
│   ├── data_card.pdf
│   ├── impact-statement.pdf
│   ├── problem_statement.pdf
│   └── stakeholder_engagement.pdf
├── scripts/
│   └── test.ipynb
├── .gitignore
├── pyproject.toml
├── README.md
└── .venv/ (if created locally)
```

## Core workflow in this repository

The notebook in `scripts/test.ipynb` demonstrates the main pipeline used in the project.

### 1. Baseline retrieval

The first section builds a TF-IDF baseline using:

- `TfidfVectorizer`
- cosine similarity
- top-5 document retrieval per query

This creates a simple search baseline that is easy to compare against more advanced retrieval methods.

### 2. Document loading and chunking

The notebook loads the document corpus with LangChain CSV loading utilities and then splits each document into smaller chunks using a recursive text splitter.

This is important because:

- large documents can exceed model limits,
- chunking improves retrieval granularity,
- smaller chunks increase the chance of matching relevant context precisely.

### 3. Embeddings and vector store

The project then creates embeddings using a SentenceTransformer model such as:

- `all-MiniLM-L6-v2`

These embeddings are stored in a Chroma collection under `data/vector_store/`, enabling semantic retrieval for query similarity search.

The retrieval layer uses:

- sentence embeddings,
- Chroma persistent client,
- similarity scoring,
- top-k filtering and ranking.

### 4. RAG-style answer generation

The notebook also includes an advanced retrieval-and-answer flow that:

- retrieves relevant chunks or documents for a query,
- assembles context from the retrieved sources,
- passes the context to an LLM prompt,
- returns an answer with citations and a brief summary.

This is a lightweight RAG pattern suitable for experimentation and benchmarking.

## Setup

### Prerequisites

- Python 3.12+
- pip or uv
- access to the local project workspace

### Recommended environment setup

Using `uv`:

```bash
uv sync
```

If needed, install the core libraries manually:

```bash
pip install pandas scikit-learn langchain langchain-community langchain-text-splitters sentence-transformers chromadb
```

Note: the current `pyproject.toml` is intentionally minimal, so the notebook itself may install or use additional packages as needed during execution.

## Running the project

### Open the notebook

From the project root:

```bash
jupyter lab
```

Then open:

```text
scripts/test.ipynb
```

### Typical workflow

1. Load the corpus from `data/documents.csv`.
2. Run the TF-IDF baseline to generate a quick ranking.
3. Chunk documents and embed them for semantic retrieval.
4. Persist the Chroma vector store.
5. Query the vector store for top-k relevant documents.
6. Build a final submission CSV for benchmarking.

## Submission format

The benchmark expects rows in a long format, with one document per query rank:

```csv
QueryId,DocumentId
1001,123
1001,456
1001,789
```

The notebook generates outputs like:

- `submission.csv`
- `submission2.csv`
- `submission_advrag_top5.csv`

These outputs are structured as ranked submissions for the contest-style retrieval evaluation.

## Evaluation metric

The problem framing and dataset metadata indicate the leaderboard metric is:

- nDCG@5

This metric rewards retrieval systems that place the most relevant documents near the top of the ranking.

## Key technical dependencies

The project relies primarily on:

- `pandas` for data loading and tabular processing,
- `scikit-learn` for TF-IDF and cosine similarity,
- `langchain` and `langchain-community` for CSV loading and document processing,
- `langchain-text-splitters` for chunking,
- `sentence-transformers` for embeddings,
- `chromadb` for persistent vector search,
- optional LLM integration for RAG answer generation.

## Notes for contributors

- Keep the retrieval experiments reproducible and documented in the notebook.
- When iterating on retrieval quality, compare against the TF-IDF baseline first.
- Track changes in chunk size, embedding model, and top-k values, as these often strongly affect rankings.
- Use the persisted Chroma store in `data/vector_store/` to avoid rebuilding embeddings unnecessarily.

## Recommended next steps

Potential extensions for this project include:

- testing multiple sentence embedding models,
- adding Hybrid retrieval with BM25 + vector search,
- evaluating model performance on the training queries,
- building a reusable Python pipeline outside the notebook,
- packaging the project into a CLI or module-based app.

## License and project context

This repository uses a benchmarking dataset for agricultural retrieval and RAG experimentation. The project is intended for research, experimentation, and educational use within the contest context described by the provided metadata and documentation files in the `docs/` directory.

## Summary

This project combines a real agricultural retrieval benchmark with a practical RAG pipeline. It demonstrates how to:

- load and process domain-specific document data,
- create a retrieval baseline,
- build semantic search with embeddings,
- answer farmer questions using retrieved context,
- export ranked outputs suitable for evaluation.

---

If you want, this README can also be expanded into a more polished competition-style version with badges, usage examples, and a section for team members and model benchmarks.
