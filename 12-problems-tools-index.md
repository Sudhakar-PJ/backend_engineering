# Problems → Tools Index

> A grep-able lookup table. When you think "which tool solves this?", come here first.

---

## Language / Runtime

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "I need static types in JavaScript" | TypeScript | T1 |
| "My build is slow / I need TS in prod" | `tsx` (dev), `tsup`/`esbuild` (build), `tsc --noEmit` (check) | T1 |
| "CPU work blocks the event loop" | `worker_threads`, `cluster` | T1, T5 |
| "I can't find where my app is slow / leaking" | `--inspect`, `clinic.js`, heap snapshots | T1, T6 |
| "I need structured logs" | Pino | T1, T2, T6 |
| "I need to cancel in-flight async work" | `AbortController` / `AbortSignal` | T1, T5 |

## Service / HTTP

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "I need an HTTP framework" | Express (primary), Fastify (comparison) | T2 |
| "Repetitive try/catch in controllers" | `asyncHandler` wrapper | T2 |
| "Inconsistent error shapes" | RFC 7807 + typed `AppError` hierarchy | T2 |
| "Request-scoped context without parameter drilling" | `AsyncLocalStorage` | T2 |
| "Config is scattered / unvalidated" | Zod-parsed config module | T2 |
| "Docs drift from code" | OpenAPI + Scalar/Redoc/Bruno | T2, T7 |
| "v1 and v2 need to coexist" | URI path versioning + `openapi-diff` + Sunset/Deprecation headers | T7 |

## Relational Data (Postgres)

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "I need a relational DB" | PostgreSQL | T3a |
| "Raw SQL isn't type-safe" | Kysely | T3a |
| "Schema changes require downtime" | Expand-contract migrations (Kysely migrations) | T3a |
| "Connection pool exhausts under load" | `pg` Pool tuning + PgBouncer | T3a |
| "Slow queries, no idea why" | `EXPLAIN ANALYZE`, `pg_stat_statements`, `auto_explain` | T3a, T6 |
| "Two users update same row, one wins silently" | Optimistic locking (version col), Pessimistic (`SELECT ... FOR UPDATE`), `SKIP LOCKED` | T3a |
| "Money math is wrong" | Integer minor units + `NUMERIC` + banker's rounding | T3a |
| "Text sorts wrong, emails collide" | Unicode normalization + ICU collation | T3a, T7 |

## Document Data (MongoDB)

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "Schema-flexible / nested data" | MongoDB | T3b |
| "Slow Mongo queries" | `.explain("executionStats")` | T3b |
| "Joins across collections" | Aggregation pipeline (`$lookup`, `$facet`, `$group`) | T3b |
| "Transactions across documents" | Client sessions + `startTransaction()` | T3b |

## Key-Value / Cache (Redis)

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "Sub-ms ephemeral state" | Redis | T3c |
| "Atomic counter / CAS across instances" | `INCR`, `SET NX`, Lua scripts, `MULTI/EXEC` | T3c |
| "Rate limiter across instances" | Sliding window / token bucket / GCRA in Redis | T4 |
| "Distributed lock (one worker runs cron)" | `SET NX EX`, Redlock | T3c, T5 |
| "Job queues with retries & scheduling" | BullMQ | T5 |
| "Real-time pub/sub" | Redis Pub/Sub, Redis Streams | T7 |
| "Redis runs out of memory" | Eviction policies, `maxmemory` | T3c |
| "Slow Redis due to round-trips" | Pipelining, Lua scripts | T3c |

## Security / Auth

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "Passwords stored weakly" | Argon2id (primary), bcrypt (awareness) | T4 |
| "Stateless auth across services" | JWT | T4 |
| "JWT can't be revoked" | Short-lived access + refresh rotation + Redis blacklist | T4 |
| "Login with Google/GitHub" | OAuth2 (PKCE) + OIDC + `openid-client` | T4 |
| "Password alone isn't enough" | TOTP (`otplib`), WebAuthn (awareness) | T4 |
| "Malicious payloads" | Zod validation, output encoding, CSP, Helmet | T4 |
| "CORS blocked / wrongly allowed" | Strict origin whitelist + preflight cache | T4 |
| "Secrets in git/logs" | `.env.example`, `gitleaks`/`trufflehog` CI scans, Vault (awareness) | T4, T6 |
| "Forged webhooks" | HMAC-SHA256 + timing-safe compare | T4, T5 |

## Async Work / Reliability

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "Slow work blocks the request" | Background jobs (BullMQ), event emitters, `setImmediate` | T5 |
| "External API fails, request fails" | Retry + jitter, circuit breaker, timeout, fallback | T5 |
| "Client retries = double charge" | Idempotency keys (Redis/DB + response cache) | T4, T5 |
| "Deploy kills in-flight requests" | Graceful shutdown + K8s `preStop` | T5, T6 |
| "Cache stampede / avalanche" | TTL jitter, single-flight mutex, negative caching | T5 |
| "2GB uploads OOM the server" | Multipart upload, presigned URLs (client → MinIO/S3) | T5 |
| "Store files without S3 bills" | MinIO | T5 |
| "Send emails without SMTP server" | Resend / Brevo / SendGrid / SES | T4, T5 |
| "Durable messaging beyond Redis" | RabbitMQ (usage), Kafka (usage), NATS (awareness) | T5 |

## Production Ops

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "Flaky tests because DB is mocked" | Testcontainers | T6 |
| "Third-party APIs flaky in tests" | MSW, Nock | T6 |
| "I need realistic test data" | `@faker-js/faker` + factory builders | T6 |
| "Works on my machine" | Docker + Docker Compose + multi-stage builds | T6 |
| "Manual deploys break things" | GitHub Actions CI/CD | T6 |
| "Deploys cause downtime" | Rolling / blue-green / canary | T6 |
| "Can't disable broken feature" | Feature flags (Unleash or Redis-backed) | T6 |
| "Need to run in K8s" | Pods, deployments, services, ingress, HPA, probes, `preStop` | T6 |
| "No visibility in prod" | Prometheus, Grafana, Loki, OpenTelemetry, Sentry | T6 |
| "Something is slow, no idea where" | OTel spans, Jaeger/Tempo, flamegraphs | T6 |
| "Need alerts" | Prometheus Alertmanager, Grafana alerts | T6 |
| "3am crash, no idea what to do" | Runbooks, postmortems, on-call rotation | T6 |

## Enterprise Surfaces

| Problem | Tool / Pattern | Tier |
|---|---|---|
| "REST too chatty internal calls" | gRPC | T7 |
| "Clients need flexible queries" | GraphQL | T7 |
| "GraphQL N+1" | DataLoader | T7 |
| "Real-time updates" | WebSockets, SSE | T7 |
| "Users expect fuzzy search + facets" | Elasticsearch, Meilisearch, Postgres FTS | T7 |
| "Multi-tenant data isolation" | Pool (RLS), Silo (DB per tenant), Schema-per-tenant | T7 |
| "App breaks in Japanese/Arabic" | `Accept-Language`, CLDR, RTL, ICU collation | T7 |
| "Money conversion loses cents" | Integer minor units + banker's rounding + ISO 4217 | T3a, T7 |