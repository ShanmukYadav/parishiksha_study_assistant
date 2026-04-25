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
- Ch 8: https://ncert.nic.in/textbook/pdf/iesc108.pdf
- Ch 9: https://ncert.nic.in/textbook/pdf/iesc109.pdf

Place in `data/raw/`.

## Corpus Source

NCERT Official: https://ncert.nic.in/textbook.php?iesc1=0-11

## Environment

- Python 3.10+
- Google gemini API ()
EOF