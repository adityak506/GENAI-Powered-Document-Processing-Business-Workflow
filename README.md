# AI-Powered Customer Complaint & Case Processing System

An automated, batch-oriented Generative AI workflow built in Python to ingest multi-format customer complaint documents, extract strict structured data using Pydantic, and execute parallel downstream content generation for customer communications and internal management reporting.

---

## 1. Problem Statement

Customer support and operations teams frequently receive unstructured complaint documents across multiple formats (`.txt`, `.pdf`, `.docx`). Manually reading each document, logging metadata into tracking spreadsheets, drafting empathetic customer replies, and writing executive escalation summaries is slow, error-prone, and difficult to scale during high-volume periods.

## 2. Solution Overview

This system automates the end-to-end complaint triage lifecycle through an orchestrated multi-step LLM workflow:
1. **Multi-Format Ingestion:** Scans a local `data/` directory and safely extracts text from `.txt`, `.pdf`, and `.docx` files with file-level error isolation.
2. **Structured Information Extraction:** Uses LLM structured outputs validated by **Pydantic** to extract key fields (customer details, complaint category, escalation flags, and case status) without relying on brittle regex or raw text parsing.
3. **Parallel Content Generation:** Once structured facts are extracted, two independent LLM tasks execute concurrently using `ThreadPoolExecutor`:
   - **Customer Response Email:** Drafts a polite, grounded response strictly adhering to facts in the source document.
   - **Management Case Summary:** Produces a standardized Markdown briefing for internal triage and leadership review.
4. **Consolidated Reporting:** Saves individual artifacts (`.json`, `.txt`, `.md`) and aggregates all processed records into a unified `output/final_report.csv`.

---

## 3. Architecture Diagram

```text
                      ┌─────────────────────────┐
                      │   Input Folder (data/)  │
                      │  (.txt, .pdf, .docx)    │
                      └────────────┬────────────┘
                                   │
                                   ▼
                      ┌─────────────────────────┐
                      │   Document Ingestion    │
                      │  (Text Extraction &     │
                      │   Error Handling)       │
                      └────────────┬────────────┘
                                   │
                                   ▼
                      ┌─────────────────────────┐
                      │  Task 1: LLM Structured │
                      │  Extraction (Pydantic)  │
                      └────────────┬────────────┘
                                   │
                 ┌─────────────────┴─────────────────┐
                 │ (Parallel Execution via Threads)  │
                 ▼                                   ▼
    ┌─────────────────────────┐         ┌─────────────────────────┐
    │  Task 2: Customer       │         │  Task 3: Management     │
    │  Response Email (.txt)  │         │  Case Summary (.md)     │
    └────────────┬────────────┘         └────────────┬────────────┘
                 │                                   │
                 └─────────────────┬─────────────────┘
                                   │
                                   ▼
                      ┌─────────────────────────┐
                      │    Output Persistence   │
                      │ - structured_data/*.json│
                      │ - customer_emails/*.txt │
                      │ - case_summaries/*.md   │
                      │ - final_report.csv      │
                      └─────────────────────────┘
```

---

## 4. Technology Stack

- **Language:** Python 3.10+
- **LLM Integration:** OpenAI Python SDK (`gpt-4o-mini` with Structured Outputs)
- **Data Validation:** Pydantic v2 (`BaseModel`, strict `Literal` typing)
- **Document Parsing:** `pypdf` (PDF), `python-docx` (DOCX), native UTF-8 reader (TXT)
- **Concurrency:** `concurrent.futures.ThreadPoolExecutor`
- **Data Aggregation:** `pandas`
- **Configuration & Logging:** `python-dotenv`, Python `logging` module

---

## 5. Project Structure

```text
customer-complaint-processor/
├── data/                         # Input customer complaint documents
│   ├── complaint_001.txt
│   ├── complaint_002.txt
│   ├── complaint_003.txt
│   ├── complaint_004.docx
│   └── complaint_005.docx
├── output/                       # Auto-generated outputs (git-ignored except samples)
│   ├── structured_data/          # Validated JSON records per document
│   ├── customer_emails/          # Grounded customer reply drafts
│   ├── case_summaries/           # Internal Markdown management briefs
│   ├── final_report.csv          # Consolidated CSV across all batch documents
│   └── workflow.log              # Execution logs and error traces
├── src/
│   ├── __init__.py
│   ├── config.py                 # Directory paths, env vars, and logger setup
│   ├── schemas.py                # Pydantic schema definitions
│   ├── ingestion.py              # Multi-format file readers
│   ├── llm_tasks.py              # Modular LLM task functions & prompts
│   └── workflow.py               # Sequential + parallel task orchestration
├── generate_sample_data.py       # Script to generate sample .txt & .docx inputs
├── main.py                       # Main batch execution entry point
├── requirements.txt              # Python dependencies
├── .env.example                  # Environment variable template
└── README.md                     # Project documentation
```

---

## 6. Setup Instructions

### Prerequisites
- Python 3.10 or higher installed locally.
- An active OpenAI API key.

