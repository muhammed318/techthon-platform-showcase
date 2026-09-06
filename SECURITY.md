# 🛡️ Enterprise Security Policy & Hardening Specifications

<div align="center">

[![Security Rating](https://img.shields.io/badge/Security_Score-A%2B%20Enterprise-emerald?style=for-the-badge&logo=shield)](https://techthon-frontend.vercel.app/)
[![OWASP Compliance](https://img.shields.io/badge/OWASP_Top_10-Compliant-blue?style=for-the-badge)](#)
[![Data Privacy](https://img.shields.io/badge/Data_Privacy-Zero_Telemetry-purple?style=for-the-badge)](#)
[![Vulnerability Status](https://img.shields.io/badge/Known_Vulnerabilities-0%20(Audited)-success?style=for-the-badge)](#)

</div>

---

## 📌 Executive Security Overview

The **TechThon • TAEF-AI** platform processes untrusted third-party codebases, enterprise repositories, and competitive hackathon submissions. Security is not an afterthought—it is engineered into the foundation through a **Defense-in-Depth (DiD)** strategy.

This document outlines the active defense mechanisms, cryptographic policies, and threat mitigations implemented across the platform.

---

## 🔐 1. Cryptographic Authentication & RBAC

### 1.1 Stateless JWT Authentication
- **Algorithm:** `HS256` utilizing cryptographically secure, 256-bit entropy keys (`openssl rand -hex 32`).
- **Token Expiration:** Fixed 7-day expiration lifecycle with declarative claims validation (`sub`, `role`, `username`).
- **Cryptographic Library:** Powered natively by `PyJWT[crypto]` backed by C/Rust cryptographic primitives, mitigating legacy side-channel attacks.

### 1.2 Password Hashing Standard
- **Engine:** Native, salt-injected `bcrypt`.
- **Buffer Safety:** Strict 72-byte string truncation pre-processing applied before hashing to eliminate memory buffer overflow vulnerabilities.

### 1.3 Role-Based Access Control (RBAC) & IDOR Mitigation
- Fine-grained authorization layers enforced directly at the FastAPI dependency boundary (`require_admin`, `get_current_user`).
- **IDOR / BOLA Prevention:** Every audit asset query enforces strict tenant isolation via canonical ownership validation (`verify_project_ownership`). Users cannot enumerate, view, or purge audits belonging to other tenants.

---

## 📦 2. Archive Ingestion & Memory Exhaustion Defense

Uploading compressed archives (`.zip`) presents significant attack vectors (Decompression Bombs, Path Traversal, and Disk Exhaustion). TAEF-AI neutralizes these before extraction:

```text
Upload Stream (Max 50MB)
           │
           ▼
[ Streaming Byte Counter ] ──(> 50MB)──► Immediate Socket Termination (413 Payload Too Large)
           │
           ▼
[ Pre-Extraction Inspection ] ──(Ratio > 15:1)──► Abort Extraction (400 Bad Request / Zip Bomb)
           │
           ▼
[ Canonical Path Resolver ] ──(Path != Sandbox)──► Block File Write (400 Bad Request / Zip Slip)
           │
           ▼
[ Sandboxed Temporary Storage ]
2.1 Zip Bomb (Recursive Expansion) Protection
Compression Ratio Guard: Analyzes archive metadata headers without extracting payload. If the uncompressed-to-compressed ratio exceeds 15:1, the archive is aborted.
Absolute Expansion Ceiling: Total uncompressed size is capped at 250 MB.
File Count Ceiling: Total indexed files inside any archive cannot exceed 5,000 entities.
2.2 Zip Slip & Path Traversal Neutralization
Archive members are strictly extracted into isolated temporary sandboxes (/extracted/{uuid}/).
Every extracted target path is resolved canonically and validated using:
code
Python
if not str(target_path.resolve()).startswith(str(sandbox_dir.resolve())):
    raise SecurityException("Path Traversal Attack Detected.")
🤖 3. Grounded AI & Anti-Hallucination Integrity
Large Language Models are inherently probabilistic and prone to hallucinating non-existent files or packages. TAEF-AI treats model responses as untrusted data:
Final Confidence
=
Model Confidence
×
(
Verified Filesystem Citations
Total Citations
)
Final Confidence=Model Confidence×( 
Total Citations
Verified Filesystem Citations
​
 )
Deterministic Cross-Referencing: An isolated post-processing algorithm parses all file paths cited in the model's audit evidence and deterministically verifies their physical presence on disk.
Penalty Enforcement: Hallucinated claims are flagged as unverified: false, and overall audit confidence is mathematically degraded.
Prompt Injection Defense: Input code is formatted within strict XML-like structural fences (===== FILE: {name} =====), disallowing instruction overriding from malicious comments inside scanned files.
🌐 4. Network, Edge & Infrastructure Defenses
4.1 Server-Side Request Forgery (SSRF) Prevention
Local Ollama status health checks (GET /models/local?url=...) are strictly locked at the socket level to loopback interfaces:
Allowed Hosts: localhost, 127.0.0.1, ::1.
All outbound requests to arbitrary private subnets (e.g., 10.0.0.0/8, 192.168.0.0/16, AWS metadata 169.254.169.254) are blocked at the gateway.
4.2 Distributed Denial of Service (DDoS) & Rate Limiting
Throttling Engine: Powered by SlowAPI with in-memory state tracking:
POST /upload/: 5 requests/minute per authenticated client/IP.
Query Endpoints: 60 requests/minute per client.
4.3 Strict Origin Isolation (CORS Policy)
Wildcards ("*") are explicitly forbidden in production when credentials are included.
Origins are locked to verified hostnames:
https://techthon1.vercel.app
https://techthon.io / https://techthon.tr
4.4 SQL Injection Neutralization
The platform exclusively employs SQLAlchemy 2.0 ORM with parameterized prepared statements.
Direct string interpolation or raw SQL concatenation is strictly forbidden across the codebase.
🛡️ 5. Privacy & Zero-Knowledge Architecture (BYOK)
Bring-Your-Own-Key (BYOK): User OpenRouter keys are stored solely in the client's local browser storage (localStorage). Keys are transmitted ephemerally via HTTPS headers and are never logged, saved, or persisted in the database.
On-Premise Air-Gapped Mode: Enterprise clients can execute audits via Local Ollama (localhost:11434), guaranteeing that proprietary source code never leaves the developer's physical machine.
🔍 6. Security Vulnerability Reporting
We welcome responsible security disclosures. If you discover a potential vulnerability within the TechThon platform, please report it immediately:
Email: security@techthon.io
Response SLA: Critical vulnerabilities are acknowledged within 24 hours, with remediation dispatched within 72 hours.
Please do not publicly disclose vulnerabilities until an official patch has been deployed.
<div align="center">
<sub>TechThon • TAEF-AI Security Compliance Document • Production Grade</sub>
</div>
```
