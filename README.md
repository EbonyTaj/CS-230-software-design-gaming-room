# CS-230 — The Gaming Room: Software Design

> Multi-platform architecture & design document for **Draw It or Lose It** (client: The Gaming Room).  
> Focus: secure, scalable, cloud-ready design with clear platform trade-offs.

**GitHub description (sidebar):**  
CS-230 design artifact for The Gaming Room—multi-platform architecture, security, and testing plan.

---

## Artifact

- **Final design document (Project Three):**
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
**Ask:** Expand **Draw It or Lose It** from Android-only to a **multi-platform** web+mobile experience that supports **real-time rounds**, **unique names for games/teams**, **one active game per team**, and **fast, fair gameplay**—with low latency and high availability.

### 2) What I did particularly well
- **Separation of concerns** across client, API, persistence, security, and deployment.
- **Concrete platform trade-offs** (Linux/Mac/Windows/mobile) with a clear recommendation.
- A **cross-platform test matrix** and **SLO footer** to make performance and reliability measurable.
- **Idempotent** write operations + a consistent **error model** to simplify retries and clients.

### 3) What in the design-doc process helped when coding
Treating the doc as a **map** reduced rework. Endpoints, DTOs, auth roles (USER/ADMIN), timing rules, and error schema were decided first, so building REST resources and WebSocket events was straightforward. Validations and **uniqueness constraints** were defined up front, which made tests predictable.

### 4) What I would revise and how
I’d deepen the **capacity & cost model**:
- Add load assumptions (CCU, RPS, cache hit ratios) and perf budgets per endpoint/socket event.
- Include a minimal **k6/Locust** script and a short **Runbook** (alerts, on-call, rollback) to tighten operability.

### 5) Interpreting user needs & why it matters
User goals—**fast rounds**, **fair play**, **no duplicate names**, **access anywhere**—became **design choices**:
- Server-authoritative timing; **WebSockets** for round updates; image fully revealed at **t = 30s** in each 60s round.
- API + DB enforce uniqueness; single active game per team.
- **Web client** for broad reach with mobile parity. Centering users preserves trust and retention.

### 6) My design approach & future techniques
- **Feynman pass:** explain simply (who talks to whom, why).
- **Quality gates early:** SLOs, authN/authZ, error schema, test matrix before coding.
- **Traceability:** requirement → design choice → test.
- **Operate what you build:** health checks, logs/metrics, deploy/rollback paths.  
**Next time:** start with a **walking skeleton**, attach **performance budgets**, iterate with **measurements**.

---

## Highlights (aligned to the submitted design)

- **Architecture:** Client–server; REST for CRUD/config, **WebSockets** for real-time round updates.
- **Game rules & identity:** Server-authoritative state; **unique names enforced** for games/teams; **one active game per team**.
- **Round timing:** Four 60s rounds; image fully revealed at **t = 30s**; steady cadence for fairness.
- **Data & idempotency:** Create/update/delete are idempotent; consistent JSON errors; safe client retries.
- **Security:** Token auth (AuthN), role-based access (USER/ADMIN, AuthZ), input validation, least privilege.
- **Ops & reliability:** Health checks, structured logs, metrics; cloud-deployable; rollback-friendly.
- **Accessibility/UX:** Keyboard + touch parity; baseline **WCAG AA** practices.
- **Performance guardrails (SLOs):** **p95 < 150 ms**, **99.9%** availability; **RPO 15 min**, **RTO 4 hr**.

---

## Cross-Platform Test Matrix
- **Browsers (Desktop):** Edge (LTS), Chrome (current & −1), Firefox (current), Safari (current & Tech Preview on macOS)
- **Mobile Web:** iOS Safari/WebKit (current & −1), Android Chrome (API 30+), orientations: portrait & landscape
- **Inputs:** Touch **and** keyboard supported; socket reconnect/timeout paths verified under nominal latency.

---

## Memory vs. Storage (how I keep it smooth)
- **RAM:** Decode to display size; small LRU (prev/current/next); background I/O; cancelable work; texture compression (BC/ETC2/ASTC).  
- **Storage:** Object storage + CDN; display-sized derivatives (JPEG/WebP/AVIF); content-hashed filenames; small on-device cache; HTTPS + checksums.  
This keeps 60-FPS feel while the library scales (~200 HD images ≈ 1.6–2.0 GB incl. overhead).

---

## Recommendation Snapshot (what I’d ship)
- **Host on Linux (Ubuntu LTS)** for cost, maturity, and container ecosystem; Windows Server only if AD/.NET requires it.  
- **Three-tier, stateless web:** CDN/WAF → ingress/LB → app pods → PostgreSQL + Redis + object storage.  
- **Security:** HTTPS, OAuth2/OIDC, short-lived JWTs, CSRF/CORS, secrets vault, scans, WAF/rate limiting, SIEM alerts.  
- **Distributed:** WebSockets primary; SSE/polling fallback; idempotent APIs; retries with backoff; health checks & circuit breakers.

---

## How to Review (5-Minute Checklist)
1. **Executive Summary** — client, goals, constraints; SLOs (**p95 < 150 ms; 99.9% availability; RPO 15 min; RTO 4 hr**).  
2. **Architecture Overview** — client → API → services → data; REST + WebSockets.  
3. **Requirements → Design Trace** — uniqueness, fairness, multi-platform → keys/timing/web client.  
4. **Security Model** — token AuthN, role-based AuthZ, validation, consistent JSON errors, admin/health routes isolated.  
5. **Data & Identity** — API + DB enforce uniqueness; idempotent writes.  
6. **Game Timing & Fairness** — server is source of truth; clients render only.  
7. **Test Matrix & A11y** — listed browsers/devices; WCAG-AA basics covered.  
8. **Deployability & Ops** — health checks, logs/metrics, rollback path, cloud-readiness.  
9. **Cost/Time & Skills** — one TS/React client; QA/device-farm budget; perf/a11y/WebSockets/test-automation skills.  
10. **Risks & Next Steps** — capacity/cost model, load script, runbook.

---

## Tech & Tools Mentioned
Java 17 • Dropwizard/JAX-RS (Jersey) • DTOs (JSON) • WebSockets • Token/role auth • CI/CD • Cloud deploy • Playwright (tests)

---

## Submission (for LMS)
- **Repository URL:** `https://github.com/EbonyTaj/cs230-software-design-gaming-room`
- **Artifact path (in repo):** `artifacts/CS 230 Project Three Software Design_EbAnderson.pdf`
- **README:** This file includes the required journal reflection answering all prompt questions.


