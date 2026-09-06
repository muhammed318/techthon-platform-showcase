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

### 1.1 Asynchronous Background Pipeline
To eliminate cloud gateway timeouts (e.g., Vercel and Render HTTP boundaries):
- **Pattern:** Non-blocking **Fire-and-Forget** workflow utilizing FastAPI's native `BackgroundTasks`.
- **Workflow:**
  1. The client dispatches project archives via `POST /upload/`.
  2. The server executes fast, I/O-bound stream decompression and initializes the audit record as `QUEUED` in the database.
  3. An immediate `202 Accepted` response returns within **~300ms**, releasing the network socket.
  4. The heavy computational pipeline (AST scanning & parallel LLM analysis) runs in an isolated background worker thread.
  5. The client maintains an adaptive short-polling loop via `GET /projects/{id}/status` to stream real-time percentage updates directly to the UI.

```text
Client Browser            FastAPI Ingestion Node         Background Worker Queue        PostgreSQL (Neon)
      │                             │                               │                         │
      ├── POST /upload/ (ZIP) ─────►│                               │                         │
      │                             ├── [Stream Decompression]      │                         │
      │                             ├── [Init State: QUEUED] ───────┼────────────────────────►│
      │◄── 202 Accepted (300ms) ────┤                               │                         │
      │                             └─── Dispatch Task ────────────►│                         │
      │                                                             ├── Phase 1: Static Scan  │
      ├── GET /projects/{id}/status ───────────────────────────────►├── Phase 2: AI Auditing  │
      │◄── { progress: 60% } ───────────────────────────────────────┼── Phase 3: Evidence Ver.│
      │                                                             └── [Final State: SUCCESS]┤
      ├── GET /projects/{id} ────────────────────────────────────────────────────────────────►│
      │◄── 200 OK (Full Report Payload) ──────────────────────────────────────────────────────┘
1.2 Deterministic Grounding & Anti-Hallucination Engine
To solve stochastic inconsistency and model hallucinations during automated source-code auditing:
Coupled Scanning: Merges a deterministic static analyzer (AST & Regex Scanner) with probabilistic LLM reasoning.
Evidence Verifier Engine: A deterministic post-processing pipeline that parses structured JSON output from the model and programmatically cross-checks every cited relative file path against the actual physical filesystem.
Mathematical Penalty Adjustment: Overall audit confidence is algorithmically adjusted if unverified or hallucinated paths are detected:
Final Confidence
=
Initial Confidence
×
(
Verified Evidence Claims
Total Evidence Claims
)
Final Confidence=Initial Confidence×( 
Total Evidence Claims
Verified Evidence Claims
​
 )
1.3 Bring-Your-Own-Key (BYOK) & Zero-Cost Model
Zero-Cost Compute Footprint: The backend ingests the user's personal OpenRouter API key directly from encrypted client-side runtime headers/payloads (localStorage), billing inference costs directly to the user.
On-Premise Privacy Fallback: For enterprise clients with strict confidentiality policies, the system provides a zero-leak fallback to Local Ollama (http://localhost:11434), running inference entirely offline on the client's local hardware at zero cost.
2. System Architecture & Topology
code
Text
┌───────────────────────────────────┐
                              │     Client Browser (Next.js)      │
                              │  - TechThon Corporate Branding    │
                              │  - Client-Side JWT Auth           │
                              │  - LocalStorage BYOK Key Manager  │
                              └─────────────────┬─────────────────┘
                                                │ (HTTPS / REST API)
                                                ▼
                              ┌───────────────────────────────────┐
                              │     FastAPI Gateway (Render)      │
                              │  - Rate Limiter (SlowAPI)         │
                              │  - Streaming Ingestion (50MB Max) │
                              │  - Zip Bomb / Zip Slip Guard      │
                              └─────────┬───────────────┬─────────┘
                                        │               │
               ┌────────────────────────┘               └────────────────────────┐
               ▼                                                                 ▼
┌───────────────────────────────┐                             ┌──────────────────────────────────┐
│   PostgreSQL 16 DB (Neon)     │                             │      Audit Worker Engine         │
│ - Connection Pool (20 + 10)   │                             │ - AST & Dependency Scanner       │
│ - DeclarativeBase ORM Models  │                             │ - Dynamic Context Batcher        │
│ - Automated Alembic Schemas   │                             │ - Evidence Verification Pipeline │
└───────────────────────────────┘                             └─────────────────┬────────────────┘
                                                                                │
                                                      ┌─────────────────────────┴────────────────┐
                                                      ▼                                          ▼
                                       ┌─────────────────────────────┐            ┌─────────────────────────────┐
                                       │   OpenRouter API Gateway    │            │    Local Ollama Instance    │
                                       │ - Qwen 2.5 72B / Claude 3.5 │            │ - Offline 100% Air-Gapped   │
                                       │ - Bounded Concurrency (x3)  │            │ - Zero Outbound Network     │
                                       └─────────────────────────────┘            └─────────────────────────────┘
3. Backend Engineering Specification
3.1 Framework & Concurrency Layer
Runtime: Python 3.12 (CPython optimization & native asynchronous task grouping).
ASGI Server: Uvicorn handling asynchronous request workers.
Validation: Pydantic v2 (Rust-backed core) enforcing strict request/response data contracts.
3.2 Database Pooling & Migration Lifecycle
Host: Neon Serverless PostgreSQL.
Session Pooling:
pool_size=20, max_overflow=10: High-concurrency traffic tolerance.
pool_pre_ping=True: Active heartbeat health checks to eliminate stale cloud connections.
pool_recycle=1800: Connection cycling every 30 minutes.
Migrations: Programmatically tracked and executed via Alembic, dynamically linked with Pydantic runtime models for zero-downtime schema evolution.
3.3 LLM Orchestration & Dynamic Batching
Aggregator Gateway: OpenRouter API for multi-model fallback.
Structured JSON Mode: Enforces response_format: {"type": "json_object"} at the API boundary, supported by an AST regex fallback extractor.
Concurrent Batching: Employs asyncio.gather bounded by asyncio.Semaphore(3). This concurrency controller processes 3 source batches simultaneously, reducing multi-tiered monorepo audit latency by 70% while strictly avoiding upstream 429 Too Many Requests rate limits.
3.4 Hardening & Ingestion Defenses
Archive Ingestion Defense:
Zip Bomb Defense: Maximum compression ratio capped at 
≤
15
:
1
≤15:1
 with a 
250
 MB
250 MB
 uncompressed expansion limit.
Zip Slip Mitigation: Rigorous canonical path validation (target_path.startswith(extract_dir)) neutralizing path traversal attacks.
Traffic Throttling: SlowAPI enforcing 
5
 uploads/min
5 uploads/min
 and 
60
 queries/min
60 queries/min
.
SSRF Prevention: Restricts local Ollama status checks strictly to loopback addresses (localhost, 127.0.0.1).
4. Frontend Engineering Specification
4.1 Architecture & Rendering Strategy
Framework: Next.js (App Router) built on React 19 and TypeScript (strict: true).
Rendering: Static generation across static pages paired with dynamic = 'force-dynamic' for runtime-parameterized routes (/projects/[id]).
4.2 UI/UX System & State Management
Design Tokens: Tailwind CSS v4 running native Dark Mode.
Corporate Color Palette:
Primary Royal Navy: #1e3a8a
Electric Tech Blue: #60a5fa
Background Slate: #070b14
Motion Layer: Framer Motion handling layout animations and multi-step progress steppers.
Icons: SVG assets provided by Lucide-React.
State Management: Decoupled component-level state isolation utilizing native React primitives (useState, useEffect) paired with secure localStorage abstraction layers.
4.3 Networking & Error Boundary
HTTP Client: Unified Axios instance.
Global Error Interceptor: Normalizes upstream network and status errors into clean UI notifications:
401/403 
→
→
 Session expiration and access control enforcement.
413 
→
→
 Payload limit breach warnings (
>
50
 MB
>50 MB
).
429 
→
→
- `429` -> Rate-limiting backoff notifications.

### 4.4 Zero-Load PDF Compilation
- **Stack:** `html2canvas` + `jsPDF`.
- **Implementation:** Dynamically parses and renders DOM nodes into high-resolution A4 PDFs locally within the client's browser. This guarantees zero server-side compute overhead during report export.

---

## 5. Infrastructure & Cloud Deployment

| Component | Platform / Host | Tier | Responsibility |
|---|---|---|---|
| **Frontend UI** | Vercel Global Edge Network | Production | CDN Caching, Edge Routing, Static Assets |
| **API Backend** | Render Web Service | Free / Standard | Ingestion, Task Workers, AST Analysis |
| **Database** | Neon Serverless | Free (0.5 GB Serverless) | User State, Hackathons, Persisted Audits |
| **Cloud AI Gateway** | OpenRouter API | BYOK Pay-as-you-go | Model Routing (Qwen, Claude, Llama) |
| **Local AI Engine** | Client Hardware (Ollama) | On-Premise | 100% Air-Gapped Local Inference |

---

## 6. Benchmarks & Performance Telemetry

- **Mean Execution Time:** A full-stack repository (50–80 source files) completes in **12–18 seconds** via concurrent batching.
- **Memory Footprint:** The backend operates with a baseline of **~110 MB RAM**, peaking at **~250 MB RAM** during multi-part decompression and AST scanning.
- **Cost Efficiency:** Net operating infrastructure cost for the AI platform is **$0.00**. User inference costs via OpenRouter (utilizing Qwen 2.5 72B) average between **$0.012 to $0.02** per full repository audit.

---

<div align="center">
  <sub>TechThon • TAEF-AI Architecture Blueprint • Production Specification Version 1.4.0</sub>
</div>
```
