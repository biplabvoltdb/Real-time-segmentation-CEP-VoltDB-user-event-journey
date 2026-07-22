# VoltDB Event Journey — Schema to Scale

An animated, interactive end-to-end trace of **real-time segmentation** and **complex event processing (CEP)** running on [VoltDB](https://www.voltactivedata.com/) — from `CREATE TABLE` to a 500-million-user peak, in a single self-contained HTML page.

> **[▶ Watch it live](https://YOUR-USERNAME.github.io/voltdb-event-journey/)** *(GitHub Pages — update this link after enabling Pages)*

## What it demonstrates

Two production-grade use cases for a large consumer commerce & payments platform (500M active users, 2–3B events/day):

| # | Use case | What you'll see |
|---|----------|-----------------|
| 1 | **Real-time segmentation** | Rolling per-user aggregates maintained by materialized views; segment membership re-evaluated on *every* event, atomically, in single-digit milliseconds |
| 2 | **CEP / in-session nudging** | Segment state + cart activity + a 30-minute *absence* of payment → a segment-matched offer pushed while the user is still online |

## The five acts

| Act | Chapters | What happens |
|-----|----------|--------------|
| **I · Foundation** | 01–03 | The 3-node cluster is provisioned; DDL creates tables, TTL rules, views and **stored procedures**; 500M profiles hash-distribute across **12 leader + 12 replica partitions** |
| **II · The event** | 04–07 | One payment queues behind other users in the partition's **serializable isolation queue**, executes as one **atomic** multi-step transaction, flips the user Silver → Gold, and commits with a synchronous replica **hash-compare** |
| **III · The nudge** | 08–12 | A cart is abandoned; **TTL + MIGRATE** converts 30 minutes of silence into an event; the procedure re-checks membership *live*, selects the offer for that segment, signs an **idempotency ledger**, and the nudge lands in single-digit ms |
| **IV · Disaster** | 13–14 | A node dies; replicas promote in under a second; the node **rejoins live from its peers**; a redelivered event bounces off the idempotency ledger — exactly-once *effect* under at-least-once delivery |
| **V · Scale** | 15–16 | Peak hour: every partition running its own serial queue in parallel — the same guarantees at tens of thousands of events per second |

Each **ACID guarantee** (plus idempotency) is spotlighted in the UI at the exact chapter where it does its work.

## Running it

No build, no dependencies, no server required — everything is inline in one file.

- **Locally:** download `index.html` and double-click it (any modern browser).
- **GitHub Pages:** Settings → Pages → deploy from the `main` branch root. The page will be served at `https://<your-username>.github.io/voltdb-event-journey/`.

## Controls

| Control | Action |
|---------|--------|
| ▶ / ⏸ | Auto-play or pause the 17-chapter journey |
| ◀ / ▶▶ | Step one chapter back / forward |
| Chapter dots | Jump to any chapter (amber = the nudge act, green = foundation/scale) |
| ← / → arrow keys | Previous / next chapter |
| ⟲ | Restart from the prologue |

Honors `prefers-reduced-motion` — animations collapse to instant state changes.

## VoltDB concepts illustrated

- **Shared-nothing partitioning** — `hash(USER_ID)` routes to one of 12 leader partitions; all of a user's rows co-locate
- **K-safety (K=1)** — every leader has a synchronous replica on a different node; results hash-compared before ack
- **Serializable isolation** — one serial execution queue per partition; no locks, no latches, no deadlocks
- **Stored procedures** — Java + SQL compiled into the engine, executing next to the data (`IngestAndSegment`, `IngestCartWatch`, `NudgeCheck`)
- **Materialized views** — per-day aggregate buckets maintained inside the ingesting transaction
- **`USING TTL … MIGRATE TO TARGET`** — a row's expiry becomes a first-class event (the 30-minute cart watch)
- **Live node rejoin** — a returning node streams its partition data from peer nodes while the cluster keeps serving
- **Idempotent design** — a transactional nudge ledger turns at-least-once delivery into exactly-once effect

## Disclaimer

All latency figures, data volumes and business rules in the animation are **illustrative**, chosen to show the *shape* of the system's behavior. A proof of concept replaces them with measured numbers for a specific workload and infrastructure.

## License

MIT — see [LICENSE](LICENSE). VoltDB is a trademark of Volt Active Data, Inc.; this is an independent demonstration page.
