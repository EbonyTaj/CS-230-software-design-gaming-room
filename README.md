# CS-230 — The Gaming Room: Software Design

> Multi-platform architecture & design document for **Draw It or Lose It** (client: The Gaming Room).  
> Focus: secure, scalable, cloud-ready design with clear platform trade-offs.

**GitHub description (sidebar):**  
CS-230 design artifact for The Gaming Room—multi-platform architecture, security, and testing plan.

---

## Artifact

- **Final design document (Project Three — Revised):**  
  [Revision_CS 230 Project Three Software Design_EbAnderson.pdf](artifacts/Revision_CS%20230%20Project%20Three%20Software%20Design_EbAnderson.pdf)

- **Prior Project Three (original submission):**  
  [CS 230 Project Three Software Design_EbAnderson.pdf](artifacts/CS%20230%20Project%20Three%20Software%20Design_EbAnderson.pdf)

- **Prior version (Project Two):**  
  [CS 230 Project Two Software Design_EbAnderson.pdf](artifacts/CS%20230%20Project%20Two%20Software%20Design_EbAnderson.pdf)

- **Related journals (optional for context):**
  - [CS 230 Module Four Journal Software Application Requirements.pdf](artifacts/CS%20230%20Module%20Four%20Journal%20Software%20Application%20Requirements.pdf)
  - [CS230 Module Six Memory and Storage Management_EbAnderson.pdf](artifacts/CS230%20Module%20Six%20Memory%20and%20Storage%20Management_EbAnderson.pdf)
  - [Revision_CS230 Module Six Memory and Storage Management_EbAnderson.pdf](artifacts/Revision_CS230%20Module%20Six%20Memory%20and%20Storage%20Management_EbAnderson.pdf)

---

## Reflection (Module Eight Journal)

### 1) Client & software requirements (summary)
**Client:** *The Gaming Room*, a casual-game studio.  
**Ask:** Expand **Draw It or Lose It** from Android-only to a **multi-platform** web + mobile experience that supports **real-time rounds**, **unique names for games/teams**, **one active game per team**, and **fast, fair gameplay**—with low latency and high availability.

### 2) What I did particularly well
- **Separation of concerns** across client, **Application Programming Interface (API)**, persistence, security, and deployment.
- **Concrete platform trade-offs** (Linux/macOS/Windows/mobile) with a clear recommendation.
- A **cross-platform test matrix** and a **Service Level Objective (SLO)** footer to make performance and reliability measurable.
- **Idempotent** write operations + a consistent **error model** to simplify retries and clients.

### 3) What in the design-doc process helped when coding
Treating the doc as a **map** reduced rework. Endpoints, **Data Transfer Objects (DTOs)**, authentication/authorization (roles USER/ADMIN), timing rules, and error schema were decided first, so building REST resources and **WebSocket** events was straightforward. Validations and **uniqueness constraints** were defined up front, which made tests predictable.

### 4) What I would revise and how
I’d deepen the **capacity & cost model**:
- Add load assumptions (concurrent users, requests per second, cache hit ratios) and performance budgets per endpoint/socket event.
- Include a minimal **k6/Locust** script and a short **Runbook** (alerts, on-call, rollback) to tighten operability.

### 5) Interpreting user needs & why it matters
User goals—**fast rounds**, **fair play**, **no duplicate names**, **access anywhere**—became **design choices**:
- Server-authoritative timing; **WebSockets** for round updates; image fully revealed at **t = 30 s** in each 60 s round.
- API + database enforce uniqueness; single active game per team.
- **Web client** for broad reach with mobile parity. Centering users preserves trust and retention.

### 6) My design approach & future techniques
- **Feynman pass:** explain simply (who talks to whom, why).
- **Quality gates early:** SLOs, authentication/authorization, error schema, test matrix before coding.
- **Traceability:** requirement → design choice → test.
- **Operate what you build:** health checks, logs/metrics, deploy/rollback paths.  
**Next time:** start with a **walking skeleton**, attach **performance budgets**, iterate with **measurements**.

---

## Highlights (aligned to the submitted design)

- **Architecture:** Client–server; REST for **Create/Read/Update/Delete (CRUD)** and configuration, **WebSockets** for real-time round updates.
- **Game rules & identity:** Server-authoritative state; **unique names enforced** for games/teams; **one active game per team**.
- **Round timing:** Four 60 s rounds; image fully revealed at **t = 30 s**; steady cadence for fairness.
- **Data & idempotency:** Create/update/delete are idempotent; consistent **JavaScript Object Notation (JSON)** errors; safe client retries.
- **Security:** Token authentication, role-based access (USER/ADMIN), input validation, least privilege.
- **Ops & reliability:** Health checks, structured logs, metrics; cloud-deployable; rollback-friendly.
- **Accessibility/User Experience (A11y/UX):** Keyboard + touch parity; baseline **Web Content Accessibility Guidelines (WCAG) AA** practices.
- **Performance guardrails (SLOs):** **p95 < 150 ms**, **99.9%** availability; **Recovery Point Objective (RPO) 15 min**, **Recovery Time Objective (RTO) 4 hr**.

---