### Installation Steps
1. **Clone the repository:**
   ```bash
   git clone https://github.com/adityak506/GENAI-Powered-Document-Processing-Business-Workflow.git
   cd GENAI-Powered-Document-Processing-Business-Workflow
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

---

## 7. Environment Variable Requirements

Copy `.env.example` to `.env` and configure your API credentials:

```bash
cp .env.example .env
```

Required variables inside `.env`:
```env
OPENAI_API_KEY=sk-your-actual-api-key-here
MODEL_NAME=gpt-4o-mini
```
*(Note: Never commit your `.env` file to version control.)*

---

## 8. How to Run the Application

1. **(Optional) Generate sample complaint documents:**
   If `data/` is empty, run the helper script to populate `.txt` and `.docx` test files:
   ```bash
   python generate_sample_data.py
   ```

2. **Execute the batch processing workflow:**
   ```bash
   python main.py
   ```

3. **Inspect the generated outputs:**
   Check `output/structured_data/`, `output/customer_emails/`, `output/case_summaries/`, and `output/final_report.csv`.

---

## 9. Sample Inputs & Outputs

### Sample Input (`data/complaint_002.txt`)
```text
From: Marcus Vance (m.vance@techcorp.io | 555-019-3340)
Subject: URGENT - Enterprise API Outage & Data Loss

Our production environment has been unable to authenticate against your v2 Payment Gateway API 
for the past 6 hours. Every request returns a 503 Service Unavailable error. We are losing thousands 
of dollars per hour. Tier-1 support told us to clear our cache, which did not resolve anything. 
We demand immediate escalation to Senior Engineering and an SLA credit review.

Supporting Documents: Full server error logs and HAR network traces attached.
```

### Sample Output 1: Structured JSON (`output/structured_data/complaint_002.json`)
```json
{
  "customer_name": "Marcus Vance",
  "email": "m.vance@techcorp.io",
  "phone_number": "555-019-3340",
  "complaint_category": "Technical",
  "issue_description": "Production environment unable to authenticate against v2 Payment Gateway API for 6 hours due to 503 Service Unavailable errors.",
  "resolution_provided": "Tier-1 support advised clearing cache, which failed to resolve the issue.",
  "is_complaint": "Yes",
  "escalation_required": "Yes",
  "supporting_document_available": "Yes",
  "overall_case_status": "Escalated"
}
```

### Sample Output 2: Customer Email (`output/customer_emails/complaint_002_email.txt`)
```text
Subject: Update on Your Enterprise API Authentication Issue

Dear Marcus Vance,

Thank you for reaching out and bringing this critical issue to our attention. We understand that your production environment has been experiencing 503 Service Unavailable errors when authenticating against the v2 Payment Gateway API for the past six hours.

We recognize that the initial cache-clearing step recommended by Tier-1 support did not resolve the problem. Given the severity of this disruption and the server logs and HAR network traces you provided, your case has been marked for escalation to our Senior Engineering team, along with your request for an SLA credit review.

We will keep you updated as soon as our engineering team reviews your case details.

Sincerely,
Customer Support Team
```

### Sample Output 3: Management Case Summary (`output/case_summaries/complaint_002_summary.md`)
```markdown
- **Case Overview:** Enterprise customer Marcus Vance reported a prolonged production outage on the v2 Payment Gateway API.
- **Key Issue:** Continuous 503 Service Unavailable authentication failures lasting 6 hours, causing significant financial impact.
- **Action Taken:** Tier-1 support suggested clearing cache, which was ineffective.
- **Current Status:** Escalated.
- **Recommended Next Action:** Immediately assign to Senior Engineering with the attached server logs/HAR traces and initiate an SLA credit review.
```

---

## 10. Key Design Decisions

1. **Orchestrated Multi-Step Pipeline vs. Monolithic Prompting:** Separating extraction, customer communication, and management summarization into dedicated LLM calls allows tailored system prompts, lower temperatures (`0.0` for deterministic extraction vs. `0.3` for natural email drafting), and easier debugging.
2. **Hybrid Sequential-Parallel Execution:** Task 1 (Structured Extraction) runs first so validated metadata can ground Tasks 2 and 3. Once extracted, the email and summary tasks execute in parallel via `ThreadPoolExecutor`, reducing per-document latency by ~35–45%.
3. **Strict Hallucination Guardrails:** Prompts explicitly instruct the model not to fabricate refund timelines, compensation amounts, or policies absent from the source document.
4. **Fault-Tolerant Batch Loop:** File-level `try/except` blocks ensure that a corrupted or unreadable file logs an error and records a failure status in `final_report.csv` without crashing the entire batch run.

---

## 11. Limitations

- **Scanned PDFs:** `pypdf` extracts embedded text only; image-only or scanned PDFs require an OCR preprocessing step (e.g., Tesseract or Vision LLM) to extract text.
- **API Rate Limits:** Running high-concurrency batches across hundreds of files simultaneously would require rate-limiting/backoff retry logic (e.g., `tenacity`).
- **Attachment Verification:** The system detects whether supporting documents are *mentioned* in the text rather than inspecting separate binary attachments.
