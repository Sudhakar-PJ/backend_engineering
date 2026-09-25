# Case Studies — Enterprise & MAANG

> Real-world engineering stories that reinforce what you're building. Organized by tier.
> **Reading pattern**: each case study is a short read with a "why this matters" note. Read when you reach the tier it reinforces.
> **Cross-referenced in Repo 2**: distributed systems case studies live in the System Design curriculum.

---

## T1 — Language & Runtime

### WhatsApp's famously small server footprint

**Why this matters**: WhatsApp scaled to 900M+ users with a small engineering team by being ruthlessly efficient — Erlang for the backend, custom optimizations, and refusing to add infrastructure "just in case." For a Node.js engineer, the lesson is that runtime mastery and operational discipline beat framework shopping every time.

**Ties to**: T1 (runtime fluency), T5 (efficient async), T6 (production restraint).

---

## T2 — Service Construction

*(No case studies assigned here. T2 builds the scaffold — later case studies reference patterns introduced here.)*

---

## T3a — Relational / Postgres

### Instagram's Postgres sharding strategy

**Why this matters**: Instagram scaled to hundreds of millions of users while staying on Postgres for years. Their approach: keep the schema simple, shard by user ID via a logical-to-physical mapping (not consistent hashing), and use every Postgres feature (JSONB, partial indexes, foreign data wrappers) before adding new infrastructure. The lesson: Postgres goes further than most engineers think.

**Ties to**: T3a (schema design, indexes, partitioning awareness).

### GitHub's zero-downtime MySQL → Vitess sharding migration

**Why this matters**: GitHub migrated from a monolithic MySQL to Vitess (a sharded MySQL) without downtime. The migration used the expand-contract pattern at database scale: dual-write, backfill, verify, cut over. Read this before your first large migration.

**Ties to**: T3a (migrations, expand-contract), T6 (deploy sequencing).

### Discord's migration from Cassandra to ScyllaDB

**Why this matters**: Discord migrated from Cassandra to ScyllaDB (a Cassandra-compatible DB written in C++) to solve tail latency. The postmortem covers the decision criteria, the migration path, and the trade-offs (operational complexity vs. performance). A good read on when NoSQL helps — and when it doesn't.

**Ties to**: T3a (NoSQL awareness), T3b (document vs wide-column).

---

## T3b — Document / MongoDB

### Meta's TAO (social graph) and Haystack (photo storage)

**Why this matters**: TAO is Meta's distributed graph store for the social graph (edges between users, posts, pages). Haystack is their photo storage layer, optimized for the fact that photos are written once and read many times. Both demonstrate the "model around access patterns" principle behind good document/KV design.

**Ties to**: T3b (document modeling), T3c (caching), T5 (object storage).

---

## T3c — Key-Value / Redis

### Meta's TAO (graph caching, read-through)

**Why this matters**: TAO caches the social graph in a distributed read-through cache, with write-through invalidation. Demonstrates cache-aside at massive scale, and why cache invalidation is genuinely hard.

**Ties to**: T3c (cache-aside, invalidation).

---

## T4 — Identity & Security

### Stripe's ledger and idempotency-key architecture

**Why this matters**: Stripe's payment API is built around idempotency keys — the client sends a key with every request, Stripe caches the response, and repeated requests return the same response. This is the canonical design for safe retries in payment systems. Read before you build any charge endpoint.

**Ties to**: T4 (idempotency), T5 (payments), Mastery Project 5.

---

## T5 — Async & Reliability

### Segment's $1M Kafka incident postmortem

**Why this matters**: Segment's Kafka cluster failed in a way that cost them ~$1M in a single day. The postmortem covers dependency failures, retry storms, insufficient circuit breakers, and the operational lessons learned. **Required reading before you build any job queue.**

**Ties to**: T5 (queues, circuit breakers, retry budgets), T6 (incidents).

### Slack's job queue and channel fan-out architecture

**Why this matters**: Slack processes millions of jobs and fans out messages to millions of channel subscribers. The engineering blog covers the queue design (priorities, retries, DLQ), and the fan-out strategy (message routing, presence). Lessons on queue depth monitoring and backpressure.

**Ties to**: T5 (queues), T7 (WebSocket fan-out).

### Reddit's queue architecture & early scaling postmortems

**Why this matters**: Reddit's famous early stability issues were queue-related — how they fixed the "everything is a queue problem." A cautionary tale about over-using queues and under-designing their throughput.

**Ties to**: T5 (queues).

