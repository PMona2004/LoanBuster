# Loan Buster 🔍

**AI Predatory Lending Decoder** · ET AI Challenge 2.0 2026  
Track: **AI for Digital Public Safety: Defeating Counterfeiting, Fraud & Digital Arrest Scams (Open Innovation)** · SDGs: **1 · 10 · 16**
Theme: **Digital Trust**

NOTE : Has been moved from GCP deployment to Vercel + Railway 
> *"You don't need a CA. You don't need a lawyer. Photograph your loan agreement. Loan Buster tells you in Kannada if you're being cheated — and gives you the paperwork to prove it."*

---

## The Problem

Millions of Indians borrow from digital lending apps. A loan advertised at **"2% per month"** routinely becomes **30–60% APR** once processing fees, insurance premiums, GST, and other charges are factored in. The loan agreement disclosing all this is 12–20 pages of legal English.

**RBI mandates full APR disclosure in a Key Fact Statement before signing. Hundreds of apps violate this. No tool existed to prove it — until now.**

---

## What Loan Buster Does

Upload any loan agreement (PDF or photo). Get back in ~60 seconds:

| Output | Detail |
|---|---|
| **Real APR** | Deterministic IRR computation per RBI KFS Circular, April 2024 |
| **APR Gap** | Declared vs. actual — how much more you're really paying |
| **RBI Compliance** | 21-rule checklist against RBI Fair Practice Code 2025 |
| **Plain-language verdict** | In English, Hindi, and Kannada |
| **Predatory Risk Score** | 0–100 composite score (APR + violations + fee traps) |
| **Confidence module** | Flags unverified fields, distinguishes missing vs. unclear |
| **Evidence report** | Downloadable PDF with math, violations, borrower rights, Ombudsman contact |

---

## Live Demo

> Upload the KreditBee KFS sample → Result: **CAUTION**, 34.87% actual vs 29.64% declared, 0 violations  
> Upload a typical microloan agreement → Result: **SEVERE**, 200%+ APR, fee deduction trap flagged

---

## Quick Start

