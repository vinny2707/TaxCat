# PA#1 — Proposal and Planning

**Course:** CSC13114 · **Team:** 3 members · **Project:** TaxCat — Expense classifier for freelancers

**Members:** 
| Tên | MSSV |
|--|--|
| Nguyễn Khắc Vượng | 23120409 | 
| Trần Quốc Vỹ | 23120410 |
| Bửu Huỳnh Vĩnh Phúc | 23120328 |
**Repository:** https://github.com/2609-AWAD-23-3/taxcat.git

---

## 1. Problem and users

**User:** Minh, 28, a freelance graphic designer in Ho Chi Minh City. Every month Minh receives 40–60 invoices from various vendors (Adobe, Figma, stationery shops, Grab, cafés for client meetings). At year-end, Minh must compile them for personal income tax filing.

**Problem:** Minh does not know which expenses are tax-deductible and which are not, so Minh currently types every invoice into Excel by hand - losing 6–8 hours a year and still misclassifying or missing receipts.

**What Minh does today instead:** Photographs receipts, stores them in Google Drive, then re-enters them into Excel from memory at year-end. Slow, error-prone, no automated tool exists for Vietnamese freelancers.

---

## 2. The LLM feature and the cost of being wrong

**Feature:** User uploads invoice images (JPG/PNG/PDF) → OCR extracts text → LLM extracts **amount, vendor, date, description** and **classifies into 10 fixed tax categories** (office supplies, marketing, travel, client meals, equipment, software, professional services, telecom, training, other). The LLM returns a **confidence label (high / medium / low)** per invoice; low-confidence items are flagged "needs review" for the user to verify.

**Cost when wrong:**

| Error type | Who is harmed | How much | Undoable? |
|---|---|---|---|
| Wrong category | Minh (user) | Administrative penalty + back taxes if audited | Fixable but time-consuming; late-filing penalties may apply |
| Missed invoice | Minh | Loses a legitimate deduction → direct money loss | Not undoable after tax return is filed |
| Wrong amount extracted | Minh | Incorrect total income reported → legal risk | Requires amended return; costly |

**How we would know:** Ground-truth test set of 30 invoices (correct category + amount). Measure precision/recall. Acceptance threshold: category precision ≥ 85%, amount extraction ≥ 95%.

---

## 3. Scope

**In-scope (this semester):**
- Upload invoice images JPG/PNG/PDF (up to 5 per batch).
- OCR extracts text from Vietnamese and English invoices.
- LLM classifies into **10 fixed categories**, extracts amount, vendor, date.
- Display results with confidence (high/medium/low); user can correct category.
- Export CSV in a simple format for tax filing.
- Basic web UI: upload, list view, edit, export.

**Out-of-scope (deliberately excluded):**
- Online tax filing or connection to tax authorities.
- Bank / e-wallet / accounting integration.
- Tax law advice or tax amount calculation.
- Handwritten invoice support (OCR unreliable).
- Multi-user, role-based access, advanced analytics dashboard.
- Mobile app.

**Why out:** Keep scope small but sharp; rubric states "ambition is not scored."

---

## 4. Plan and ownership — 6 checkpoints

| Checkpoint | Work | Owner | Deadline |
|---|---|---|---|
| CP1 | Finalize user/problem/feature/scope; set up repo + environment | **Phúc** | Week 3 |
| CP2 | OCR pipeline (PaddleOCR + OpenCV preprocessing) | **Phúc** | Week 6 |
| CP3 | Prompt engineering + LLM classification; define 10 categories | **Vượng** | Week 8 |
| CP4 | Web UI (upload, display, edit, export CSV) + FastAPI backend | **Vỹ** | Week 10 |
| CP5 | Ground-truth test set of 30 invoices + precision/recall measurement | **Vượng** (lead) + Phúc | Week 12 |
| CP6 | Demo, report, self-assessment, submission | **Vỹ** (lead) + cả nhóm | Week 14 |

**Note:** Each checkpoint has one lead owner; others support as needed. Plan aligns with the course's six-checkpoint calendar.

---

## 5. Risks

**Risk 1 — OCR accuracy drops on real-world photos (tilted, blurry, poor lighting).**
- *Why it sinks the project:* If OCR fails, LLM classifies garbage — the entire feature becomes worthless. Benchmarks show PaddleOCR accuracy drops sharply on low-quality phone photos.
- *Mitigation (start this week):* Build an OpenCV preprocessing pipeline (grayscale → threshold → deskew → denoise) before OCR; prepare a fallback using **Gemini Flash Vision** (free tier 1,000 requests/month) for low-quality images; require clear uploads with UI guidance.

**Risk 2 — LLM misclassifies "ambiguous" expenses (e.g., "meal with client" could be client entertainment or personal).**
- *Why it sinks the project:* Misclassification directly causes tax risk for the user — the exact cost named in Section 2.
- *Mitigation (start this week):* Restrict LLM to the 10 fixed categories (no new categories); require confidence + reason from the LLM; flag low-confidence items for review; build a 30-invoice ground-truth set to measure precision/recall and iterate on prompts.

---

## 6. Technology choices

| Component | Choice | One-line justification | Cost |
|---|---|---|---|
| **Frontend** | React + Vite | Enough for upload/view/edit UI, well-documented, team is familiar | Free |
| **Backend** | FastAPI (Python) | OCR + LLM are both Python, easy integration, async for uploads | Free |
| **OCR** | PaddleOCR | Best Vietnamese diacritic handling among OSS options, CPU-friendly, has Vietnamese invoice reference projects | Free (Apache 2.0) |
| **LLM (primary)** | Gemini 2.0 Flash (vision + text) | Free tier 1,000 req/month, enough for 30–50 invoice demo, native vision | **$0** in free tier |
| **LLM (local fallback)** | Qwen3-VL-2B (GGUF 4-bit) via Ollama | Runs locally free when Gemini quota is exhausted or privacy is needed | Free |
| **Database** | SQLite | Enough for demo, no server, easy backup | Free |
| **Model hosting (dev)** | Kaggle Notebook (free T4 GPU 30h/week) + Hugging Face (model download) | Develop and test pipeline without a powerful local machine | Free |

**Estimated total cost:** $0 if within Gemini free tier and using local PaddleOCR. If Gemini quota exceeded: ~$0.10–$0.50 for 100–200 extra requests.

---

## Repository link

https://github.com/2609-AWAD-23-3/taxcat

*(First commit will be pushed after CP1.)*