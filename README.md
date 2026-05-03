# PariShiksha Study Assistant
### Production-Grade RAG Study Assistant for NCERT Science
**Week 10 Mini-Project | PG Diploma in AI-ML & Agentic AI Engineering | Cohort 1**

---

## Overview

PariShiksha is a bounded RAG-based study assistant that answers NCERT Class 9 Science
questions reliably, grounded strictly in textbook content. Built for Tier-2/3 city
students where tutor availability is limited and trust in correct answers is critical.

**v2.0 (Wk10)** adds dense retrieval, strict grounded generation with citations,
honest 12-question evaluation on three axes, and a documented fix iteration.

**v1.0 (Wk9)** built the retrieval-ready foundation: PDF extraction, tokenizer
comparison, BM25 retrieval, and initial grounding prompt.

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
venv\\Scripts\\activate
# Mac/Linux:
source venv/bin/activate
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file in project root:
GROQ_API_KEY=your_groq_key_here
GEMINI_API_KEY=your_gemini_key_here

Get Groq API key free at: https://console.groq.com/keys

---

## Run

```bash
jupyter notebook notebook.ipynb
```

Run all cells top to bottom. Chroma collection persists to `./chroma_wk10` —
embeddings will not re-run on subsequent restarts.

---

## Project Structure
parishiksha_study_assistant/
├── data/
│   ├── raw/                    <- PDFs go here (NOT committed)
│   └── processed/
│       ├── chunks.json         <- Wk9 chunks
│       └── wk10_chunks.json    <- Wk10 token-aware chunks with metadata
├── chroma_wk10/                <- Persisted Chroma vector store (NOT committed)
├── evaluation/
│   ├── eval_raw.csv            <- Raw ask() output for all 12 questions
│   ├── eval_scored.csv         <- v1 hand-scored on 3 axes
│   ├── eval_v2_raw.csv         <- Raw output after fix
│   └── eval_v2_scored.csv      <- v2 hand-scored, honest delta
├── notebook.ipynb              <- Main notebook, runs end-to-end
├── README.md
├── reflection.md               <- Wk10 reflection questionnaire
├── chunking_diff.md            <- Wk9 vs Wk10 chunking comparison
├── retrieval_log.json          <- 10-question dense retrieval log
├── retrieval_misses.md         <- 3 miss diagnoses
├── prompt_diff.md              <- Permissive vs strict prompt comparison
├── fix_memo.md                 <- Stage 5 fix attempt with honest delta
├── requirements.txt
└── .gitignore

---

## Model Details

| Component | Choice | Reason |
|-----------|--------|--------|
| LLM | Llama-3.3-70b-versatile via Groq | Free tier, fast, strong instruction following |
| Embeddings | Gemini gemini-embedding-001 | Free tier, 768-dim, good on scientific text |
| Vector store | Chroma (local persistent) | No API cost, persistent across restarts |
| BM25 | rank_bm25 BM25Okapi | Lexical baseline, strong on exact physics terms |
| Tokenizers compared | GPT-2 BPE, BERT WordPiece, T5 SentencePiece | Stage 1 comparison |

---

## Evaluation Results — v1 vs v2

| Metric | v1 (baseline) | v2 (after fix) | Delta |
|--------|--------------|----------------|-------|
| Correct | 8/9 | 7/9 | -1 (regression) |
| Partial | 1/9 | 1/9 | 0 |
| Grounded | 9/9 | 8/9 | -1 |
| OOS refused | 3/3 | 3/3 | 0 |

Fix attempted: junk-chunk filter (drop non-worked_example chunks < 50 words).
Result: negative delta. Q1 regressed because filter removed a supporting chunk.
Honest documentation in `fix_memo.md`.

---

## Key Design Decisions

**Chunking:** 250-token chunks (tiktoken cl100k_base), 50-token overlap.
Worked examples kept whole. End-of-chapter questions split individually.
Junk artifact chunks (graph labels, cross-refs < 30 words) filtered at index time.

**Grounding:** Strict prompt with exact refusal string and enforced citation format
`[Source: wk10_XXXX]`. Permissive vs strict comparison documented in `prompt_diff.md`.

**Retrieval:** Dense (Gemini embeddings + Chroma cosine) as primary. BM25 available
as fallback. Hybrid not implemented — documented as Wk11 target.

**Temperature:** 0.0 for all evaluation runs for reproducibility.

**Citation:** Every ask() call returns `{answer, sources, chunk_ids}`. Wrong answers
diagnosable in 30 seconds by checking cited chunk content.

---

## What This Project Does NOT Do

- No GPU required — CPU sufficient
- No reranking — documented as next priority in fix_memo.md
- No hybrid retrieval — BM25 + dense fusion is Wk11 target
- No Hindi language support — documented as pilot readiness gap
- Not ready for 100-student pilot — 3 specific gaps in reflection.md D3

---

## Loom Walkthrough

[Link to be added before submission]

Shows: ask() running on 3 queries (in-scope / paraphrased / out-of-scope refused),
eval table, and fix delta.

---

## Team

| Name | GitHub | Contribution |
|------|--------|-------------|
| Shanmuk Yadav | [@ShanmukYadav](https://github.com/ShanmukYadav) | Chunking, retrieval, generation, evaluation, fix iteration |
| Ankit Prashar | [@ankit-2244](https://github.com/ankit-2244) | Reflection, documentation, review |

---

## Submission

- **Deadline:** Sunday 03-05-2026, 11:00 PM IST
- **Cohort:** 1
- **Track:** Core
- **Project:** Week 10 — Study Assistant v2.0
"""

with open("README.md", "w", encoding="utf-8") as f:
    f.write(readme)

print("README.md written ✓")
