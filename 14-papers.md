# Seminal Papers

> Classic systems papers mapped to tiers. Read when you reach the tier they reinforce.
> **Most theoretical papers live in Repo 2 (System Design).** Only two papers live here — the ones whose primary relevance is backend engineering, not distributed theory.

---

## Why Only Two Papers

The bulk of classic systems papers (Lamport clocks, Paxos, Raft, Spanner, Dynamo, Bigtable, etc.) reinforce **distributed systems theory** — they belong in Repo 2. Including them here would re-introduce exactly the content we deliberately excluded from Backend Engineering.

Two papers genuinely reinforce **backend engineering topics** you actually build in T1–T7:

1. **Google Dapper** — because you implement tracing in T6.
2. **Google MapReduce** — because you build batch pipelines in T5 and beyond.

Both are optional reads. Neither is required to complete the curriculum.

---

## Google Dapper Paper (Sigelman et al., 2010)

**Ties to**: T6 (Observability).

**Time Budget**: 45 mins.

**Target Depth**: Abstract & Architecture Only.

**What it covers**: Dapper is Google's distributed tracing system. The paper describes how trace context is propagated across services (via request IDs in headers), how spans are structured (parent-child hierarchy), and how sampling keeps overhead low (~0.01% in production).

**Why this matters today**: Dapper is the direct blueprint for OpenTelemetry, Jaeger, Zipkin, and every modern tracing system. The paper explains **why** trace context propagation works the way it does — which matters when you're debugging traces in your own services.

**Read before**: implementing OpenTelemetry in T6.

**Key concepts**:
- Trace context propagation via headers
- Span hierarchy (root span, child spans)
- Sampling strategies (head-based, adaptive)
- Low-overhead tracing via out-of-band sampling
- Trace ID as a cross-service correlation primitive

---

## Google MapReduce Paper (Dean & Ghemawat, 2004)

**Ties to**: T5 (background jobs, batch processing).

**Time Budget**: 45 mins.

**Target Depth**: Abstract & Architecture Only.

**What it covers**: MapReduce is Google's framework for processing massive datasets by splitting work into `map` and `reduce` phases, with a scheduler distributing work across a cluster. The paper describes worker fault tolerance, data locality optimization, and the simplicity of the programming model.

**Why this matters today**: MapReduce is the conceptual ancestor of Hadoop, Spark, and Flink, but more importantly, its **batch processing mindset** is directly applicable to how you design background job pipelines in T5. The idea that "work is split into stages, each stage is idempotent, and failures are retried" is the same principle behind BullMQ pipelines.

**Read before**: designing your T5 background job pipeline.

**Key concepts**:
- Map phase (parallel processing of input chunks)
- Shuffle phase (grouping by key)
- Reduce phase (aggregation)
- Worker fault tolerance via retry
- Data locality (process where the data lives)
- Simple programming model for complex parallelism

---

## Papers That Moved to Repo 2 (System Design)

These papers live in the System Design & Distributed Systems curriculum. Listed here for reference:

- **Lamport's Time, Clocks, and the Ordering of Events** (1978) — logical clocks, event ordering
- **Sagas** (Garcia-Molina & Salem, 1987) — distributed transactions, compensation
- **Paxos Made Simple** (Lamport, 2001) — consensus
- **Chord** (Stoica et al., 2001) — DHT, consistent hashing
- **Google GFS** (Ghemawat et al., 2003) — distributed file system
- **Google Bigtable** (Chang et al., 2006) — wide-column storage, LSM trees
- **Amazon Dynamo** (DeCandia et al., 2007) — eventually consistent KV, quorums
- **Google Dremel** (Melnik et al., 2010) — columnar storage, OLAP
- **Apache ZooKeeper** (Hunt et al., 2010) — distributed coordination
- **Apache Kafka Paper** (Kreps et al., 2011) — distributed commit log
- **Google Spanner** (Corbett et al., 2012) — global ACID, TrueTime
- **Jay Kreps' The Log Manifesto** (2013) — event-driven architecture, CDC
- **Facebook TAO** (Bronson et al., 2013) — multi-region graph caching
- **Raft Consensus Paper** (Ongaro & Ousterhout, 2014) — consensus, leader election
- **Amazon Aurora** (Verbitski et al., 2017) — cloud-native DB architecture

When you reach Repo 2, each paper has a full reading-time budget, depth target, and "why this matters today" annotation.

---

## Optional Reading Strategy

If you want to read more than the two papers here, follow this rule: **read papers that reinforce something you're building.** Not papers about things you've never touched.

Concretely:

- Building a queue? Read the **Kafka paper** (in Repo 2) for context — but understand you're building on BullMQ, not Kafka.
- Building search? Read the **MapReduce** paper here, and possibly **Dremel** (Repo 2) for columnar storage context.
- Building an event-driven system? Read **The Log Manifesto** (Repo 2).
- Debugging a distributed issue? Read **Dapper** here for tracing, **Lamport** (Repo 2) for ordering.

Papers are leverage, not obligation. The curriculum is complete without them.