---

## T6 — Production Operations

### Cloudflare's postmortem culture & global outage RCAs

**Why this matters**: Cloudflare publishes detailed postmortems on every major incident, including their own mistakes. Study them as templates for what a good postmortem looks like: timeline, root cause, action items, follow-through.

**Ties to**: T6 (incidents, postmortems).

### Netflix's chaos engineering culture

**Why this matters**: Netflix pioneered Chaos Monkey (randomly kills production instances) and the Simian Army. The lesson is not "kill your production" — it's that resilience is verified, not assumed. Every circuit breaker, retry, and fallback should be tested in production-like conditions.

**Ties to**: T6 (chaos awareness, resilience).

*Cross-referenced from Repo 2 for deeper multi-region context.*

---

## T7 — Enterprise Surfaces

### Stripe's date-based API versioning and per-request transformation layer

**Why this matters**: Stripe versions their API by date (`Stripe-Version: 2024-01-15`), with per-request transformation layers. Each API version is a transformation pipeline from the internal model to the response shape of that version. This is the cleanest approach to long-lived public APIs.

**Ties to**: T7 (API versioning, transformation layers).

### Airbnb's service mesh & migration to SOA

**Why this matters**: Airbnb moved from a Ruby monolith to a service mesh architecture with an API gateway. The blog series covers the migration strategy, service discovery, and the operational practices that made distributed services work at their scale.

**Ties to**: T7 (gateway, service composition).

### Figma/Google Docs: CRDT vs OT trade-offs in practice

**Why this matters**: Both Figma and Google Docs are collaborative editors, but they use different conflict resolution algorithms (Figma uses CRDTs, Docs uses OT). The engineering blogs cover the trade-offs: complexity, latency, memory footprint. Read before building any collaborative feature.

**Ties to**: T7 (WebSockets, real-time).

### Shopify's Pods architecture (scaling the monolith via cells)

**Why this matters**: Shopify scaled their Rails monolith by partitioning it into "Pods" — isolated cells that each handle a subset of shops. This is the cell-based architecture pattern: instead of microservices, use a monolith per cell. A masterclass in scaling without rewriting.

**Ties to**: T7 (gateway), Mastery Project 6.

*Cross-referenced from Repo 2 for cell architecture depth.*

### Amazon Prime Video's microservices → monolith reversal

**Why this matters**: Prime Video famously moved from microservices back to a monolith for their audio/video monitoring service — and cut costs by 90%. Not every problem needs microservices. Read this before you split your first service.

**Ties to**: T7 (service composition), Mastery Phase (architectural restraint).

*Cross-referenced from Repo 2 for microservices-vs-monolith debate.*

### Uber's H3 geospatial indexing and dispatch system evolution

**Why this matters**: Uber uses H3 (hexagonal spatial indexing) for their dispatch system. The blog series covers why hexagonal grids beat geohash in some cases, and how location indexing scales to millions of drivers. Relevant for any spatial backend.

**Ties to**: T7 (spatial awareness), T3a (indexing).

*Cross-referenced from Repo 2 for geospatial depth.*

### Google GFS & Bigtable Architecture in Practice

**Why this matters**: These papers describe how Google built distributed file storage and a wide-column database. The practical lessons: chunk-based storage, append-only logs, single-master trade-offs. Relevant for anyone designing storage at scale.

**Ties to**: T3b (document modeling awareness), T5 (object storage), Mastery Project 12.

*Cross-referenced from Repo 2 for storage internals.*

### Amazon's original Dynamo paper vs. DynamoDB productization

**Why this matters**: Dynamo (the paper) is a masterclass in eventual consistency, quorum reads/writes, and sloppy quorums. DynamoDB (the product) is the pragmatic engineering version of it, with operational trade-offs. Reading both shows how theory meets production.

**Ties to**: T3c (eventually consistent KV), Mastery Project 12.

*Cross-referenced from Repo 2 for distributed KV depth.*

---

## Case Studies Cross-Referenced to Repo 2

These live in the System Design & Distributed Systems curriculum. They are listed here for completeness:

- Shopify's Pods (cell architecture depth)
- Amazon Prime Video (microservices-vs-monolith depth)
- Netflix chaos (multi-region + failure domain depth)
- Uber H3 (geospatial algorithm depth)
- Google GFS & Bigtable (storage internals)
- Amazon Dynamo (quorum math, consistency models)

When you reach Repo 2, these case studies will be re-engaged with more technical depth.