### Prerequisites
- Python 3.12+
- Node.js 18+
- A [Google AI Studio](https://aistudio.google.com/) API key (free)

### Backend
```bash
cd backend
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS/Linux
pip install -r requirements.txt
cp .env.example .env
# Edit .env: set GEMINI_API_KEY=your_key_here
uvicorn app.main:app --reload --port 8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev
# Open http://localhost:5173
```

### Integration Test
```bash
cd backend
python test_day2.py
# Uploads a real PDF → prints full analysis result
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│            FRONTEND (React + Vite)                      │
│  Upload → Loading → Results → Download Evidence PDF      │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS POST /api/v1/analyze
┌──────────────────────▼──────────────────────────────────┐
│              BACKEND (FastAPI + Python)                  │
│                                                         │
│  1. File validation & preprocessing                      │
│  2. Gemini 2.5 Flash Vision → structured JSON extraction │
│  3. APR Engine (IRR + simple, deterministic, zero AI)   │
│  4. RBI Rule Checker (21 rules, PASS/FAIL/UNCLEAR)       │
│  5. Predatory Risk Score (0–100 composite)              │
│  6. Multilingual verdict (EN/HI/KN via Gemini)          │
│  7. PDF Evidence Report (WeasyPrint)                    │
│  8. Full AnalysisResult JSON → Frontend                 │
└──────┬────────────────┬────────────────────────────────┘
       │                │
┌──────▼──────┐  ┌──────▼──────┐
│ Gemini 2.5  │  │  WeasyPrint │
│ Flash Vision│  │  PDF Engine │
│ (Google AI) │  │  (in-proc)  │
└─────────────┘  └─────────────┘
```

---

## Project Structure

```
loan-buster/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── analyze.py          ← Main orchestration endpoint
│   │   │   └── report.py           ← PDF generation endpoint
│   │   ├── core/
│   │   │   └── config.py           ← Settings (Pydantic BaseSettings)
│   │   ├── models/
│   │   │   └── schemas.py          ← All Pydantic models (ExtractedLoanData, APRBreakdown, AnalysisResult...)
│   │   └── services/
│   │       ├── gemini_extraction.py ← Gemini Vision multi-page extraction
│   │       ├── apr_engine.py        ← Deterministic APR/IRR engine
│   │       ├── rbi_checker.py       ← 21-rule RBI compliance checker
│   │       └── verdict_service.py   ← Multilingual verdict generation
│   ├── tests/                       ← Unit tests (APR engine, RBI rules)
│   ├── .env.example
│   ├── Dockerfile
│   ├── requirements.txt
│   └── test_day2.py                ← Integration test with real PDFs
└── frontend/
    ├── src/
    │   └── App.jsx                 ← Full React app (single file)
    ├── index.html
    ├── vite.config.js
    └── package.json
```

---

## APR Engine — How It Works

The APR engine is **100% deterministic Python** — no AI involved in the math.

```
Effective APR = IRR-based compound rate per RBI KFS Circular, April 2024

Inputs:  Principal, Tenure, Interest Rate (flat/reducing/monthly),
         Processing Fee, Insurance Premium, GST on Fees, Other Charges

Step 1:  Cross-check against total_repayment_amount if stated in document
         (document data wins over computed data)
Step 2:  Resolve interest type (Gemini extracts; heuristic fallback)
Step 3:  Aggregate all mandatory charges (excluding contingent/penal fees)
Step 4:  Compute simple APR: (total_cost / principal) × (365 / tenure_days)
Step 5:  Compute IRR APR:
         - EMI loans (>45 days): npf.irr on monthly cash flows → annualize
         - Bullet/payday loans: analytical 2-cash-flow solution
Step 6:  Fee deduction trap detection
Step 7:  Short-tenure annualization note (loans < 30 days)
```

**Key insight for judges:** A standard APR calculator requires the borrower to already know all their numbers and type them in. Loan Buster reads the agreement, extracts every number, and computes the result. The gap between 29.64% declared and 34.87% actual on the KreditBee document? It's ₹589 in fees charged on the full principal even though the borrower only received ₹34,411. No borrower would calculate this manually.

---

## RBI Compliance — 21 Rules Checked

Based on RBI Digital Lending Directions 2025 + Fair Practice Code:

| Group | Rules |
|---|---|
| **A: Key Fact Statement** | KFS present · APR explicitly stated · All fees itemized · Grievance mechanism |
| **B: Loan Terms** | Tenure ≥ 30 days · Repayment schedule · Prepayment terms · Penal charges · Cooling-off ≥ 3 days |
| **C: Lender Identity** | NBFC/Bank name · RBI registration number · Grievance officer contact |
| **D: Cost Transparency** | Processing fee stated · Insurance disclosed separately · GST stated · Disbursed = sanctioned |
| **E: Recovery Practices** | No undisclosed deductions · Bank transfer only · Late fee cap compliant · No upfront fee · App permissions |

Each rule returns **PASS / FAIL / UNCLEAR** with a plain-language detail string.

---

## Deployment

### Backend → Google Cloud Run
```bash
gcloud run deploy loanlens-api \
  --source backend/ \
  --region asia-south1 \
  --set-env-vars GEMINI_API_KEY=your_key \
  --allow-unauthenticated
```

### Frontend → Firebase Hosting
```bash
# Update frontend/.env.local with your Cloud Run URL
cd frontend
npm run build
firebase deploy --only hosting
```

**Total infrastructure cost: ₹0** (Cloud Run free tier: 2M requests/month; Gemini 2.5 Flash: free tier)

---

## Test Results (Verified)

| Document | Declared APR | Actual APR | Violations | Severity |
|---|---|---|---|---|
| KreditBee KFS (270-day ₹35,000 loan) | 29.64% | 34.87% | 0 | CAUTION |
| mPokket penal policy (microloan) | — | 200%+ | Multiple | SEVERE |

---

## SDG Alignment

| SDG | How |
|---|---|
| **SDG 1: No Poverty** | Predatory lending traps borrowers in debt cycles. Loan Buster arms them with proof to challenge unfair terms. |
| **SDG 10: Reduced Inequalities** | Equalizes information asymmetry between sophisticated lenders and first-time borrowers. |
| **SDG 16: Strong Institutions** | Enforces RBI regulations that already exist but are invisible to the average borrower. |

---

## RBI References

- [RBI Digital Lending Directions, 2025](https://www.rbi.org.in) (May 8, 2025 — supersedes 2022 Guidelines)
- [RBI Fair Practice Code for NBFCs (Master Circular)](https://www.rbi.org.in)
- [RBI KFS Circular (April 2024)](https://www.rbi.org.in)
- RBI Integrated Ombudsman: [cms.rbi.org.in](https://cms.rbi.org.in) · Helpline: **14448** (toll-free, Mon–Fri 9am–5pm)

---

## Future Roadmap (Post-Challenge)

- Human-in-the-loop field correction for low-confidence extractions
- RBI Ombudsman complaint auto-draft with prefilled evidence
- Lender credibility score (RBI's DLA directory API)
- Community database of flagged loan agreements (anonymized)

---

*Loan Buster v2.0 MVP | Solution Challenge 2026 | Solo Dev | Track: Unbiased AI Decision*