## Cross-Platform Test Matrix
- **Browsers (Desktop):** Edge **Long-Term Support (LTS)**, Chrome (current & −1), Firefox (current), Safari (current & Tech Preview on macOS)
- **Mobile Web:** iOS Safari/WebKit (current & −1), Android Chrome (**Application Programming Interface (API)** level 30+), orientations: portrait & landscape
- **Inputs:** Touch **and** keyboard supported; socket reconnect/timeout paths verified under nominal latency.

---

## Memory vs. Storage (how I keep it smooth)
- **Random Access Memory (RAM):** Decode to display size; small **Least Recently Used (LRU)** cache (prev/current/next); background I/O; cancelable work; texture compression (Block Compression (BC)/Ericsson Texture Compression 2 (ETC2)/Adaptive Scalable Texture Compression (ASTC)).  
- **Storage:** Object storage + **Content Delivery Network (CDN)**; display-sized derivatives (JPEG/WebP/AVIF); content-hashed filenames; small on-device cache; HTTPS + checksums.  
This keeps a smooth feel while the library scales (~200 HD images ≈ 1.6–2.0 GB including overhead).

---

## Recommendation Snapshot (what I’d ship)
- **Host on Linux (Ubuntu LTS)** for cost, maturity, and container ecosystem; Windows Server only if **Active Directory (AD)** or .NET requires it.  
- **Three-tier, stateless web:** CDN/**Web Application Firewall (WAF)** → ingress/load balancer → app pods → PostgreSQL + Redis + object storage.  
- **Security:** HTTPS, **OAuth 2.0/OpenID Connect (OIDC)**, short-lived **JSON Web Tokens (JWTs)**, **Cross-Site Request Forgery (CSRF)**/**Cross-Origin Resource Sharing (CORS)**, secrets vault, scans, WAF/rate limiting, **Security Information and Event Management (SIEM)** alerts.  
- **Distributed:** WebSockets primary; **Server-Sent Events (SSE)**/polling fallback; idempotent APIs; retries with backoff; health checks & circuit breakers.

---

## How to Review (5-Minute Checklist)
1. **Executive Summary** — client, goals, constraints; SLOs (**p95 < 150 ms; 99.9% availability; RPO 15 min; RTO 4 hr**).  
2. **Architecture Overview** — client → API → services → data; REST + WebSockets.  
3. **Requirements → Design Trace** — uniqueness, fairness, multi-platform → keys/timing/web client.  
4. **Security Model** — token authentication, role-based authorization, validation, consistent JSON errors, admin/health routes isolated.  
5. **Data & Identity** — API + database enforce uniqueness; idempotent writes.  
6. **Game Timing & Fairness** — server is source of truth; clients render only.  
7. **Test Matrix & Accessibility (A11y)** — listed browsers/devices; WCAG-AA basics covered.  
8. **Deployability & Operations (Ops)** — health checks, logs/metrics, rollback path, cloud-readiness.  
9. **Cost/Time & Skills** — one **TypeScript (TS)**/**React** client; **Quality Assurance (QA)**/device-farm budget; performance/accessibility/WebSockets/test-automation skills.  
10. **Risks & Next Steps** — capacity/cost model, load script, runbook.

---

## Tech & Tools Mentioned
Java 17 • Dropwizard/**Jakarta RESTful Web Services (JAX-RS)** (Jersey) • DTOs (JSON) • WebSockets • Token/role auth • **Continuous Integration/Continuous Delivery (CI/CD)** • Cloud deploy • Playwright (tests)

---

## Submission (for LMS)
- **Repository URL:** `https://github.com/EbonyTaj/cs230-software-design-gaming-room`  
- **Artifact path (in repo):** `artifacts/Revision_CS 230 Project Three Software Design_EbAnderson.pdf` (revised) and `artifacts/CS 230 Project Three Software Design_EbAnderson.pdf` (original)  
- **README:** This file includes the required journal reflection answering all prompt questions.

---

## Acronyms (first mention spelled out)
- AD — Active Directory  
- A11y — Accessibility  
- API — Application Programming Interface  
- API (Android) — Application Programming Interface (platform level)  
- ASTC — Adaptive Scalable Texture Compression  
- AVIF — AV1 Image File Format  
- BC — Block Compression  
- CDN — Content Delivery Network  
- CI/CD — Continuous Integration/Continuous Delivery  
- CORS — Cross-Origin Resource Sharing  
- CRUD — Create, Read, Update, Delete  
- CSRF — Cross-Site Request Forgery  
- DTO — Data Transfer Object  
- JAX-RS — Jakarta RESTful Web Services  
- JSON — JavaScript Object Notation  
- JWT — JSON Web Token  
- LRU — Least Recently Used  
- LMS — Learning Management System  
- LTS — Long-Term Support  
- OIDC — OpenID Connect  
- QA — Quality Assurance  
- RAM — Random Access Memory  
- RPO — Recovery Point Objective  
- RTO — Recovery Time Objective  
- SSE — Server-Sent Events  
- SLO — Service Level Objective  
- TS — TypeScript  
- UX — User Experience  
- WAF — Web Application Firewall  
- WCAG — Web Content Accessibility Guidelines
