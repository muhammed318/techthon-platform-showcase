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
| **06** | Client $\rightarrow$ Server | Final payload retrieval via `GET /projects/{id}` upon `SUCCESS` state. | `200 OK` Full Report Payload |

---

### 1.2 Deterministic Grounding & Anti-Hallucination Engine
To solve stochastic inconsistency and model hallucinations during automated source-code auditing:

| Strategy | Mechanism & Operational Scope |
| :--- | :--- |
| **Coupled Scanning** | Merges a deterministic static analyzer (AST & Regex Scanner) with probabilistic LLM reasoning. |
| **Evidence Verifier Engine** | Deterministic post-processing pipeline that parses structured JSON output and programmatically cross-checks every cited relative file path against the physical filesystem. |

#### Mathematical Penalty Adjustment Formula:
Overall audit confidence is algorithmically adjusted if unverified or hallucinated paths are detected:

$$\text{Final Confidence} = \text{Initial Confidence} \times \left( \frac{\text{Verified Evidence Claims}}{\text{Total Evidence Claims}} \right)$$

---

### 1.3 Bring-Your-Own-Key (BYOK) & Zero-Cost Model

| Operational Mode | Implementation | Business & Privacy Impact |
| :--- | :--- | :--- |
| **Zero-Cost Compute Footprint** | Ingests user's personal OpenRouter API key directly from encrypted client-side headers/payloads (`localStorage`). | Bills inference costs directly to the user; $0 server compute overhead. |
| **On-Premise Privacy Fallback** | Zero-leak fallback to Local Ollama (`http://localhost:11434`). | Runs inference 100% offline on client hardware with zero outbound network traffic. |

---

## 2. System Architecture & Topology

| Topology Node | Hosting & Environment | Core Responsibilities & Handshake Protocols |
| :--- | :--- | :--- |
| **Client Browser** | Next.js on Vercel | TechThon Branding, Client-Side JWT Auth, `localStorage` BYOK Key Manager. |
| **FastAPI Gateway** | Render Web Service | Rate Limiter (`SlowAPI`), Stream Ingestion (50MB Max), Zip Bomb / Zip Slip Guard. |
| **Database Tier** | Neon PostgreSQL 16 | Connection Pool (`20 + 10`), `DeclarativeBase` ORM Models, Automated Alembic Schemas. |
| **Audit Worker Engine** | Dedicated Background Worker | AST & Dependency Scanner, Dynamic Context Batcher, Evidence Verification Pipeline. |
| **Cloud AI Gateway** | OpenRouter API Gateway | Qwen 2.5 72B / Claude 3.5 Sonnet, Bounded Concurrency (`x3 Semaphore`). |
| **Local AI Engine** | Local Ollama Instance | 100% Air-Gapped Offline Inference (`localhost:11434`), Zero Outbound Network. |

---

## 3. Backend Engineering Specification

### 3.1 Framework & Concurrency Layer
| Component | Technology | Role & Optimization |
| :--- | :--- | :--- |
| **Runtime** | Python 3.12 | CPython optimization & native asynchronous task grouping. |
| **ASGI Server** | Uvicorn | High-throughput asynchronous request worker management. |
| **Data Validation** | Pydantic v2 | Rust-backed core enforcing strict request/response data contracts. |

---

### 3.2 Database Pooling & Migration Lifecycle
- **Host:** Neon Serverless PostgreSQL.
- **Session Pooling Parameters:**
  - `pool_size=20`, `max_overflow=10`: High-concurrency traffic tolerance.
  - `pool_pre_ping=True`: Active heartbeat health checks to eliminate stale cloud connections.
  - `pool_recycle=1800`: Connection cycling every 30 minutes.
- **Migrations:** Programmatically tracked and executed via **Alembic**, dynamically linked with Pydantic runtime models for zero-downtime schema evolution.

---

### 3.3 LLM Orchestration & Dynamic Batching
| Feature | Implementation Specification |
| :--- | :--- |
| **Aggregator Gateway** | OpenRouter API for multi-model dynamic fallback routing. |
| **Structured JSON Mode** | Enforces `response_format: {"type": "json_object"}` with AST regex fallback extractor. |
| **Concurrent Batching** | `asyncio.gather` bounded by `asyncio.Semaphore(3)`, processing 3 source batches simultaneously; **reduces monorepo audit latency by 70%** while avoiding 429 rate limits. |

