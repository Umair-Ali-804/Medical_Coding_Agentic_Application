<div align="center">

<img src="backend/app/demo/assets/logo_full.png" alt="AAXIS AI Automations" width="220">

# Medical Coding Intelligence

### From clinical note to a clean claim, with every suggestion checked against the documentation.

AI-assisted medical coding and pre-bill claim integrity platform by **AAXIS AI Automations**

![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-API-009688?logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-Review%20UI-000000?logo=nextdotjs&logoColor=white)
![Tests](https://img.shields.io/badge/tests-77%20passing-2ea44f)
![Code sets](https://img.shields.io/badge/ICD--10--CM%20%C2%B7%20PCS%20%C2%B7%20HCPCS%20%C2%B7%20CPT-FY2026-555)

[Overview](#overview) · [Screenshots](#screenshots) · [How it works](#how-it-works) · [Results](#evaluation-results) · [Quick start](#quick-start) · [Contact](#contact)

</div>

---

## Overview

Claim denials rarely come from complex medicine. They come from details: an incomplete diagnosis code, units above the Medicare limit, a missing modifier, or a code set that expired last quarter. Each error costs a resubmission, an appeal, or revenue that is never recovered.

**Medical Coding Intelligence** closes the gap between clinical documentation and a clean claim:

1. **Reads the clinical note** and identifies conditions and procedures, recognizing negation, uncertainty and family history.
2. **Suggests codes** across ICD-10-CM, HCPCS, CPT and ICD-10-PCS, each checked against a verbatim quote from the note.
3. **Cites the most relevant Official Coding Guideline passage** for each code.
4. **Validates every suggestion** with a deterministic rule engine and a platform-computed confidence score.
5. **Drafts the claim** and runs a **pre-bill denial check** before submission, mapping each finding to the payer denial code it would trigger.
6. **Routes everything to a certified coder**, who approves, edits or rejects each code, with a complete audit trail.

> **Design principle: the AI proposes, a coder decides.** Suggestions are grounded in official reference text, verified by rules that cannot hallucinate, scored by the platform rather than by the model's self-reported confidence, and approved by a human. Every step is versioned and written to a tamper-evident audit log.

### Who it is for
Medical billing companies, physician practices, clinics and revenue cycle management (RCM) teams that want **fewer denials, faster coding and fewer missed charges**.

---

## Screenshots

<table>
<tr>
<td width="50%"><img src="screen_shorts/AAXIS_Medical_Coding_04.png" alt="Evidence-backed code suggestions"><br><sub><b>Code a note:</b> diagnoses and procedures with verbatim evidence, confidence and validation status</sub></td>
<td width="50%"><img src="screen_shorts/AAXIS_Medical_Coding_05.png" alt="Pre-bill claim scrubber"><br><sub><b>Claim scrubber:</b> denial risks mapped to payer reason codes, with fixes and rule sources</sub></td>
</tr>
<tr>
<td width="50%"><img src="screen_shorts/AAXIS_Medical_Coding_01.png" alt="Workbench overview"><br><sub><b>Workbench:</b> paste a clinical note and review codes, the draft claim and the denial check on one screen</sub></td>
<td width="50%"><img src="screen_shorts/AAXIS_Medical_Coding_07.png" alt="Branded PDF reports"><br><sub><b>Reports:</b> one-click branded PDFs for coders, auditors and billing teams</sub></td>
</tr>
</table>

---

## Key capabilities

| | Capability | What it delivers |
|---|---|---|
| 🩺 | **Clinical understanding** | Section-aware NLP with ConText/NegEx assertion detection: negated, uncertain, historical, family and hypothetical findings are never coded as current conditions |
| 🔎 | **Evidence-backed coding** | Hybrid retrieval (dense embeddings + BM25) over the official code sets; every suggestion is checked against a verbatim quote from the note, and unsupported quotes are flagged |
| 📘 | **Guideline citations** | ICD-10-CM and ICD-10-PCS Official Guidelines indexed into citable passages (for example `I.C.4.a`, `B3.1b`); the most relevant passages are attached to each code for the coder to review |
| ⚙️ | **Procedure coding** | CPT/HCPCS for outpatient and ICD-10-PCS for inpatient encounters, only for procedures documented as performed; drug units derived from the documented dose. Procedure suggestions always go to coder review |
| ✅ | **Deterministic validation** | Code existence and billability on the date of service, verbatim-evidence check, Excludes1 conflicts, laterality, sex/age edits, manifestation sequencing, duplicates |
| 🛡️ | **Pre-bill claim scrubber** | 30+ rule checks (CMS MUE unit limits, modifier logic, E/M + modifier 25, HCPCS coverage and termination, diagnosis validity, sequencing, sex/age edits, diagnosis pointers; NCCI bundling, medical necessity and dollars at risk when those CMS/payer files are loaded), each with the typical CARC/RARC, a fix and a risk score |
| 📄 | **PDF reports** | Branded coding reports, claim-check reports and code reference sheets |
| 🔐 | **Security & audit** | RBAC, encrypted PHI at rest, hashed API keys, hash-chained audit log, PHI-free logs |

---

## How it works

```mermaid
flowchart LR
    UI[Review UI / Workbench] -->|session| API
    N8N[n8n orchestrator] -->|API key| API
    subgraph API[FastAPI service]
      ING[Ingestion<br/>PDF · DOCX · TXT<br/>cleaning · sections] --> NLP[Clinical NLP<br/>lexicon · problem list · LLM<br/>ConText negation]
      NLP --> RAG[Hybrid retrieval<br/>dense + BM25 · RRF]
      RAG --> LLM[Coding model<br/>LLM strict JSON or<br/>deterministic coder]
      LLM --> VAL[Validation engine<br/>evidence · Excludes1 ·<br/>laterality · sex/age]
      VAL --> CONF[Confidence<br/>+ routing]
      CONF --> SCRUB[Claim scrubber<br/>MUE · NCCI · modifiers ·<br/>CARC mapping]
    end
    API --> PG[(PostgreSQL<br/>encrypted PHI · audit chain · job queue)]
    RAG --> QD[(Vector store)]
    WORKER[Worker replicas] --> PG
    API -. signed webhooks .-> N8N
```

---

## Evaluation results

Measured on a **synthetic benchmark** (36 clinical notes, 71 gold ICD-10-CM codes) using the deterministic retrieval-only coder with lexical retrieval and no LLM. The held-out test split (12 notes, 25 gold codes) was run once, after all tuning on the dev and validation splits. With a small test set, results can vary noticeably on new data.

| Split | Precision | Recall | F1 | Category F1 | Exact match | Unsupported codes | Retrieval recall |
|---|---|---|---|---|---|---|---|
| Dev (tuned) | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 0% | 100% |
| Validation | 0.88 | 0.88 | 0.88 | 0.96 | 0.83 | 0% | 96% |
| **Test (held out)** | **0.92** | **0.96** | **0.94** | **1.00** | **0.92** | **0%** | **100%** |

| Claim scrubber (26 labeled test claims) | Result |
|---|---|
| Expected errors detected | **22 / 22 (100%)** |
| False alarms on clean claims | **0** |
| End-to-end time per note (codes, claim check and PDF, offline coder) | **≈ 3 seconds** |

> These results come from synthetic data and are not a guarantee of real-world accuracy. Accuracy on real documentation is established per client through a pilot on de-identified charts. The LLM baselines (LLM only → + RAG → + validation → full pipeline) are supported by the evaluation runner, require an OpenRouter key, and have not yet been benchmarked. With an LLM enabled, processing time per note increases (typically tens of seconds, depending on the model).

Full methodology and error analysis: [docs/evaluation.md](docs/evaluation.md)

---

## Quick start

### Option 1: Google Colab (fastest demo)
Open [`colab/MedicalCodingAI_Colab.ipynb`](colab/MedicalCodingAI_Colab.ipynb) or follow [colab/COLAB_COMMANDS.md](colab/COLAB_COMMANDS.md). The notebook loads the reference data, builds the search index and launches the workbench UI with a public link.

### Option 2: Docker (full platform)

```bash
python scripts/setup_env.py            # creates .env with generated secrets
docker compose up -d --build
docker compose logs -f migrate         # wait for "knowledge base ready" (first run: 5–10 min)
```

| Service | URL |
|---|---|
| Review UI | http://localhost:3000 |
| API documentation | http://localhost:8000/docs |
| n8n automation | http://localhost:5678 |

Create a coder account and an integration key:

```bash
docker compose exec api python -m app.cli create-user coder@example.org --role coder
docker compose exec api python -m app.cli create-api-key n8n --role service
```

For production, `docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d` adds TLS termination (Caddy) and removes directly published ports. See [docs/operations.md](docs/operations.md).

> The Colab workflow is the most thoroughly tested path for demos. Verify the Docker deployment in your environment before production use.

### Option 3: Local development

```bash
cd backend
pip install -e ".[dev,kb,embeddings]"
export DATABASE_URL=postgresql+psycopg://medcoding:medcoding@localhost:5432/medcoding
alembic upgrade head
python -m app.cli kb load-all --data-dir ../data_sources
python -m app.cli kb index --system all
python -m app.cli create-user you@example.org --role admin
uvicorn app.main:app --reload            # API on :8000
python -m app.worker                     # background jobs
python -m app.demo.gradio_app            # workbench UI on :7860
```

Without an LLM key, set `LLM_PROVIDER=heuristic` to run the complete pipeline with the deterministic coder.

**Useful commands**

```bash
python -m app.cli kb freshness --dos 2026-10-01          # are code sets valid for this date of service?
python -m app.cli code-note ../data/sample_notes/01_office_visit_knee_injection.txt --sex M --age 67
python -m app.cli claims scrub claim.json                # pre-bill check for a claim
pytest                                                   # 77 pass, 3 need a Postgres test server
```

---

## Reference data

The platform loads official CMS/CDC files in their published formats. Each set carries a validity window, and the platform flags any code set that is not valid for a claim's date of service.

| Source | Content |
|---|---|
| ICD-10-CM (CDC/CMS) | Codes, billability, instructional notes (Excludes1/2, code first, use additional code) |
| ICD-10-PCS (CMS) | Inpatient procedure codes (supported when `icd10pcs_codes_<year>.txt` is added to `data_sources/`) |
| HCPCS Level II (CMS) | Codes, modifiers, coverage codes, termination dates, processing notes |
| NCCI MUE / PTP (CMS) | Unit-of-service limits (included) and procedure-to-procedure bundling edits (supported; add the CMS PTP file) |
| Official Guidelines (CMS/NCHS) | ICD-10-CM guideline passages (included) and ICD-10-PCS passages (add the PCS guidelines PDF) |
| CPT® (AMA) | Loaded only from a licensed file; **no CPT content is distributed** with this repository. CPT suggestions are limited to the codes in the licensed file you load |

See [docs/DATA_SOURCES.md](docs/DATA_SOURCES.md) for the full inventory, update schedule and download links.

---

## Project structure

```
backend/                 FastAPI service, worker, CLI, migrations, tests
  app/ingestion          document extraction, cleaning, sections, encrypted storage
  app/extraction         clinical lexicon, problem-list parser, ConText, LLM reconciliation
  app/knowledge          code-set loaders (CMS/CDC formats), claim edits, guidelines index
  app/rag                embeddings, vector stores, BM25, hybrid retriever
  app/llm                LLM client, schemas, versioned prompts
  app/coding             diagnosis and procedure coding engine
  app/validation         deterministic rule engine, evidence matching
  app/confidence         confidence scoring and calibration
  app/claims             pre-bill claim scrubber
  app/evaluation         baselines, metrics, error analysis
  app/services           pipeline, review, audit, jobs, webhooks
  app/demo               workbench UI (HTML/CSS) and PDF reports
frontend/                Next.js review workspace
n8n/workflows/           document intake and event automation workflows
data/                    synthetic evaluation data and sample notes
data_sources/            official reference files
colab/                   Colab notebook and commands
docs/                    architecture, security, operations, evaluation, data sources
```

---

## Security and compliance

- Role-based access control (admin, coder, auditor, service); only human coders can approve codes
- Encryption of PHI columns and stored documents at rest, with key rotation
- Hashed API keys, bcrypt passwords, login rate limiting, security headers
- Hash-chained audit log (on PostgreSQL, a database trigger also blocks edits and deletions)
- PHI-free application logs and webhooks
- Full traceability of model, prompt, retrieval and knowledge-base versions for every suggestion

See [docs/security-compliance.md](docs/security-compliance.md).

> **Important.** This software is decision support for certified coders, not an autonomous coding system. Before processing real patient data, establish the legal and compliance requirements for your market (for example, HIPAA and a Business Associate Agreement with every processor, including any LLM provider), complete a security review, and validate accuracy on your own de-identified data. CPT® is a registered trademark of the American Medical Association; use CPT content only under a valid license.

---

## Contact

<div align="center">

<img src="backend/app/demo/assets/logo_wordmark.png" alt="AAXIS AI" width="200">

**AAXIS AI Automations**: intelligent automation for healthcare revenue cycles

📞 **+92 314 7357980** · ✉️ **aliumair64488@gmail.com**

Interested in a live demo or a pilot on your own charts? Get in touch.

</div>
