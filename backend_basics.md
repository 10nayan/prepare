# Basics

## HTTP basics

What it is
HTTP (HyperText Transfer Protocol) is a stateless request-response protocol over TCP. Every interaction is: client sends request → server sends response. No memory of previous requests (hence stateless).

Request anatomy
```
GET /users/42 HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Content-Type: application/json
Components: Method (GET/POST/PUT/DELETE/PATCH), Path + query string, Headers (metadata), Body (for POST/PUT/PATCH).
```

Status code families
```
2xx — success (200 OK, 201 Created, 204 No Content)
3xx — redirection (301 Moved, 304 Not Modified)
4xx — client error (400 Bad Request, 401 Unauthorized, 404 Not Found, 422 Unprocessable)
5xx — server error (500 Internal, 502 Bad Gateway, 503 Unavailable)
HTTP/1.1 vs HTTP/2 vs HTTP/3
HTTP/1.1: one request per TCP connection at a time (head-of-line blocking)
HTTP/2: multiplexed streams on one TCP connection, header compression (HPACK)
HTTP/3: runs over QUIC (UDP-based), eliminates TCP head-of-line blocking entirely
```

## CRUD operations

Mapping to HTTP methods and SQL
CREATE  → POST   /resources        → INSERT INTO
READ    → GET    /resources/:id    → SELECT FROM
UPDATE  → PUT    /resources/:id    → UPDATE ... SET
         PATCH  /resources/:id    → UPDATE (partial fields only)
DELETE  → DELETE /resources/:id   → DELETE FROM
PUT vs PATCH
PUT replaces the entire resource — send all fields or they reset to null. PATCH sends only the changed fields. For large objects, always prefer PATCH in production — it's smaller and safer.

Response conventions
POST → 201 Created + Location header or new object body
PUT/PATCH → 200 OK + updated object, or 204 No Content
DELETE → 204 No Content (no body needed)
GET (list) → 200 + array + pagination metadata

## Idempotency
Definition
An operation is idempotent if calling it multiple times produces the same result as calling it once. Critical for reliability — networks fail and clients retry.

Idempotent
GET, PUT, DELETE, PATCH (usually)

```
DELETE /orders/42 called 3 times → order is deleted (same end state)
Not idempotent
POST (by default)
```
```
POST /orders called 3 times → 3 duplicate orders created
Making POST idempotent
Use an idempotency key — the client generates a unique UUID and sends it as a header:
```

```
POST /payments
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

{ "amount": 5000, "to": "user_42" }
The server stores the key + result. If the same key comes again, return the cached result without re-executing. Stripe, PayPal all use this pattern.

Why it matters
Without idempotency, network retries cause duplicate charges, double-shipped orders, and race conditions. Always design mutation APIs to be retry-safe.

## SQL fundamentals
Core query structure
```
SELECT u.name, COUNT(o.id) AS order_count
FROM   users u
JOIN   orders o ON o.user_id = u.id
WHERE  u.created_at > '2024-01-01'
GROUP  BY u.id, u.name
HAVING COUNT(o.id) > 5
ORDER  BY order_count DESC
LIMIT  20;
Execution order (not the same as write order)
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

This matters: you can't use a SELECT alias in a WHERE clause because WHERE runs before SELECT.

JOIN types
INNER JOIN: rows that match in both tables
LEFT JOIN: all rows from left + matched rows from right (NULLs for no match)
RIGHT JOIN: opposite of LEFT JOIN
FULL OUTER JOIN: all rows from both, NULLs where no match
Aggregates
COUNT(), SUM(), AVG(), MAX(), MIN() — always pair with GROUP BY when using multiple columns. HAVING filters on aggregated values; WHERE filters on raw rows.

## Data modeling basics
Normalization
Organizing tables to eliminate redundancy and maintain data integrity:

1NF: atomic values, no repeating groups (no arrays in a column)
2NF: every non-key column depends on the full primary key (not a partial key)
3NF: no transitive dependencies — non-key columns depend only on the primary key
Denormalization
Intentionally breaking normalization rules for read performance. E.g., storing user_name on the orders table to avoid a JOIN on every order read. Trade-off: faster reads, harder writes (must update in two places).

Relationships
One-to-one: user → user_profile (foreign key on either side)
One-to-many: user → orders (foreign key on the "many" side: orders.user_id)
Many-to-many: users ↔ roles via a user_roles junction table
Choosing data types
Use UUID for distributed IDs. Use BIGINT for sequential IDs at scale. Use TIMESTAMPTZ (with timezone) instead of TIMESTAMP. Use JSONB (PostgreSQL) for semi-structured data you still need to query.

