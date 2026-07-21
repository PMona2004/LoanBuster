# LoanLens 🔍
### AI Predatory Lending Decoder for Indian Borrowers
**Google Solution Challenge 2026 · Track: Unbiased AI Decision**

---

> *47 crore Indians take microloans. A loan advertised at "2% per month" is often 300–700% APR once all fees are included. Borrowers have zero tools to compute this from their 18-page loan agreement — until now.*

## What It Does

Upload any loan agreement (photo or PDF). LoanLens:

1. **Extracts** every fee, charge, and rate using Gemini Vision (PDF multi-page, JPG, PNG, WebP)
2. **Computes** the true annualized APR — simple and IRR-based (per RBI KFS Circular April 2024)
3. **Checks** 21 RBI Fair Practice Code compliance rules
4. **Explains** the verdict in English, Hindi, and Kannada
5. **Generates** a downloadable PDF evidence report for the RBI Ombudsman

## SDGs Addressed
- **SDG 1** — No Poverty (predatory loans trap borrowers in debt cycles)
- **SDG 10** — Reduced Inequalities (lower-income borrowers disproportionately targeted)
- **SDG 16** — Justice and Accountability (make RBI protections accessible to all)

## Architecture

```
React Frontend (Firebase Hosting)  ← Syne + Inter fonts, mobile-first
         ↓ HTTPS /api/v1
FastAPI Backend (Cloud Run)
    ├── Gemini 1.5 Flash  → Document extraction + multilingual verdict
    ├── APR Engine         → Pure Python, deterministic (simple + IRR)
    ├── RBI Checker        → 21-rule compliance evaluation
    └── WeasyPrint         → PDF evidence report generation
```

## Tech Stack (All Free Tier)

| Component | Technology |
|---|---|
| Backend | FastAPI, Python 3.11+ |
| AI | Gemini 1.5 Flash |
| Deployment | Google Cloud Run |
| Frontend | React + Vite |
| Frontend hosting | Firebase Hosting |
| PDF | WeasyPrint |

**Infrastructure cost: ₹0**

---

## Getting Started

### Prerequisites
- Python 3.11+
- Node.js 18+
- Google AI Studio API key ([get one free](https://aistudio.google.com))

### Backend Setup

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate
# Linux/Mac
source venv/bin/activate

pip install -r requirements.txt
cp .env.example .env
# Open .env → add your GEMINI_API_KEY from aistudio.google.com
```

### Day 1: Validate APR Engine (no API key needed)

```bash
python validate_day1.py --demo
```

Expected output:
```
APR Simple:   62.3%  (expect ~62–66%)   ✓
APR Compound: 88.7%  (expect ~75–85%)   ✓
Total cost:   ₹7,677  (expect ~₹7,720)  ✓
✅ APR engine working correctly — proceed with full build!
```

### Run Tests

```bash
pytest tests/ -v
# 17 passed in 0.21s ✅
```

### Start the Backend

```bash
uvicorn app.main:app --reload --port 8000
```

### Validate with a Real Loan PDF (requires API key in .env)

```bash
python validate_day1.py --file path/to/loan.pdf
```

### Frontend Setup

```bash
cd ../frontend
npm install
npm run dev
# Open http://localhost:5173
```

---

## API Endpoints

```
GET  /health                → Health check + model info
POST /api/v1/analyze        → Upload loan doc → full AnalysisResult JSON
POST /api/v1/report/pdf     → AnalysisResult JSON → downloadable PDF bytes
```

### Test the API

```bash
curl http://localhost:8000/health

curl -X POST http://localhost:8000/api/v1/analyze \
  -F "file=@sample_loan.pdf" | python -m json.tool
```

---

## RBI Rules Checked (21 total)

**Group A — KFS Compliance**: KFS present · APR explicitly stated · Fees itemized · Grievance mechanism

**Group B — Loan Terms**: Tenure ≥ 30 days · Repayment schedule · Prepayment terms · Penal charges · Cooling-off period ≥ 3 days · No compound penal interest

**Group C — Lender Identity**: Lender name · RBI registration number · Grievance officer contact

**Group D — Cost Transparency**: Processing fee stated · Insurance disclosed separately · GST stated · Disbursed = sanctioned minus disclosed fees only

**Group E — Recovery Practices**: No upfront fees before disbursement · Bank transfer repayment only · Late fee cap compliance · No contact list access

---

## Project Structure

```
loanlens/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI app + CORS + routers
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── analyze.py             # POST /api/v1/analyze
│   │   │   └── report.py              # POST /api/v1/report/pdf
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   └── config.py              # pydantic-settings (key, model, CORS)
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   └── schemas.py             # All Pydantic models + Enums
│   │   └── services/
│   │       ├── __init__.py
│   │       ├── gemini_extraction.py   # Gemini Vision, PyMuPDF multi-page
│   │       ├── apr_engine.py          # Simple + IRR-based APR computation
│   │       ├── rbi_checker.py         # 21 RBI compliance rules
│   │       ├── verdict_service.py     # EN/HI/KN verdict via Gemini
│   │       └── pdf_report.py          # WeasyPrint PDF evidence report
│   ├── tests/
│   │   ├── __init__.py
│   │   └── test_apr_engine.py         # 17 unit tests
│   ├── conftest.py                    # pytest sys.path fix (required)
│   ├── pytest.ini                     # asyncio_mode = auto (no warnings)
│   ├── validate_day1.py               # Day 1 sanity script (--demo / --file)
│   ├── Dockerfile                     # Cloud Run deployment
│   ├── requirements.txt
│   ├── .env.example                   # Template — copy to .env
│   ├── .env                           # Your secrets — never commit
│   ├── LoanLens_PRD.md               # Product requirements document
│   └── LOVABLE_PROMPT.md             # Frontend spec (reference only)
└── frontend/
    ├── src/
    │   ├── App.jsx                    # Full React SPA (624 lines)
    │   └── main.jsx                   # React entry point
    ├── index.html                     # Vite entry + SEO meta
    ├── vite.config.js                 # Vite + /api proxy to :8000
    ├── package.json
    ├── .env.local                     # VITE_API_URL=http://localhost:8000/api/v1
    └── .gitignore
```

---

## Deploy

### Backend → Cloud Run

```bash
cd backend
gcloud run deploy loanlens-api \
  --source . \
  --region asia-south1 \
  --allow-unauthenticated \
  --set-env-vars GEMINI_API_KEY=your-key-here
```

### Frontend → Firebase Hosting

```bash
cd frontend && npm run build
firebase deploy --only hosting
```

---

## RBI References

- [RBI Digital Lending Directions, 2025](https://www.rbi.org.in)
- [RBI Fair Practice Code for NBFCs (Master Circular)](https://www.rbi.org.in)
- [RBI KFS Circular (April 2024)](https://www.rbi.org.in)
- RBI Integrated Ombudsman: [cms.rbi.org.in](https://cms.rbi.org.in) · Helpline: **14448** (toll-free)

## Future Roadmap

- WhatsApp bot interface (reach borrowers where they are)
- RBI Ombudsman complaint auto-draft from LoanLens report
- Lender legitimacy check against RBI DLA directory (July 2025)
- Community database of flagged loan agreements (anonymized)

---

*Built for Google Solution Challenge 2026 · SDG 1 (No Poverty) · SDG 10 (Reduced Inequalities) · SDG 16 (Justice)*
