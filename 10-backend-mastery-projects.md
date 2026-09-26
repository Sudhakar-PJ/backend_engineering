# Backend Mastery Projects

> **After T7.** This is where you build production-grade services from scratch — no tutorials, no hand-holding.
> 13 projects total. **Core 6** are the completion marker. **Optional 7** are extensions.

---

## How Mastery Projects Work

### Format (10D — Brief Spec, Blind Build, Reference Reveal)

Each project follows this protocol:

1. **Read the brief.** One paragraph description + capabilities checklist. That's all you get.
2. **Design and build it blind.** No hints during the build. Use everything from T1–T7.
3. **Submit your build.** When you consider it done, tell the AI instructor.
4. **Reference reveal.** The AI reveals a reference solution + design doc.
5. **Gap analysis.** You compare your build against the reference. Identify what you missed, what you over-engineered, and what you got right.

### Reuse the scaffold

Every project starts from the T2 `production-api-template`. You never rebuild the scaffold.

### Production-grade means

Every project must be:

- **Fully documented** (`README.md` with architecture diagram, trade-offs, limitations)
- **Fully tested** (unit + integration with Testcontainers)
- **Containerized** (Docker + Docker Compose)
- **Benchmarked** (`k6` or `autocannon`, results documented)
- **Observable** (metrics, structured logs, tracing)
- **Deployable** (via GitHub Actions + Docker image)

A project is not "done" until it ships.

### Core 6 vs Optional 7

**Core 6** — you build all of these. They cover every major backend capability. When they're done, the curriculum is complete.

**Optional 7** — extensions. Build them for domain practice, interview prep, or portfolio depth. Skip them if you've proved the capability elsewhere.

---

## Core 6 — Required

### 1. URL Shortener

**Brief**: Build a service that accepts a long URL and returns a short code. Redirecting the short code should 302 to the original. Track click counts per code.

**Capabilities checklist**:

- `POST /shorten` with a long URL, returns short code
- `GET /:code` redirects to the original URL (302)
- `GET /:code/analytics` returns click count and last-clicked timestamp
- Base62 key generation with collision handling
- Redis caching for hot codes (LRU)
- Indexed lookups on Postgres
- Rate limiting on the shorten endpoint
- Handles concurrent creation of the same URL (idempotency)

**What it proves**: layered architecture, caching, key generation, rate limiting, redirect semantics.

**Deliverables**: `projects/mastery/01-url-shortener/`

---

### 2. Feature Flag Service

**Brief**: A service that stores feature flags and evaluates them for users. Supports percentage rollouts and user segment targeting. Real-time flag updates via WebSocket.

**Capabilities checklist**:

- CRUD for feature flags (create, read, update, delete)
- Evaluation endpoint: `POST /evaluate` with `{ flag, userId, attributes }` → `{ enabled: boolean }`
- Percentage rollout (deterministic per user)
- Segment targeting (e.g., `plan === 'enterprise'`)
- Real-time updates: flag changes pushed to WebSocket subscribers
- Audit log for flag changes
- Cached evaluation in Redis with invalidation on flag change
- Per-tenant isolation (flags belong to a tenant)

**What it proves**: CRUD, caching + invalidation, real-time push, multi-tenancy, evaluation logic.

**Deliverables**: `projects/mastery/02-feature-flag-service/`

---

### 3. Notification Platform

**Brief**: A service that dispatches notifications across email, SMS, and in-app channels with user preferences, rate limits, and retries.

**Capabilities checklist**:

- `POST /notifications` with `{ userId, channel, template, payload }`
- User preference enforcement (opt-out per channel)
- Multi-channel: email (Resend/Brevo), SMS (mock), in-app (Redis pub/sub)
- Background delivery via BullMQ with retries and DLQ
- Rate limiting per user per channel (e.g., 3 emails/hour)
- Priority queue (transactional OTP vs promotional)
- Template rendering (variables, i18n locale)
- Delivery status tracking (`pending` → `sent` → `delivered` / `failed`)
- Delivery metrics per channel

**What it proves**: background jobs, multi-channel integration, rate limiting, preferences, templates, i18n.

**Deliverables**: `projects/mastery/03-notification-platform/`

---

### 4. Identity Provider

**Brief**: Build an OAuth2 + OIDC identity provider from scratch. Other services integrate with it for authentication and authorization.

**Capabilities checklist**:

- User registration with email verification
- Argon2id password hashing
- OAuth2 Authorization Code Flow with PKCE
- `/authorize`, `/token`, `/userinfo`, `/.well-known/openid-configuration` endpoints
- JWT signing (RS256) with JWKS endpoint (`/.well-known/jwks.json`)
- Client registration (for apps integrating with the IdP)
- Scopes and consent
- Refresh tokens with rotation
- Session management with revocation
- Rate limiting on auth endpoints
- Audit logging