## Indexing
What an index is
A separate data structure (usually a B-Tree) that stores a sorted copy of one or more columns with pointers to the full row. Turns an O(n) full table scan into O(log n) lookup.

When to index
Columns in WHERE clauses that filter large tables
Columns used in JOIN conditions
Columns in ORDER BY and GROUP BY
Foreign key columns (often missed, causes slow deletes/updates on the parent)
Index types
B-Tree: default, works for =, >, <, BETWEEN, LIKE 'prefix%'
Hash: only for equality (=), faster than B-tree for pure lookup
GIN: for JSONB, arrays, full-text search
Composite: index on (col_a, col_b) — column order matters (leftmost-prefix rule)
Partial: CREATE INDEX ON orders(user_id) WHERE status = 'active' — smaller, faster
Cost of indexes
Every index slows down INSERT/UPDATE/DELETE (the index must be updated too). Don't index every column — analyze slow queries first with EXPLAIN ANALYZE.

## Transactions
ACID properties
Atomicity: all operations in a transaction succeed or all are rolled back — no partial state
Consistency: database moves from one valid state to another (constraints enforced)
Isolation: concurrent transactions don't see each other's intermediate state
Durability: committed data survives crashes (written to WAL/disk)
Isolation levels
READ UNCOMMITTED  -- can read dirty (uncommitted) data
READ COMMITTED    -- only see committed data (PostgreSQL default)
REPEATABLE READ   -- same rows return same values within a transaction
SERIALIZABLE      -- full isolation, as if transactions ran one by one
Common problems solved by isolation
Dirty read: reading data that was written but not committed (then rolled back)
Non-repeatable read: same SELECT returns different values within one transaction
Phantom read: new rows appear in repeated range queries
Pessimistic vs optimistic locking
Pessimistic
SELECT ... FOR UPDATE — locks the row immediately. Good for high-contention writes.
Optimistic
Read with a version/timestamp, check on write. Good for low-contention reads. Fails with a conflict error if another writer changed the row first.

## Basic async behavior
Sync vs async
Synchronous: the caller waits and blocks until the operation completes. Simple but wastes CPU time on I/O waits.

Asynchronous: the caller registers a callback / awaits a future, then the runtime handles other work until the result is ready.

Event loop (single-threaded async)
Used in Node.js, Python asyncio. One thread runs a loop: pick a task from the queue → execute until it hits I/O → suspend it → pick the next task. No parallelism, but very high concurrency for I/O-bound work.

```
async def fetch_user(user_id):
    # suspends here, yields to event loop
    user = await db.get(user_id)
    return user
```

Threads vs async vs processes
Threads: true parallelism, but shared memory = locks, race conditions
Async/await: concurrency without parallelism, ideal for I/O-heavy workloads
Processes: full isolation, ideal for CPU-heavy workloads (each gets its own GIL in Python)
When to go async
Database queries, HTTP calls, file I/O — any wait-heavy operation benefits from async. CPU-heavy work (image processing, ML inference) doesn't benefit — use threads or processes instead.

Mid-level
## Caching strategies
Cache-aside (lazy loading)
App checks cache first. On miss, app reads from DB, writes to cache, returns data. Most common pattern.

```
value = cache.get(key)
if not value:
    value = db.query(...)
    cache.set(key, value, ttl=300)
return value
```
Write-through
On every write, update DB and cache together. Cache is always warm. Downside: write latency doubles; cache fills with data that may never be read.

Write-behind (write-back)
Write to cache immediately, persist to DB asynchronously. Very fast writes but risk of data loss if cache crashes before DB sync.

Read-through
Cache sits in front of the DB transparently — the app always talks to the cache, and the cache handles DB fetching on misses. Cache is managed as infrastructure (e.g., DAX in AWS).

Cache invalidation strategies
TTL (time-to-live): auto-expire after N seconds. Simple, but staleness window exists.
Event-driven invalidation: on DB write, delete or update the cache key. Zero staleness, but harder to implement consistently.
Cache versioning: key includes a version hash. Old keys expire naturally, new keys are populated on demand.
Eviction policies
LRU (least recently used) — default in Redis. LFU (least frequently used) — better for skewed access patterns. TTL — evict expired keys first.

## Concurrency handling
Race conditions
Two threads read a value, both modify it, one write clobbers the other. Classic example: two requests both read balance=100, both subtract 50, both write 50 — balance should be 0.

Mutex / locks
Only one thread enters the critical section at a time. Works well in single-process apps. Distributed systems need distributed locks (Redis SETNX, Zookeeper).

Database-level concurrency control
-- Pessimistic: lock the row
```
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 50 WHERE id = 1;
```

