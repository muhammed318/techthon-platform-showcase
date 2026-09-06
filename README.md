<div align="center">

# 🏆 TechThon • TAEF-AI Platform
### Automated Software Project Auditor & Next-Gen Hackathon Hub in MENA

[![Platform Status](https://img.shields.io/badge/Platform-Live-success?style=for-the-badge&logo=vercel)](https://techthon-frontend.vercel.app/)
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

## 🌟 Core Platform Pillars & Capabilities

| Core Pillar | Description & Operational Scope | Key Implementation Detail |
| :--- | :--- | :--- |
| **🏆 Hackathons Hub** | Real-world coding challenges, competitions, and transparent prize distributions. | Automated developer workflow and team management. |
| **🤖 TAEF-AI Auditor** | Evaluates repositories with zero human bias, combining deterministic AST scanning with ground-truth LLM analysis. | AST facts generation and multi-tier code evaluations. |
| **🔍 Evidence Verifier** | Cross-references every AI claim against physical filesystem files, systematically eliminating hallucinations. | Strict file-path verification on disk. |
| **⚡ Dual Runtime Engine** | **Cloud AI:** Supports OpenRouter API (Qwen 2.5 72B, Claude 3.5 Sonnet, Llama 3.3).<br/>**On-Premise Privacy:** Users can toggle to Local Ollama (`localhost:11434`) for 100% offline, free, zero-leak auditing. | Flexible execution runtime based on user privacy requirements. |
| **🔑 Bring Your Own Key (BYOK)** | Zero-cost server footprint by empowering users to use their personal API keys directly. | Client-injected API credentials with zero server storage. |
| **📊 Executive Reports** | Generates in-depth architecture telemetry, security observations, actionable roadmaps, and downloadable official PDF audit certificates. | Automated dynamic client-side report generation. |

---

## 🛠️ High-Level Architecture & Tech Stack

| Layer | Technology Stack | Architecture Role & Specifications |
| :--- | :--- | :--- |
| **Frontend** | Next.js 14+ (App Router), TypeScript, Tailwind CSS, Framer Motion, Axios, jsPDF | Vercel Edge Network deployment with client-side PDF export and reactive UI state. |
| **Backend** | Python 3.12, FastAPI, SQLAlchemy 2.0, PyJWT, SlowAPI, Uvicorn | Production REST API on Render Cloud Service with JWT auth, rate limiting, and async pipelines. |
| **Database** | PostgreSQL 16 (Neon.tech), Connection Pooling, Alembic | Serverless relational data store for User Profiles, RBAC, Hackathons, and Audit Telemetry. |
| **AI Inference** | OpenRouter API (Cloud) + Local Ollama (Offline) + AST Verifier Engine | Hybrid inference layer with deterministic code parsing and evidence matching. |
| **Security Suite** | Custom Stream Sanitization & Security Middleware | Zip bomb & Zip slip defense, stream upload limits (Max 50MB), strict CORS, and SSRF protection. |

---

## 📊 Live Verification & Pipeline Flow

| Stage | Pipeline Phase | Operations & Security Enforcement |
| :---: | :--- | :--- |
| **01** | **Upload ZIP Archive** | Enforces stream file size ceiling (Max 50MB). |
| **02** | **Security & Sanitization** | Verifies compression ratio (`<15:1`) and prevents path traversal / Zip slip attacks. |
| **03** | **Deterministic Static Scan** | Extracts AST facts, dependency trees, and Monorepo framework structures. |
| **04** | **Concurrent AI Auditing** | Executes parallel LLM batches utilizing `asyncio.gather` for rapid evaluation. |
| **05** | **Evidence Verification** | Cross-matches AI citations against physical codebase files residing on disk. |
| **06** | **Executive PDF & Report** | Delivers real-time token metering, scoring matrices, actionable recommendations, and official PDF certificates. |

---

## 📄 Documentation Links

| Document | Focus & Coverage |
| :--- | :--- |
| **[System Architecture Blueprint](./ARCHITECTURE.md)** | Deep dive into design patterns, concurrency models, and database schemas. |
| **[Security & Hardening Policy](./SECURITY.md)** | Comprehensive report on applied enterprise-grade security measures. |

---

## 🏢 Contact & Corporate Partnerships

Are you an enterprise looking to host automated hackathons or scout top-ranked tech talent in Turkey and the Middle East?

| Channel | Details |
| :--- | :--- |
| **🌐 Official Website** | [https://techthon-frontend.vercel.app/](https://techthon-frontend.vercel.app/) |
| **✉️ Inquiries & Partnerships** | `contact@techthon.io` / `admin@techthon.io` |
| **📍 Headquarters** | Istanbul, Turkey |

---

<div align="center">
<sub>TECHTHON • TAEF-AI © 2026. All rights reserved. Proprietary software showcase.</sub>
</div>
