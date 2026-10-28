# Hotel QnA — Deployable RAG Application

**A deployable Retrieval-Augmented Generation app that lets hotel staff ask natural-language policy questions and get source-grounded answers, with a confidence score that routes low-confidence answers to manual lookup.**

> **Note on this repo:** unlike a notebook-only analysis, the deliverable here is a **running application** (FastAPI/Flask backend + Streamlit frontend, containerized with Docker). There is no static HTML report / GitHub Pages site — the value is the app and its architecture. The accompanying notebook documents how the RAG pipeline was built and evaluated.

---

## Problem

A mid-sized hotel group (18 properties, growing) answers ~1,200 guest queries/week. About 40% are complex, multi-document lookups (booking terms, cancellation, refunds, room info, guidelines) that take **8–12 minutes each** and carry a **15–20% error rate** because answers are pieced together manually. With ~45% annual staff turnover, institutional knowledge is constantly lost.

## Objective

Build a **proof-of-concept internal app** where staff enter a question and receive an answer grounded **only** in the hotel's documents, with the supporting sources shown. The goal is to reduce the number of queries that require manual lookup or escalation — validating reliability before any broader rollout.

## Approach

- **Document ingestion:** parse policy PDFs, chunk, and embed into a **FAISS** vector index.
- **Retrieval + grounded generation:** retrieve the most relevant chunks and generate an answer constrained to that context, returning the cited sources.
- **Confidence routing:** score each answer on a composite of faithfulness, contextual recall/precision, and answer relevancy; **auto-answer** when the score clears the threshold, otherwise fall back to **manual lookup**.
- **Deployment:** a backend service and a Streamlit frontend, each with a `Dockerfile`, designed to run in GitHub Codespaces or any container host.

## Success criteria (defined in the project)

| Business objective | Metric | Threshold |
|---|---|---:|
| Avoid unsupported information | Faithfulness | ≥ 0.85 |
| Retrieve all relevant info | Contextual Recall | ≥ 0.80 |
| Retrieve relevant chunks | Contextual Precision | ≥ 0.75 |
| Answer the question asked | Answer Relevancy | ≥ 0.75 |
| Reduce manual lookups | Queries confidently answered | ≥ 50% |

Composite score = 0.35·Faithfulness + 0.25·Recall + 0.20·Precision + 0.20·Relevancy; answers scoring ≥ 0.75 are auto-answered, below are routed to manual lookup.

## Data

Course-provided AuroraStay policy documents (synthetic, **no personal data**):

- `data/aurorastay_booking_terms_and_cancellation_policy.pdf`
- `data/aurorastay_refunds_and_additional_charges_guide.pdf`
- `data/aurorastay_room_and_property_information.pdf`
- `data/aurorastay_guest_conduct_accessibility_and_resolution_guidelines.pdf`
- `data/evaluation.csv` — evaluation question set

## Technologies

Python · LangChain · **FAISS** · OpenAI / HuggingFace embeddings · **Flask** (backend) · **Streamlit** (frontend) · **Docker** · DeepEval · Docling (PDF parsing) · Jupyter

## Running the app

> **Requires an OpenAI API key.** Provide it via environment variables or a `config.json` created from `config.example.json` (both git-ignored). The FAISS index is rebuilt from `data/` (not committed).

**Backend**
```bash
cd backend
pip install -r requirements.txt
export OPENAI_API_KEY="sk-..."
python app.py        # or: docker build -t hotel-qna-backend . && docker run -p 8000:8000 hotel-qna-backend
```

**Frontend**
```bash
cd frontend
pip install -r requirements.txt
streamlit run app.py   # or build/run the Docker image
```

The [notebook](notebooks/hotel_qna_rag.ipynb) walks through ingestion, retrieval, generation, and the evaluation that informed the thresholds above.

## Repository layout

```text
hotel-qna-rag-app/
├── README.md
├── backend/                   # RAG service (app.py, rag_utils.py, Dockerfile, requirements.txt)
├── frontend/                  # Streamlit UI (app.py, Dockerfile, requirements.txt)
├── notebooks/
│   └── hotel_qna_rag.ipynb    # build + evaluation walkthrough
├── data/                      # course-provided policy PDFs + evaluation set
├── config.example.json        # template — copy to config.json and add your key
├── requirements.txt
├── .gitattributes
└── .gitignore
```

## Limitations & potential improvements

- Proof-of-concept, internal-facing; not hardened for production (no auth, rate limiting, or observability).
- FAISS index and evaluation are built on a small, static document set.
- Requires hosted LLM/embedding APIs; offline/self-hosted models would need adaptation.
- No CI/CD or managed deployment — containers are provided for manual/Codespaces deployment.

---

### About this project

Completed as a capstone project in the **Post Graduate Program in Artificial Intelligence & Machine Learning (UT Austin / Great Learning)**. Shared to demonstrate building and deploying a RAG application. The scenario and documents are course-provided, synthetic academic material; metrics are proof-of-concept targets and are **not** claims of production deployment or commercial outcomes. No API keys or credentials are included in this repository.
