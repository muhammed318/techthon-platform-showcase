<div align="center">

# 🏆 TechThon • TAEF-AI Platform
### Automated Software Project Auditor & Next-Gen Hackathon Hub in MENA

[![Platform Status](https://img.shields.io/badge/Platform-Live-success?style=for-the-badge&logo=vercel)](https://techthon1.vercel.app)
[![API Status](https://img.shields.io/badge/API-Online-blue?style=for-the-badge&logo=fastapi)](https://techthon-api.onrender.com/health)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=for-the-badge)](#)
[![Region](https://img.shields.io/badge/Region-Turkey%20%26%20MENA-purple?style=for-the-badge)](#)

<br/>

**[🌐 Visit Live Platform](https://techthon-frontend.vercel.app/)** • **[📑 API Health Check](https://techthon-api.onrender.com/health)** • **[⚙️ Technical Architecture](./ARCHITECTURE.md)**

<br/>

</div>

---

## 📌 Executive Summary

**TechThon** is an enterprise hackathon platform and developer talent ecosystem headquartered in Turkey. Integrated at its core is **TAEF-AI**, an autonomous, evidence-grounded software code auditing engine designed to replace subjective human grading in hackathons with deterministic, bias-free technical evaluations.

> 🔒 **Notice:** This repository is an **Architectural Showcase & Public Blueprint**. The proprietary core source code and algorithmic weights are kept in private production repositories.

---

## 🌟 Core Platform Pillars

```text
                             TechThon Ecosystem
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
   [ Hackathons Hub ]       [ TAEF-AI Engine ]         [ Talent & Hiring Board ]
  Real-world coding         Autonomous auditing        Verified skill metrics
  challenges & prizes       & evidence validation      for corporate sponsors
Autonomous TAEF-AI Auditor: Evaluates repositories with zero human bias, combining deterministic AST scanning with ground-truth LLM analysis.
Deterministic Evidence Verifier: Cross-references every AI claim against physical filesystem files, systematically eliminating hallucinations.
Dual Execution Runtime (Cloud + 100% Private Offline):
Cloud AI: Supports OpenRouter API (Qwen 2.5 72B, Claude 3.5 Sonnet, Llama 3.3).
On-Premise Privacy: Users can toggle to Local Ollama (localhost:11434) for 100% offline, free, zero-leak auditing.
Bring Your Own Key (BYOK): Zero-cost server footprint by empowering users to use their personal API keys directly.
Interactive Executive Reports: Generates in-depth architecture telemetry, security observations, actionable roadmaps, and downloadable official PDF audit certificates.
🛠 High-Level Architecture & Tech Stack
code
Text
[ Next.js 14+ Frontend ]  ──(JWT / REST API)──►  [ FastAPI Production Backend ]
   (Vercel Edge Network)                             (Render Cloud Service)
                                                               │
                                       ┌───────────────────────┴───────────────────────┐
                                       ▼                                               ▼
                         [ PostgreSQL 16 DB (Neon) ]                 [ AI Inference Engine ]
                         - User Profiles & RBAC                      - OpenRouter API (BYOK)
                         - Hackathons & Registrations                - Local Ollama (Private)
                         - Persisted Audits & Telemetry              - Evidence Verifier Engine
Frontend: Next.js (App Router), TypeScript, Tailwind CSS, Framer Motion, Axios, jsPDF.
Backend: Python 3.12, FastAPI, SQLAlchemy 2.0, PyJWT, SlowAPI, Uvicorn.
Database: Serverless PostgreSQL on Neon.tech with Connection Pooling & Alembic migrations.
Security: Zip bomb & Zip slip defense, stream upload limits (50MB), strict CORS, SSRF protection.
📊 Live Verification & Pipeline Flow
code
Text
Upload ZIP (Max 50MB) 
   │
   ▼
[ Security & Sanitization ] ──► Verifies compression ratio (<15:1) & path traversal
   │
   ▼
[ Deterministic Static Scan ] ──► Gathers AST facts, dependencies & Monorepo frameworks
   │
   ▼
[ Concurrent AI Auditing ] ──► Executes parallel LLM batches (asyncio.gather)
   │
   ▼
[ Evidence Verification ] ──► Matches citations against physical files on disk
   │
   ▼
[ Executive PDF & Report ] ──► Real-time token metering, scoring & recommendations
📄 Documentation Links
System Architecture Blueprint: Deep dive into design patterns, concurrency, and database schemas.
Security & Hardening Policy: Comprehensive report on applied enterprise security measures.
🏢 Contact & Corporate Partnerships
Are you an enterprise looking to host automated hackathons or scout top-ranked tech talent in Turkey and the Middle East?
Website: https://techthon-frontend.vercel.app/
Email: contact@techthon.io / admin@techthon.io
Location: Istanbul, Turkey
<div align="center">
<sub>TECHTHON • TAEF-AI © 2026. All rights reserved. Proprietary software showcase.</sub>
</div>
```