**What it proves**: OAuth2/OIDC from scratch, JWT, RSA signing, JWKS, full auth lifecycle.

**Deliverables**: `projects/mastery/04-identity-provider/`

---

### 5. Payment Gateway Simulator

**Brief**: A payment service that accepts charges, handles retries, supports refunds, and reconciles with a mock external PSP. Money-safe, idempotent, webhook-driven.

**Capabilities checklist**:

- `POST /charges` with amount, currency, idempotency key
- Money stored as integer minor units; ledger uses `NUMERIC`
- Double-entry ledger (debits = credits always)
- Mock external PSP with configurable failure rate
- Webhook handler for PSP callbacks (HMAC verification)
- Idempotency: same key + same body → same response; same key + different body → 422
- Refund endpoint (partial and full)
- Nightly reconciliation job: compare internal ledger vs PSP statement, report discrepancies
- Transaction-safe state machine (`pending` → `authorized` → `captured` / `failed`)
- Audit log for every state change

**What it proves**: money correctness, idempotency, webhook security, transactions, reconciliation, state machines.

**Deliverables**: `projects/mastery/05-payment-gateway-simulator/`

---

### 6. E-commerce Backend

**Brief**: Build the backend for an e-commerce platform. This is the composition project — it reuses every capability you've built.

**Capabilities checklist**:

- User auth (reuse IdP patterns or embed T4 auth)
- Product catalog with search (Postgres + Meilisearch/Elasticsearch sync via outbox)
- Inventory management (stock, reservations, transfers)
- Shopping cart (session-backed, Redis)
- Order placement with transactional stock decrement
- Payment integration (Stripe test mode)
- Order status tracking with WebSocket updates
- Multi-tenant (per-shop isolation via RLS)
- Admin endpoints (product CRUD, order management, reporting)
- Emails: order confirmation, shipping notification
- i18n: currency display, localized errors, timezone-aware order timestamps
- API versioning: `/v1/*` and `/v2/*` with Sunset headers
- Rate limiting per tenant
- Observability: metrics, structured logs, traces
- Load test: verify order throughput under concurrent load

**What it proves**: everything. This is the "final boss" of the curriculum.

**Deliverables**: `projects/mastery/06-ecommerce-backend/`

---

## Optional 7 — Extensions

### 7. CMS Backend

**Brief**: A content management system with versioning, drafts, publishing workflow, and file attachments.

**Capabilities**: content models, version history, draft/publish state machine, file uploads (MinIO), scheduled publishing (BullMQ), multi-language content, search, RBAC (author, editor, admin).

**Why optional**: reinforces document modeling (T3b), file handling (T5), and auth patterns (T4) in a domain where they naturally fit.

---

### 8. Learning Management System Backend

**Brief**: A backend for online courses: courses, lessons, enrollments, progress tracking, quizzes, and certificates.

**Capabilities**: course catalog, enrollment (idempotent), progress tracking (per-lesson completion), quiz submission and grading, certificate generation (PDF), scheduled reminders (cron), multi-tenancy (per-institution), i18n.

**Why optional**: reinforces state machines, scheduled jobs, and file generation.

---

### 9. Chat Backend

**Brief**: A real-time chat backend with WebSockets, message history, presence, and group channels.

**Capabilities**: WebSocket gateway, message ordering (per-channel sequence), message persistence (Postgres or MongoDB), presence tracking (Redis), group fan-out (Redis Pub/Sub), typing indicators, read receipts, message history pagination (keyset), attachment support.

**Why optional**: reinforces real-time patterns (T7), Pub/Sub fan-out, and message ordering.

---

### 10. Video Platform Backend

**Brief**: A backend for uploading, transcoding, and streaming video.

**Capabilities**: presigned multipart uploads (MinIO), job queue for transcoding (BullMQ + FFmpeg mock), HLS segmentation, streaming endpoint with range requests, thumbnail generation, video metadata, playback analytics.

**Why optional**: reinforces large file handling, background jobs, and streaming.

---

### 11. Search Engine (From Scratch)

**Brief**: Build a simple inverted-index search engine as a service.

**Capabilities**: document ingestion, tokenization, inverted index (in-memory + persistent), TF-IDF or BM25 scoring, phrase queries, prefix search, ranking, faceted filters, index persistence and rebuild.

**Why optional**: deepens understanding of what Elasticsearch/Meilisearch do under the hood. Not needed for using them.

---

### 12. Object Storage (From Scratch)

**Brief**: Build an S3-like object storage service.

**Capabilities**: bucket management, PUT/GET/DELETE objects, multipart uploads, presigned URLs, object metadata, listing with pagination, storage on local disk with block layout, basic auth, quota enforcement, versioning.

**Why optional**: reinforces storage internals, HTTP semantics, and streaming.