---

### 3.4 Hardening & Ingestion Defenses

| Security Layer | Defense Mechanism | Threshold / Rule Enforced |
| :--- | :--- | :--- |
| **Zip Bomb Defense** | Compression ratio inspection | Compression ratio capped at $\le 15:1$ with a **250 MB** uncompressed expansion limit. |
| **Zip Slip Mitigation** | Canonical path validation | Validates `target_path.startswith(extract_dir)` neutralizing path traversal attacks. |
| **Traffic Throttling** | `SlowAPI` Middleware | Enforces **5 uploads/min** and **60 queries/min** rate limits. |
| **SSRF Prevention** | Loopback validation | Restricts local Ollama status checks strictly to loopback addresses (`localhost`, `127.0.0.1`). |

---

## 4. Frontend Engineering Specification

### 4.1 Architecture & Rendering Strategy
- **Framework:** Next.js (App Router) built on React 19 and TypeScript (`strict: true`).
- **Rendering Strategy:** Static Generation (SSG) across landing pages paired with `dynamic = 'force-dynamic'` for runtime-parameterized routes (`/projects/[id]`).

---

### 4.2 UI/UX System & State Management
- **Design Tokens:** Tailwind CSS v4 running native Dark Mode.
- **Corporate Color Palette:**
  - **Primary Royal Navy:** `#1e3a8a`
  - **Electric Tech Blue:** `#60a5fa`
  - **Background Slate:** `#070b14`
- **Motion Layer:** Framer Motion handling layout animations and multi-step progress steppers.
- **Icons:** SVG assets provided by `Lucide-React`.
- **State Management:** Decoupled component-level state isolation utilizing native React primitives (`useState`, `useEffect`) paired with secure `localStorage` abstraction layers.

---

### 4.3 Networking & Error Boundary
- **HTTP Client:** Unified Axios instance.
- **Global Error Interceptor:**

| Status Code | Intercepted Scenario | Handled UI Action |
| :---: | :--- | :--- |
| `401 / 403` | Unauthorized / Forbidden | Session expiration and access control enforcement. |
| `413` | Payload Too Large | Payload limit breach notification ($> 50\text{ MB}$). |
| `429` | Too Many Requests | Rate-limiting backoff notification. |

---

### 4.4 Zero-Load PDF Compilation
- **Stack:** `html2canvas` + `jsPDF`.
- **Implementation:** Dynamically parses and renders DOM nodes into high-resolution A4 PDFs locally within the client's browser. This guarantees zero server-side compute overhead during report export.

---

## 5. Infrastructure & Cloud Deployment

| Component | Platform / Host | Tier | Responsibility |
| :--- | :--- | :--- | :--- |
| **Frontend UI** | Vercel Global Edge Network | Production | CDN Caching, Edge Routing, Static Assets |
| **API Backend** | Render Web Service | Free / Standard | Ingestion, Task Workers, AST Analysis |
| **Database** | Neon Serverless | Free (0.5 GB Serverless) | User State, Hackathons, Persisted Audits |
| **Cloud AI Gateway** | OpenRouter API | BYOK Pay-as-you-go | Model Routing (Qwen, Claude, Llama) |
| **Local AI Engine** | Client Hardware (Ollama) | On-Premise | 100% Air-Gapped Local Inference |

---

## 6. Benchmarks & Performance Telemetry

| Performance Metric | Measured Output | Operational Significance |
| :--- | :--- | :--- |
| **Mean Execution Time** | **12–18 seconds** | Complete audit of full-stack repository (50–80 source files) via concurrent batching. |
| **Memory Footprint** | **~110 MB baseline $\rightarrow$ ~250 MB peak** | Measured during multi-part decompression and AST scanning on Render. |
| **Infrastructure Cost** | **$0.00 / month** | Net operating infrastructure cost for the entire AI platform. |
| **User Inference Cost** | **$0.012 – $0.02** | Average OpenRouter cost (using Qwen 2.5 72B) per full repository audit. |

---

<div align="center">
  <sub>TechThon • TAEF-AI Architecture Blueprint • Production Specification Version 1.4.0</sub>
</div>
