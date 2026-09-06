# 🛡️ Enterprise Security Policy & Hardening Specifications

<div align="center">

[![Security Rating](https://img.shields.io/badge/Security_Score-A%2B%20Enterprise-emerald?style=for-the-badge&logo=shield)](https://techthon-frontend.vercel.app/)
[![OWASP Compliance](https://img.shields.io/badge/OWASP_Top_10-Compliant-blue?style=for-the-badge)](#)
[![Data Privacy](https://img.shields.io/badge/Data_Privacy-Zero_Telemetry-purple?style=for-the-badge)](#)
[![Vulnerability Status](https://img.shields.io/badge/Known_Vulnerabilities-0%20(Audited)-success?style=for-the-badge)](#)

</div>

---

## 📌 Executive Security Overview

The **TechThon • TAEF-AI** platform processes untrusted third-party codebases, enterprise repositories, and competitive hackathon submissions. Security is engineered into the foundation through a **Defense-in-Depth (DiD)** strategy.

This document outlines the active defense mechanisms, cryptographic policies, and threat mitigations implemented across the platform.

---

## 🔐 1. Cryptographic Authentication & RBAC

| Security Layer | Implementation Standard | Threat Mitigation Target |
| :--- | :--- | :--- |
| **1.1 Stateless JWT Authentication** | Algorithm: `HS256` utilizing cryptographically secure, 256-bit entropy keys (`openssl rand -hex 32`).<br/>• **Token Expiration:** Fixed 7-day expiration lifecycle with declarative claims validation (`sub`, `role`, `username`).<br/>• **Library:** Powered natively by `PyJWT[crypto]` backed by C/Rust primitives. | Mitigates legacy side-channel attacks and eliminates server-side session hijacking. |
| **1.2 Password Hashing Standard** | Engine: Native, salt-injected `bcrypt`.<br/>• **Buffer Safety:** Strict 72-byte string truncation pre-processing applied before hashing. | Eliminates memory buffer overflow vulnerabilities in bcrypt backends. |
| **1.3 RBAC & IDOR Mitigation** | Authorization layers enforced at FastAPI dependency boundaries (`require_admin`, `get_current_user`).<br/>• **IDOR / BOLA Prevention:** Audit asset queries enforce strict tenant isolation via canonical ownership validation (`verify_project_ownership`). | Prevents horizontal privilege escalation and unauthorized tenant asset enumeration. |

---

## 📦 2. Archive Ingestion & Memory Exhaustion Defense

Uploading compressed archives (`.zip`) presents significant attack vectors (Decompression Bombs, Path Traversal, and Disk Exhaustion). TAEF-AI neutralizes these before extraction:

| Pipeline Step | Defense Check | Condition / Threshold | Enforced Reaction |
| :---: | :--- | :--- | :--- |
| **01** | **Upload Stream Metering** | Max archive payload check | Streaming byte counter $> 50\text{ MB} \rightarrow$ **Immediate Socket Termination (`413 Payload Too Large`)** |
| **02** | **Pre-Extraction Inspection** | Header metadata inspection | Compression ratio $> 15:1 \rightarrow$ **Abort Extraction (`400 Bad Request / Zip Bomb`)** |
| **03** | **Canonical Path Resolver** | Filesystem boundary check | Target path outside sandbox $\rightarrow$ **Block File Write (`400 Bad Request / Zip Slip`)** |
| **04** | **Sandboxed Storage** | Isolated ephemeral write | Write permitted to isolated directory `/extracted/{uuid}/` |

---

### 2.1 Zip Bomb & Path Traversal Mitigations

| Protection Metric | Configured Limit / Implementation | Security Rationale |
| :--- | :--- | :--- |
| **Compression Ratio Guard** | $\le 15:1$ | Analyzes archive metadata headers without extracting payload to detect recursive decompression traps. |
| **Absolute Expansion Ceiling** | **250 MB** | Maximum allowed uncompressed byte expansion on disk. |
| **File Count Ceiling** | **5,000 entities** | Caps total indexed files inside any archive to prevent inode exhaustion. |
| **Zip Slip Canonical Check** | `if not str(target_path.resolve()).startswith(str(sandbox_dir.resolve())): raise SecurityException("Path Traversal Attack Detected.")` | Canonical path validation neutralizing directory traversal attacks. |

---

## 🤖 3. Grounded AI & Anti-Hallucination Integrity

Large Language Models are inherently probabilistic and prone to hallucinating non-existent files or packages. TAEF-AI treats model responses as untrusted data:

| Defense Component | Formula & Engineering Mechanism |
| :--- | :--- |
| **Grounding Formula** | $\text{Final Confidence} = \text{Model Confidence} \times \left( \frac{\text{Verified Filesystem Citations}}{\text{Total Citations}} \right)$ |
| **Deterministic Cross-Referencing** | An isolated post-processing algorithm parses all file paths cited in the model's audit evidence and deterministically verifies their physical presence on disk. |
| **Penalty Enforcement** | Hallucinated claims are flagged as `unverified: false`, and overall audit confidence is mathematically degraded. |
| **Prompt Injection Defense** | Input code is formatted within strict XML-like structural fences (`===== FILE: {name} =====`), disallowing instruction overriding from malicious comments inside scanned files. |

---

## 🌐 4. Network, Edge & Infrastructure Defenses

| Infrastructure Domain | Implementation Specification |
| :--- | :--- |
| **4.1 SSRF Prevention** | Local Ollama status health checks (`GET /models/local?url=...`) are strictly locked at the socket level to loopback interfaces:<br/>• **Allowed Hosts:** `localhost`, `127.0.0.1`, `::1`.<br/>• **Blocked Subnets:** All outbound requests to arbitrary private subnets (e.g., `10.0.0.0/8`, `192.168.0.0/16`, AWS metadata `169.254.169.254`) are blocked at the gateway. |
| **4.2 DDoS & Rate Limiting** | Throttling engine powered by `SlowAPI` with in-memory state tracking:<br/>• `POST /upload/`: **5 requests/minute** per authenticated client/IP.<br/>• **Query Endpoints:** **60 requests/minute** per client. |
| **4.3 Strict Origin Isolation (CORS)** | Wildcards (`"*"`) are explicitly forbidden in production when credentials are included. Origins are locked to verified hostnames:<br/>• `https://techthon1.vercel.app`<br/>• `https://techthon.io` / `https://techthon.tr` |
| **4.4 SQL Injection Neutralization** | The platform exclusively employs **SQLAlchemy 2.0 ORM** with parameterized prepared statements. Direct string interpolation or raw SQL concatenation is strictly forbidden across the codebase. |

---

## 🛡️ 5. Privacy & Zero-Knowledge Architecture (BYOK)

| Privacy Mode | Operational Lifecycle | Data Security Guarantee |
| :--- | :--- | :--- |
| **Bring-Your-Own-Key (BYOK)** | User OpenRouter keys are stored solely in the client's local browser storage (`localStorage`). | Keys are transmitted ephemerally via HTTPS headers and are never logged, saved, or persisted in the database. |
| **On-Premise Air-Gapped Mode** | Enterprise clients can execute audits via Local Ollama (`localhost:11434`). | Guarantees that proprietary source code never leaves the developer's physical machine. |

---

## 🔍 6. Security Vulnerability Reporting

We welcome responsible security disclosures. If you discover a potential vulnerability within the TechThon platform, please report it immediately:

| Contact Method | Direct Channel | Policy & SLA |
| :--- | :--- | :--- |
| **Official Security Email** | `security@techthon.io` | Acknowledged within **24 hours**. |
| **Remediation SLA** | Automated triage workflow | Patches dispatched within **72 hours**. |

> *Please do not publicly disclose vulnerabilities until an official patch has been deployed.*

---

<div align="center">
<sub>TechThon • TAEF-AI Security Compliance Document • Production Grade</sub>
</div>
