# ⚙️ Master Technical Specification & Architecture Blueprint

<div align="center">

[![Platform](https://img.shields.io/badge/Platform-TechThon%20•%20TAEF--AI-blue?style=for-the-badge)](https://techthon-frontend.vercel.app/)
[![Architecture](https://img.shields.io/badge/Architecture-Decoupled%20SOA-indigo?style=for-the-badge)](#)
[![Backend](https://img.shields.io/badge/Backend-FastAPI%20%7C%20Python%203.12-success?style=for-the-badge)](#)
[![Frontend](https://img.shields.io/badge/Frontend-Next.js%20%7C%20TypeScript-black?style=for-the-badge)](#)
[![Database](https://img.shields.io/badge/Database-Neon%20PostgreSQL-336791?style=for-the-badge)](#)

</div>

---

## 📑 Table of Contents
- [1. Core Architectural Paradigms](#1-core-architectural-paradigms)
  - [1.1 Asynchronous Background Pipeline](#11-asynchronous-background-pipeline)
  - [1.2 Deterministic Grounding & Anti-Hallucination Engine](#12-deterministic-grounding--anti-hallucination-engine)
  - [1.3 Bring-Your-Own-Key (BYOK) & Zero-Cost Model](#13-bring-your-own-key-byok--zero-cost-model)
- [2. System Architecture & Topology](#2-system-architecture--topology)
- [3. Backend Engineering Specification](#3-backend-engineering-specification)
  - [3.1 Framework & Concurrency Layer](#31-framework--concurrency-layer)
  - [3.2 Database Pooling & Migration Lifecycle](#32-database-pooling--migration-lifecycle)
  - [3.3 LLM Orchestration & Dynamic Batching](#33-llm-orchestration--dynamic-batching)
  - [3.4 Hardening & Ingestion Defenses](#34-hardening--ingestion-defenses)
- [4. Frontend Engineering Specification](#4-frontend-engineering-specification)
  - [4.1 Architecture & Rendering Strategy](#41-architecture--rendering-strategy)
  - [4.2 UI/UX System & State Management](#42-uiux-system--state-management)
  - [4.3 Networking & Error Boundary](#43-networking--error-boundary)
  - [4.4 Zero-Load PDF Compilation](#44-zero-load-pdf-compilation)
- [5. Infrastructure & Cloud Deployment](#5-infrastructure--cloud-deployment)
- [6. Benchmarks & Performance Telemetry](#6-benchmarks--performance-telemetry)

---

## 1. Core Architectural Paradigms

The platform is designed as a **Hybrid-Cloud & Local Privacy-First System**, built upon the principles of **Separation of Concerns (SoC)**, zero-downtime execution, and deterministic validation.

---

### 1.1 Asynchronous Background Pipeline
To eliminate cloud gateway timeouts (e.g., Vercel and Render HTTP boundaries):
- **Pattern:** Non-blocking **Fire-and-Forget** workflow utilizing FastAPI's native `BackgroundTasks`.

| Step | Initiator & Target | Operation & Protocol | Execution Metric |
| :---: | :--- | :--- | :--- |
| **01** | Client $\rightarrow$ Server | Dispatches project archive via `POST /upload/`. | Initial stream ingress |
| **02** | Ingestion Node | Fast I/O stream decompression & database audit initialization as `QUEUED`. | Low-latency I/O |
| **03** | Server $\rightarrow$ Client | Dispatches immediate `202 Accepted` response, releasing socket. | **~300ms Gateway Return** |
| **04** | Worker Queue | Isolated worker executes AST scanning & concurrent LLM analysis. | Background execution |
| **05** | Client $\rightarrow$ Server | Adaptive short-polling loop via `GET /projects/{id}/status`. | Real-time percentage streaming |
| **06** | Client $\rightarrow$ Server | Final payload retrieval via `GET /
