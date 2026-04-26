# PariShiksha Study Assistant
### Retrieval-Ready Study Assistant for NCERT Science
**Week 9 Mini-Project | PG Diploma in AI-ML & Agentic AI Engineering | Cohort 2**

---

## Overview

PariShiksha is a bounded RAG-based study assistant that answers NCERT Class 9 Science questions reliably, grounded strictly in textbook content. Built for Tier-2/3 city students where tutor availability is limited and trust in correct answers is critical.

This week's project builds the retrieval-ready foundation:
- Extracts and structures NCERT chapter content
- Compares tokenization strategies
- Implements BM25 + Dense + Hybrid retrieval
- Generates grounded answers via LLM with explicit refusal for out-of-scope queries
- Evaluates honestly on 18 questions with failure analysis

---

## Corpus

**Source:** NCERT Official — https://ncert.nic.in/textbook.php?iesc1=0-11

| Chapter | Title | Pages | Chunks |
|---------|-------|-------|--------|
| Chapter 4 | Describing Motion Around Us | 24 | 50 |
| Chapter 6 | How Forces Affect Motion | 22 | 51 |

PDFs are NOT committed to this repository. Download directly from NCERT.

**Download links:**
- Ch4: https://ncert.nic.in/textbook/pdf/iesc104.pdf
- Ch6: https://ncert.nic.in/textbook/pdf/iesc106.pdf

Place both files in `data/raw/`.

---

## Setup

### Requirements
- Python 3.10+
- Fresh virtual environment

### Install

```bash
git clone https://github.com/ShanmukYadav/parishiksha_study_assistant.git
cd parishiksha_study_assistant
python -m venv venv
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file in project root:

```
GROQ_API_KEY=your_groq_key_here
GEMINI_API_KEY=your_gemini_key_here
```

Get Groq API key free at: https://console.groq.com/keys

---

## Run

```bash
jupyter notebook notebook.ipynb
```

Open `notebook.ipynb` and run all cells top to bottom.

---

## Project Structure

```
parishiksha_study_assistant/
├── data/
│   ├── raw/              <- PDFs go here (NOT committed)
│   └── processed/        <- chunked JSON data
├── evaluation/
│   ├── evaluation_results.md
│   └── evaluation_results.csv
├── notebook.ipynb        <- main notebook, runs end-to-end
├── README.md
├── reflection.md
├── failure_modes.md
├── requirements.txt
└── .gitignore
```

---

## Model Details

| Component | Choice | Reason |
|-----------|--------|--------|
| LLM | Llama-3.1-8b-instant via Groq | Free tier, fast, sufficient for grounded QA |
| Retriever | BM25 + Dense + Hybrid | All three compared and documented |
| Embeddings | all-MiniLM-L6-v2 | Lightweight, runs on CPU |
| Tokenizers compared | GPT-2 BPE, BERT WordPiece, T5 SentencePiece | Required by Stage 1 |

---

## Evaluation Results Summary

| Metric | Score |
|--------|-------|
| Total questions | 18 |
| Correct | 11/13 answered |
| Partial | 2/13 |
| Grounded | 13/13 |
| Refusals appropriate | 5/5 |

See `evaluation/evaluation_results.md` for full details and failure analysis.

---

## Key Design Decisions

**Chunking:** 300 token chunks with 50 token overlap. Worked examples kept whole to prevent problem-solution splits. End-of-chapter questions split individually.

**Grounding:** Explicit REFUSAL instruction in prompt. LLM must refuse if answer not in context. Tested on 5 out-of-scope questions, all correctly refused.

**Retrieval:** BM25 baseline then Dense (all-MiniLM-L6-v2) then Hybrid (alpha=0.5). All three compared on same queries. High-frequency physics terms reduce BM25 discriminability — documented in failure_modes.md.

**Temperature:** Set to 0.0 for all evaluation runs for reproducibility.

---

## What This Project Does NOT Do

- No GPU required — CPU sufficient
- No full vector database — that is Week 10
- No fine-tuned models — uses off-the-shelf embeddings
- No Hindi language support yet — documented as pilot readiness gap

---

## Team

| Name | Contribution |
|------|-------------|
| Shanmuk Yadav | Corpus extraction, chunking, BM25, Groq integration, evaluation, advanced retrieval |
| Teammate Name | Reflection, documentation, review |

---

## Submission

- **LMS Deadline:** Sunday(26-04-2026)
- **Cohort:** 2
- **Project:** Week 9 — Retrieval-Ready Study Assistant

## Teammates

| Name | GitHub |
|------|--------|
| Shanmuk Yadav | [@ShanmukYadav](https://github.com/ShanmukYadav) |
| Ankit Prashar| [@ankit-2244](https://github.com/ankit-2244) |