---

### 13. API Gateway (From Scratch)

**Brief**: Build an API gateway that routes requests to backend services.

**Capabilities**: path-based routing, service discovery (static config), request/response transformation, JWT validation, rate limiting (Redis), circuit breaker per route, access logging, metrics, health checks, graceful shutdown, config hot-reload.

**Why optional**: reinforces gateway patterns and reverse-proxy mechanics. Useful for interviews.

---

## Recommended Order

If you build all 13:

1. **URL Shortener** (warm-up)
2. **Feature Flag Service** (real-time + caching)
3. **Notification Platform** (queues + multi-channel)
4. **Identity Provider** (OAuth2 from scratch)
5. **Payment Gateway Simulator** (money + idempotency)
6. **E-commerce Backend** (composition — the final boss)

Then optional:

7. **CMS Backend** (document modeling)
8. **Learning Management System Backend** (scheduling + state machines)
9. **Chat Backend** (real-time)
10. **Video Platform Backend** (large files + jobs)
11. **Search Engine** (from scratch)
12. **Object Storage** (from scratch)
13. **API Gateway** (from scratch)

---

## Why Not? (Mastery Phase Decisions)

### Why Build the Core 6 Before Optional 7?

- **Core 6 cover every major capability**: caching, queues, auth, payments, composition. If you only built one project and stopped, these 6 would be the answer.
- **Optional 7 are domain extensions**: content, real-time, media, search internals, storage internals, gateway internals. Valuable for portfolios and interview prep, but they repeat capabilities you've already proven.
- **The order matters**: URL Shortener → Feature Flag → Notification → Identity → Payments → E-commerce. Each builds on skills from the previous. E-commerce is the composition capstone; it should be last.

### Why Reuse the T2 Scaffold Instead of Starting Fresh?

- **Starting fresh** — you rewrite routing, validation, logging, error handling, DI, config for every project. That's 20+ hours of boilerplate per project, none of it teaching you anything new.
- **Reusing the scaffold** — you start every project with the architecture already in place. 100% of your time goes into the domain-specific problem.
- **In real jobs**: every team has a service template. You don't rewrite it per project. Reusing the T2 scaffold mirrors how real engineering works.
- **The Mastery Phase tests judgment, not typing speed**. The scaffold frees you to exercise judgment.

### Why Blind Builds (10D) Over Guided Builds?

- **Guided builds** — hints at every step. You end up following a script.
- **Blind builds** — you get a one-paragraph brief and a capabilities checklist. You design and build it yourself. The AI only reveals the reference _after_ you're done.
- **Why blind**: the entire point of Mastery is proving you can build without hand-holding. Guided builds prove nothing.
- **Cost**: higher risk of building something wrong. But the gap analysis is where the learning happens.

### Why Reference Solutions Are Not "The Correct Answer"?

- **Reference** — one plausible design with documented trade-offs. Not the only correct one.
- **Your design** — may be better or worse on different axes. The gap analysis identifies differences; you decide which approach was right for your constraints.
- **Rule**: never treat the reference as authoritative. Treat it as a second opinion. Your design is judged by whether it makes coherent trade-offs, not whether it matches the reference.

### Why Not Stop at Core 6?

- **You can** — Core 6 completes the curriculum. Optional 7 is genuinely optional.
- **Why you might continue**: portfolio depth, domain-specific practice, or interview prep for a specific company (e.g., if interviewing at Figma, build the Collaborative Editor).
- **Why you might stop**: diminishing returns. After Core 6, you've proven every capability. Optional 7 gives marginal practice, not new skills.

### Why Each Project Must Include ADRs, Benchmarks, and README?

- **READMEs** — architectural reasoning, trade-offs, limitations. The artifact interviewers look at.
- **Benchmarks** — `k6`/`autocannon` results. Proves you can measure performance, not just claim it.
- **ADRs** — the "why" behind every significant decision. Senior-engineer signal.
- **Without these**, a project is code without context. With them, it's a portfolio piece.

---

## What "Done" Looks Like

You've completed Backend Mastery when:

- **Core 6** are all built, tested, containerized, benchmarked, documented, and deployed
- Each Core project has a README, benchmarks, and a running Docker Compose setup
- You can start a new project from `production-api-template` and ship a real feature within a day
- You can look at an unfamiliar backend problem and know which tools to reach for

**Optional 7** are exactly that: optional. Stop at Core 6 and the curriculum is complete.

---

## Reference Solutions

For each project, a reference solution exists (revealed after your build). The reference includes:

- Full source code
- Architecture diagram
- Design doc (trade-offs, alternatives considered)
- Benchmarks (throughput, latency)
- A "what I'd do differently at 10x scale" section

The reference is not the only correct answer. It's **a** correct answer — used to run gap analysis, not to grade.