-- Optimistic: check version at write
```
UPDATE accounts 
SET balance = balance - 50, version = version + 1
WHERE id = 1 AND version = :expected_version;
```
-- If 0 rows affected → conflict, retry
Deadlocks
Thread A holds lock 1, waits for lock 2. Thread B holds lock 2, waits for lock 1. Neither can proceed. Prevention: always acquire locks in the same global order. Detection: the DB detects and kills one transaction automatically.

Connection pooling
Opening a DB connection is expensive (~100ms). A pool keeps N connections open and reuses them. PgBouncer (PostgreSQL), HikariCP (Java), Django's built-in pool. Tune pool size to match your DB's max_connections.

## Asynchronous processing
Why offload work
Some operations are too slow for a synchronous HTTP response (sending email, generating a PDF, calling 3rd-party APIs, resizing images). Instead: return 202 Accepted immediately, do the work in a background worker.

Task queue pattern
```
Request → API Server → enqueue("send_email", payload) → 202 Accepted
                              ↓
                       Message Queue (Redis/RabbitMQ/SQS)
                              ↓
                       Worker Process → send_email(payload)
                              ↓
                       Update DB: task_status = "done"
```
Celery (Python) example
```
@celery.task(bind=True, max_retries=3)
def generate_report(self, report_id):
    try:
        data = fetch_data(report_id)
        pdf = render_pdf(data)
        upload_to_s3(pdf)
    except Exception as exc:
        raise self.retry(exc=exc, countdown=60)
```
Fire-and-forget vs reliable delivery
Fire-and-forget: push to queue and don't wait. Simple but you lose visibility if the task fails silently. Reliable delivery: store task in DB first (status=pending), then enqueue. If the queue crashes, requeue from DB on restart. Called the "transactional outbox" pattern.

## Replication techniques
What it is
Copying data from one node (primary) to one or more other nodes (replicas) to achieve high availability and read scalability.

Statement-based replication
Ship the SQL statements to replicas. Simple, but breaks for non-deterministic functions like NOW() or RAND() — replicas get different results.

Row-based replication
Ship the actual changed row data. Deterministic, safe. More bandwidth but used by most modern DBs (MySQL binlog row mode, PostgreSQL WAL streaming).

Synchronous vs asynchronous replication
Synchronous
Primary waits for at least one replica to acknowledge before confirming the write. Zero data loss risk. Higher write latency.
Asynchronous
Primary confirms immediately, ships to replicas in background. Low latency writes but replica lag — reads from replica may be stale.
Read replicas
Route SELECT queries to replicas to scale read throughput. Critical rule: always read from primary after a write (read-your-own-write guarantee). Read from replica for analytics, reporting, and stale-tolerant reads.

Replication lag
Replicas fall behind under heavy write load. Monitor lag with pg_stat_replication. Application must tolerate lag or route sensitive reads to primary.

## Consistency guarantees
CAP theorem
In a distributed system, you can only guarantee two of three: Consistency (every read returns the latest write), Availability (every request gets a response), Partition tolerance (system works despite network splits). Since partitions always happen in real networks, you choose CP or AP.

Consistency models spectrum
Strong consistency (linearizability): every read sees the latest write. Slowest. Requires coordination across nodes.
Sequential consistency: all nodes see operations in the same order, but not necessarily real-time.
Eventual consistency: given no new writes, all replicas converge to the same value. Fast but temporarily stale. Used in Cassandra, DynamoDB, Redis (async).
Read-your-own-writes: after writing, that same client always reads its own write. Not all nodes agree but you always see your own changes.
Practical implications
Use strong consistency for financial data, inventory counts, authentication. Use eventual consistency for social feeds, view counts, search indexing, activity logs — where temporary staleness is acceptable.

## Data partitioning
Why partition
A single DB node can't hold or serve unlimited data. Partitioning splits data across nodes (sharding) or within a single DB (table partitioning).

Horizontal partitioning (sharding)
Split rows across nodes. Shard key determines which node holds which rows.

Range-based: users A–M on shard 1, N–Z on shard 2. Simple but creates hotspots if data isn't evenly distributed.
Hash-based: shard_id = hash(user_id) % num_shards. Even distribution but range queries hit all shards.
Directory-based: lookup table maps each key to a shard. Most flexible but lookup table becomes a bottleneck.
Vertical partitioning
Split columns across tables/nodes. Move rarely-accessed or large columns (e.g., user_bio, profile_photo) to a separate store. Reduces row size and speeds up common queries.

Problems with sharding
Cross-shard JOINs don't exist — you must denormalize or do application-level joins. Rebalancing when adding shards is painful. Most teams defer sharding as long as possible (read replicas + vertical scaling first).

