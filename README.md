cat > README.md << 'EOF'
# PariShiksha Study Assistant

A retrieval-augmented generation (RAG) system for NCERT Class 9 Science,
built as Week 9 Mini-Project for PG Diploma in AI-ML & Agentic AI Engineering.

## Setup

```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Run

Open `notebook.ipynb` in Jupyter and run all cells top to bottom.

## Data

PDFs are NOT committed. Download from:
- motion around us: https://ncert.nic.in/textbook/pdf/iesc104.pdf
- How forces effect motion: https://ncert.nic.in/textbook/pdf/iesc106.pdf

Place in `data/raw/`.

## Corpus Source

NCERT Official: https://ncert.nic.in/textbook.php?iesc1=0-11

## Environment

- Python 3.10+
- Google gemini API () for testing 
- Groq API (llama model)

## Model Details

- **LLM**: Llama-3.1-8b-instant via Groq API (free tier)
- **Retriever**: BM25 (rank-bm25) with content-type boosting
- **Tokenizers compared**: GPT-2 BPE, BERT WordPiece, T5 SentencePiece
- **Embedding model**: sentence-transformers/all-MiniLM-L6-v2 (Advanced stage)

## Environment Variables

Create a `.env` file with:
```
GROQ_API_KEY=your_groq_key_here
GEMINI_API_KEY=your_gemini_key_here  
```
EOF
