# 🎓 Exam Prep — Multi-Agent Assistant

A multi-agent study assistant that turns a lecture file (PDF or PPTX) into a complete exam-preparation kit: summaries, a glossary, concept comparisons, paper recommendations, adaptive practice questions, a mastery dashboard, a revision plan, and a downloadable PDF report. Everything runs through a Gradio web app powered by a locally loaded LLM.

---

## ✨ Features

| Tab | What it does |
|---|---|
| 📝 **Summary** | Summarizes the lecture page by page, validates keyword coverage, and retries if coverage is too low. Exports `summary.md`. |
| 📖 **Glossary** | Explains key terms in 1–2 sentences using *only* the lecture content, and flags low-grounding answers. |
| ⚖️ **Comparison** | Builds a comparison table between concepts based on criteria you choose (e.g. definition, purpose, example). |
| 🔎 **Resources** | Finds related research papers on arXiv for each keyword. |
| ❓ **Practice** | Generates cloze and multiple-choice questions, runs up to 3 adaptive rounds that focus on your weak topics. |
| 📅 **Plan & Dashboard** | Shows topic mastery chart and builds a day-by-day revision plan up to your exam date. |
| 📄 **Report** | Exports everything into a single `study_report.pdf`. |

---

## 🧠 Agents Architecture

The system is split into specialized agents, each with one job:

- **Retriever Agent**: semantic search over the lecture pages (FAISS + sentence embeddings).
- **Summarizer Agent**: writes exam-oriented summaries for groups of pages.
- **Validator Agent**: checks that important terms from the source appear in the summary and returns a coverage score plus the missing terms.
- **Supervisor Agent**: coordinates the Summarizer and Validator, re-running the summarizer (up to `MAX_RETRIES`) with the missing terms until coverage reaches `COVERAGE_THRESHOLD`.
- **Keyword Explainer Agent**: retrieves context and explains a term, with a grounding score to detect outside-knowledge answers.
- **Comparison Agent**: fills a comparison table using the retriever for each concept/criterion pair.
- **Resource Finder Agent**: queries the arXiv API for relevant papers.
- **Question Generator Agent**: creates cloze (fill-in-the-blank) questions from the text and MCQs from the LLM, filtering out MCQs whose correct answer is not supported by the lecture.
- **Answer Checker Agent**: grades answers and optionally explains mistakes using the lecture text.
- **Weak-Spot Tracker Agent**: computes per-topic weakness/mastery and picks topics that need more practice.
- **Planner Agent**: distributes weak topics across the days left before the exam.

```
 Lecture (PDF / PPTX)
        │
   load_lecture ──► FAISS vector store ◄──── Retriever Agent
        │                                        ▲
        ├─► Supervisor ⇄ Summarizer / Validator  │
        ├─► Keyword Explainer ───────────────────┤
        ├─► Comparison Agent ────────────────────┘
        ├─► Resource Finder (arXiv)
        └─► Question Generator ─► Answer Checker ─► Weak-Spot Tracker ─► Planner
                                                              │
                                              Dashboard + PDF Report
```

---

## 🛠️ Tech Stack

- **LLM:** [`Qwen/Qwen2.5-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct), loaded in 4-bit with `bitsandbytes`
- **Embeddings:** `sentence-transformers/all-MiniLM-L6-v2`
- **Vector store:** FAISS via LangChain
- **Document parsing:** PyMuPDF (PDF), python-pptx (PPTX, including speaker notes)
- **UI:** Gradio
- **Charts / data:** Matplotlib, pandas
- **PDF export:** ReportLab
- **External API:** arXiv

---

## ⚙️ Requirements

- Python 3.9+
- A **CUDA GPU** (the 7B model is loaded in 4-bit; a free Google Colab T4 works)
- Internet access (to download the models and query arXiv)

Install dependencies:

```bash
pip install -q gradio langchain-community langchain-core faiss-cpu sentence-transformers \
    pymupdf python-pptx transformers accelerate bitsandbytes reportlab
```

---

## 🚀 How to Run

1. Open `Final_project_Sarah_Mahmoud_fathy.ipynb` in Google Colab (or Jupyter with a GPU).
2. Run all cells from top to bottom. The first run downloads the model, so it takes a few minutes.
3. The last cell launches the app:
   ```python
   demo.launch(share=True, debug=True)
   ```
4. Open the generated link (a public `gradio.live` URL is created because `share=True`).

---

## 📖 How to Use

1. **Upload** a lecture (`.pdf` or `.pptx`) and click **Load lecture**. Pages with fewer than 30 characters are skipped, and suggested keywords are extracted automatically. You can edit the keywords box.
2. Use the tabs in any order:
   - **Summary**: click *Generate summary*, then download `summary.md`.
   - **Glossary**: click *Generate glossary* for the keywords in the box.
   - **Comparison**: enter at least 2 concepts and 1 criterion (comma separated).
   - **Resources**: find related arXiv papers (a 3-second delay between requests respects the API).
   - **Practice**: generate the question pool, then press *Start / Next round*, answer, and submit. Rounds 2–3 focus on your weak topics.
   - **Plan & Dashboard**: enter the exam date as `YYYY-MM-DD` to see mastery and a revision plan.
   - **Report**: build and download the full `study_report.pdf`.

> 💡 Do some practice rounds **before** opening the dashboard or planner, since they depend on your answers.

---

## 🔧 Configuration

Defined in the *Global State* cell:

| Constant | Default | Meaning |
|---|---|---|
| `WEAK_THRESHOLD` | `0.5` | Weakness score at or above which a topic is considered weak |
| `COVERAGE_THRESHOLD` | `0.7` | Minimum keyword coverage for a summary to be accepted |
| `MAX_RETRIES` | `2` | Summary regeneration attempts per section |
| `MAX_ROUNDS` | `3` | Number of practice rounds |
| `MAX_BATCH` | `8` | Max questions shown in one round |

---

## 📁 Outputs

| File | Description |
|---|---|
| `summary.md` | Page-by-page lecture summary |
| `study_report.pdf` | Full report: summary, glossary, comparison, resources, missed questions, dashboard, revision plan |

---