## Event-driven patterns
Core idea
Services communicate by emitting events rather than calling each other directly. Decouples producers from consumers — the order service doesn't know about the email service, it just emits order.placed.

Event types
Event notification: small message, "something happened" — consumer fetches details if needed
Event-carried state transfer: full data payload in the event — consumer doesn't need to call back
Event sourcing: the event log IS the source of truth — current state is derived by replaying all events
Pub/sub vs message queue
Pub/Sub (Kafka, SNS)
One event, many consumers. Each consumer gets every message independently. Fan-out.
Message Queue (SQS, RabbitMQ)
One event, one consumer (competing consumers). Work distribution. Exactly-once processing.
Outbox pattern (reliable events)
Write the event to an outbox table in the same DB transaction as the business write. A separate process reads the outbox and publishes to the queue. Guarantees no events are lost even if the queue is down when the write happens.

Challenges
Event ordering, duplicate delivery, schema evolution, debugging distributed flows. Use correlation IDs to trace an event across services.

## Queueing strategies
Why queues
Absorb traffic spikes (producer can outpace consumer temporarily), decouple services, enable async processing, provide durability.

Delivery guarantees
At-most-once: message may be lost, never duplicated. Fast, for non-critical data (metrics, logs).
At-least-once: message delivered at least once, may be duplicated. Consumers must be idempotent. Default for most queues (SQS, Kafka).
Exactly-once: neither lost nor duplicated. Very hard in distributed systems, requires 2-phase commit or Kafka transactions.
Dead letter queue (DLQ)
After N failed processing attempts, the message is moved to a DLQ. Prevents a bad message from blocking the queue forever. Monitor DLQs — they're your signal of processing failures.

Queue depth and back-pressure
If the queue grows without bound, your system has more load than capacity. Monitor queue depth as a key metric. Apply back-pressure: reject or slow down producers when queue depth exceeds a threshold. Scale consumers horizontally to drain the queue.

Priority queues
Route high-priority jobs (user-facing) to a separate queue processed by more workers. Low-priority jobs (bulk exports, nightly reports) to a slower queue. Don't mix SLA-sensitive and batch work in the same queue.

## Retry mechanisms
When to retry
Retry on transient failures: network timeouts, 503 Service Unavailable, 429 Too Many Requests. Never retry on 4xx errors (bad request, wrong credentials) — retrying won't fix them.

Exponential backoff
Double the wait time between each retry to avoid hammering a struggling service:

attempt 1: wait 1s
attempt 2: wait 2s
attempt 3: wait 4s
attempt 4: wait 8s
attempt 5: give up → DLQ or error
Jitter
Add randomness to backoff to prevent thundering herd — all clients retrying at exactly the same time making things worse:

wait = min(cap, base * 2 ** attempt)
wait_with_jitter = random(0, wait)  # full jitter
# or: wait * (0.5 + random(0, 0.5)) # equal jitter
Circuit breaker pattern
Track failure rate of calls to a dependency. When failures exceed a threshold, "open" the circuit — fail fast without even trying. After a timeout, allow a probe request. If it succeeds, close the circuit.

States: CLOSED (normal) → OPEN (failing fast) → HALF-OPEN (testing)
Libraries: resilience4j (Java), pybreaker (Python), opossum (Node.js)
Idempotency + retries
Retries are only safe if the operation is idempotent. Always design mutation operations to be safely retryable before building retry logic on top.

## Load balancing basics
What it does
Distributes incoming requests across multiple server instances to prevent any single server from being overwhelmed. Also provides failover — if one instance dies, the load balancer stops routing to it.

Algorithms
Round-robin: requests go to each server in turn. Simple, works when all servers are identical.
Weighted round-robin: servers with higher capacity get more requests proportionally.
Least connections: route to the server currently handling the fewest active connections. Better for long-lived connections (WebSocket, streaming).
IP hash / sticky sessions: same client always routes to the same server. Needed for stateful apps that store session in memory — but avoidable if sessions are stored in Redis.
Layer 4 vs Layer 7
L4 (TCP/UDP)
Routes based on IP + port. Very fast, minimal processing. Can't inspect HTTP headers or paths. AWS NLB.
L7 (HTTP)
Routes based on URL path, headers, cookies. Can do A/B routing, canary deploys, auth offloading. AWS ALB, Nginx, HAProxy.
Health checks
Load balancer pings /health on each instance every N seconds. Remove from rotation if it fails K times in a row. Add back when it recovers. Always implement a lightweight health endpoint that checks DB connectivity and returns 200.

Session stickiness trade-off
Sticky sessions make load balancing uneven and make scaling harder. The right solution is stateless servers — store all state (sessions, user data) in Redis/DB, not in-memory. Then any server can handle any request.