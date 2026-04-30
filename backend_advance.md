## API Design
REST vs GraphQL vs gRPC

REST
Resource-based URLs, HTTP verbs, stateless. Each resource has its own endpoint. Simple, widely understood, works with any HTTP client.

Problems: over-fetching (endpoint returns 50 fields, you need 3) and under-fetching (need data from 3 endpoints so you make 3 round trips — the N+1 problem).

GraphQL
Single endpoint (POST /graphql). Client specifies exactly what fields it needs. Server returns exactly that — no over/under-fetching.
```
query {
  user(id: "42") {
    name
    orders(last: 5) {
      id
      total
      status
    }
  }
}
```
Best for: mobile clients (bandwidth-sensitive), complex data graphs, when many different clients need different shapes of the same data.

Problems: complex queries can cause expensive DB joins, caching is harder (no GET requests), N+1 DB queries need DataLoader batching.

gRPC
Uses Protocol Buffers (binary serialization) over HTTP/2. You define a .proto schema and generate client + server code in any language. 5–10x faster than REST/JSON for the same payload.
```
// users.proto
service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (stream User);
}
```
```
message User {
  string id = 1;
  string name = 2;
  int64 created_at = 3;
}
```
Best for: internal service-to-service communication, high-throughput microservices, streaming use cases.

Problems: not browser-friendly (needs gRPC-web proxy), harder to debug (binary format), requires proto schema management.

Pick REST when
Public API, simple CRUD, unknown clients, caching needed
Pick GraphQL when
Many different frontends, complex nested data, BFF pattern
Pick gRPC when
Internal microservices, performance-critical, polyglot team
Avoid gRPC when
Browser clients without a proxy, public-facing APIs

## Authentication & authorization
Authentication vs authorization
Authentication (AuthN): who are you? Verify identity. Authorization (AuthZ): what are you allowed to do? Verify permissions. Always do AuthN first, then AuthZ.

Session-based auth
On login, server creates a session in DB/Redis and returns a session ID cookie. On each request, the server looks up the session ID. Stateful — server must store session state. Easy to invalidate.

JWT (JSON Web Token)
Server issues a signed token containing claims (user_id, roles, expiry). Client sends it in every request header. Server verifies the signature — no DB lookup needed. Stateless.

Header.Payload.Signature

Payload:
```
{
  "sub": "user_42",
  "roles": ["admin"],
  "exp": 1735689600,   ← expiry timestamp
  "iat": 1735603200    ← issued at
}
```

Authorization: Bearer eyJhbGci...
JWT problem: you can't invalidate a token before it expires (no server state). Solution: short expiry (15min) + refresh tokens (long-lived, stored in DB). On logout, delete the refresh token.

OAuth2
A delegation protocol — lets users grant a third-party app limited access to their account without sharing their password. Four roles: Resource Owner (user), Client (your app), Authorization Server (Google/GitHub), Resource Server (API).

Flow (Authorization Code):
1. App redirects user to /oauth/authorize?client_id=...&scope=email
2. User logs in at provider, grants permission
3. Provider redirects back with ?code=AUTH_CODE
4. App exchanges code for access_token (server-to-server)
5. App uses access_token to call API
API Keys
Simple long-lived token in header (X-API-Key). Good for machine-to-machine auth. Hash and store in DB (never store plaintext). Scope keys to minimum permissions. Support rotation.

RBAC vs ABAC
RBAC (Role-Based): user has roles, roles have permissions. Simple and fast. Good for most apps. ABAC (Attribute-Based): permissions based on attributes of user, resource, and context. Complex but handles fine-grained rules like "user can edit documents they own in their department."

## API versioning

Why version APIs
You'll need to make breaking changes (rename a field, change a response structure, remove an endpoint). Without versioning, you break all existing clients simultaneously.

URL versioning
GET /v1/users/42
GET /v2/users/42
Most common, most visible. Easy to test in a browser. Easy to route at the API gateway level. Downside: proliferates routes and forces clients to update URLs.

Header versioning
GET /users/42
Accept: application/vnd.myapi.v2+json
or custom header:
API-Version: 2024-01-01
Keeps URLs clean. Stripe uses date-based header versioning — each account is pinned to the API version it was created on. Harder to test in a browser.

Query param versioning
GET /users/42?version=2
Simple but messy — versions get dropped from logs and proxies easily.

Versioning strategy
Only version when making breaking changes: removing fields, changing field types, changing semantics. Additive changes (new fields, new endpoints) are not breaking — old clients ignore new fields. Maintain two versions simultaneously, give clients a sunset date (6–12 months), monitor usage before deprecating.

## Rate limiting & throttling
Why
Protect your API from abuse, prevent one client from starving others, manage cost, comply with third-party quotas. Return 429 Too Many Requests with a Retry-After header.

Token bucket algorithm
Each client has a bucket that fills with tokens at a constant rate (e.g., 10 tokens/second, max 100). Each request consumes one token. If the bucket is empty, the request is rejected. Allows short bursts (up to bucket capacity) while enforcing a long-term average rate.
```
bucket.tokens += rate * elapsed_time
bucket.tokens = min(bucket.tokens, capacity)
if bucket.tokens >= 1:
    bucket.tokens -= 1
    allow_request()
else:
    reject_429()
```
Leaky bucket algorithm
Requests enter a queue (the bucket). A worker processes them at a fixed rate — the "leak". Regardless of burst size, output is always smooth. Good for smoothing traffic to a downstream service that can't handle bursts.

Fixed window counter
Count requests per client in a fixed time window (e.g., 1000 req/hour). Simple. Problem: a client can send 1000 requests in the last second of window 1 and 1000 in the first second of window 2 — 2000 requests in 2 seconds.

Sliding window log
Store timestamp of every request in a sorted set. On each request, count how many timestamps fall within the last N seconds. Most accurate, but high memory usage at scale.

Implementation with Redis
-- Sliding window counter (efficient approximation)
```
key = "ratelimit:{user_id}:{window_start}"
count = INCR key
EXPIRE key 60  -- 1 minute window
if count > 100:
    return 429
```
Rate limit levels
Apply at multiple levels: per IP (DDoS protection), per API key (fair use), per endpoint (protect expensive operations), per user (business quotas). Use different limits per tier — free vs paid.

## WebSockets / SSE / long polling
Why HTTP alone isn't enough
HTTP is request-driven — the server can only send data when the client asks. For real-time use cases (chat, live scores, notifications, collaborative editing), the server needs to push data without waiting for a request.

Long polling
Client makes a request. Server holds it open until there's new data (or a timeout). Client immediately re-connects after receiving a response. Simulates push over regular HTTP.

Client: GET /events (holds connection open)
Server: ... waits 30s ... sends response when event occurs
Client: immediately sends GET /events again
Works everywhere, no special infrastructure. High server-side connection overhead at scale.

SSE (Server-Sent Events)
One-way persistent HTTP connection — server streams events to client. Client can't send data back through it. Built into browsers via EventSource API. Auto-reconnects on disconnect.
```
GET /stream  HTTP/1.1
Content-Type: text/event-stream

data: {"type":"order_update","id":42}\n\n
data: {"type":"price_change","item":"BTC"}\n\n
```
Best for: dashboards, notifications, live feeds, progress tracking. Simple — just a long HTTP response with a specific content type.

WebSockets
Full-duplex, persistent TCP connection. Starts as HTTP upgrade handshake, then switches protocols. Both sides can send messages any time.

HTTP upgrade:
GET /ws HTTP/1.1
Upgrade: websocket
Connection: Upgrade

After handshake:
Client → Server: {"action":"send_message","text":"hello"}
Server → Client: {"type":"message","from":"user2","text":"hi"}
Best for: chat, multiplayer games, collaborative editing, trading platforms. More complex — need to handle reconnection, heartbeats, and scaling across multiple server instances (sticky sessions or a shared pub/sub layer like Redis).

Use SSE when
Server→client only. Notifications, live feeds, progress bars.
Use WebSocket when
Bidirectional. Chat, games, collaborative editing.

## DNS basics
What DNS does
Translates human-readable domain names (api.example.com) to IP addresses (104.21.58.32). Like a phone book for the internet. Without DNS, every user would need to remember IP addresses.

Resolution chain
Browser checks local cache
  → OS /etc/hosts file
    → Recursive resolver (your ISP or 8.8.8.8)
      → Root nameserver (knows who handles .com)
        → TLD nameserver (knows who handles example.com)
          → Authoritative nameserver (returns the actual IP)
              ← answer propagates back through the chain
Record types
A — domain → IPv4 address
AAAA — domain → IPv6 address
CNAME — domain → another domain (alias). Can't be used at root domain.
MX — mail server for the domain
TXT — arbitrary text, used for domain verification, SPF, DKIM
NS — which nameservers are authoritative for this domain
TTL (Time To Live)
Each DNS record has a TTL in seconds. Resolvers cache the answer for that duration. Low TTL = faster propagation when you change records. High TTL = less DNS lookup overhead. For production: 300s normally, drop to 60s before a planned migration.

DNS in system design
DNS-based load balancing: return multiple A records for the same domain. Clients pick one (usually the first). Not true load balancing — DNS doesn't know server health. GeoDNS: return different IPs based on the client's geography, routing users to the nearest data center. DNS failover: health-check aware DNS (Route53, Cloudflare) that removes unhealthy IPs from responses.

## Distributed Systems Patterns

CQRS
The problem it solves
Read and write workloads have fundamentally different requirements. Writes need consistency, normalization, transactions. Reads need speed, denormalized flat data, complex joins already resolved. Forcing both through the same model is a compromise that serves neither well.

Core idea
Separate the write model (Commands) from the read model (Queries). They can have different schemas, different stores, different scaling strategies.

Write side (Command):
POST /orders → OrderCommandService
  → validates business rules
  → writes normalized data to PostgreSQL (orders, order_items)
  → emits OrderPlaced event

Read side (Query):
GET /orders/42 → OrderQueryService
  → reads from Redis or a denormalized "order_view" table
  → returns a single flat object with all needed data pre-joined
Why the read model is different
Your write model is normalized (3NF) — good for integrity. Your read model is shaped for the UI. A single order view might need: order fields + user name + product names + shipping address + latest status. On the write side, that's 4 tables joined. On the read side, it's one pre-materialized document or row.

Sync strategies
When the write side commits, it emits an event. A projection service consumes the event and updates the read model. The read model is eventually consistent with the write model — there's a lag of milliseconds to seconds.

When to use CQRS
When read and write loads are asymmetric (e.g., 1000:1 read/write ratio). When read requirements keep forcing denormalization onto your write schema. When you need multiple read representations of the same data (e.g., a dashboard view and a detail view of the same order).

Don't use CQRS for simple CRUD. The operational complexity (two models, eventual consistency, sync bugs) is only worth it at scale.

## Saga pattern
The problem
A business transaction spans multiple services. Example: placing an order requires: (1) reserve inventory, (2) charge payment, (3) schedule delivery. A traditional DB transaction can't span service boundaries. If payment fails after inventory is reserved, you have inconsistent state.

Core idea
Break the distributed transaction into a sequence of local transactions, each publishing an event. If any step fails, execute compensating transactions to undo completed steps.

Choreography saga
No central coordinator. Each service listens for events and reacts. Decoupled, but hard to visualize the overall flow and debug failures.

OrderService emits: OrderCreated
  → InventoryService hears it → reserves stock → emits StockReserved
    → PaymentService hears it → charges card → emits PaymentCharged
      → DeliveryService hears it → schedules → emits DeliveryScheduled

On failure:
PaymentService fails → emits PaymentFailed
  → InventoryService hears it → releases stock → emits StockReleased
    → OrderService hears it → marks order failed
Orchestration saga
A central saga orchestrator (a state machine) tells each service what to do and tracks progress. Easier to reason about, easier to debug, but the orchestrator becomes a central dependency.

SagaOrchestrator:
  step 1: send ReserveStock to InventoryService
    on success → step 2
    on failure → mark failed, done
  step 2: send ChargePayment to PaymentService
    on success → step 3
    on failure → send ReleaseStock (compensate step 1), mark failed
  step 3: send ScheduleDelivery to DeliveryService
    on success → mark complete
Compensating transactions
Not the same as a DB rollback — you're not undoing a DB write, you're executing business logic that reverses the effect. Refund a charge, release reserved stock, cancel a booking. Some operations can't be compensated (e.g., an email was already sent). Design for this upfront.

Saga vs 2-Phase Commit (2PC)
2PC: all services lock resources until all agree to commit. Strong consistency but blocking — one slow/dead service holds everyone. Rarely used in microservices. Saga: eventual consistency, non-blocking, each service commits locally. Correct choice for microservices.

## Distributed locking

Why you need it
Multiple application instances run simultaneously. DB-level locks only work within one DB transaction. For cross-service critical sections (e.g., only one worker should process a given job), you need a lock that all instances can see.

Redis SETNX pattern
-- Acquire lock (SET if Not eXists)
SET lock:resource_id owner_id NX PX 30000
-- NX = only set if key doesn't exist
-- PX 30000 = auto-expire in 30s (safety: prevent deadlock if holder crashes)

-- Release lock (only if you own it)
if GET lock:resource_id == owner_id:
    DEL lock:resource_id
The owner ID (UUID) ensures you don't accidentally release someone else's lock if yours expired and another process acquired it.

Redlock algorithm
For stronger guarantees across Redis instances. Acquire the lock on N/2+1 Redis nodes within a time window. If enough nodes agree, you have the lock. Protects against a single Redis node failure. Controversial — Martin Kleppmann argues clock drift can still cause safety violations.

Lock expiry and fencing tokens
A lock holder can pause (GC, network hiccup), its lock expires, another process acquires the lock, then the first process resumes thinking it still holds the lock. Solution: fencing tokens — a monotonically increasing number issued with the lock. The resource server rejects writes with a lower token than the last seen.

When not to use distributed locks
If you can achieve the same result with DB-level optimistic locking or idempotency keys, do that instead — it's simpler. Distributed locks are for coordinating access to external resources (a file, a 3rd-party API, a job) that don't have their own concurrency control.

## Service discovery
The problem
In microservices, service instances are ephemeral — they start and stop, their IPs change. You can't hardcode IPs. Services need a way to find each other dynamically.

Client-side discovery
The calling service queries a service registry (Consul, Eureka) to get a list of healthy instances, then load-balances across them itself. More control but every service needs discovery logic.

OrderService wants to call InventoryService:
1. Query Consul: GET /v1/health/service/inventory-service
2. Get list of healthy instances: [10.0.0.1:8080, 10.0.0.2:8080]
3. Pick one (round-robin), send request directly
Server-side discovery
The calling service sends requests to a load balancer or API gateway. The LB queries the registry and routes to a healthy instance. The service doesn't need to know about discovery at all. Used by AWS ALB + ECS, Kubernetes.

Kubernetes DNS-based discovery
Each service gets a stable DNS name: inventory-service.default.svc.cluster.local. kube-proxy handles routing to healthy pods. The calling service just uses the service name — discovery is transparent.

// No discovery code needed in Kubernetes:
fetch("http://inventory-service/api/stock/42")
Health registration
Services self-register on startup and deregister on shutdown. If a service dies without deregistering, the registry uses health checks (HTTP ping, TTL heartbeat) to detect and remove unhealthy instances automatically.

## API Gateway pattern
What it is
A single entry point that sits in front of all your microservices. Clients talk to the gateway, never directly to individual services. The gateway handles cross-cutting concerns that would otherwise be duplicated in every service.

What the gateway does
Routing: /orders/* → Order Service, /inventory/* → Inventory Service
Auth: validate JWT/API key once at the gateway — services trust the forwarded identity header
Rate limiting: per-client quotas before requests reach services
SSL termination: decrypt HTTPS once at the gateway — internal traffic can be HTTP
Request/response transformation: adapt legacy service contracts to modern API shapes
Aggregation (BFF): combine responses from multiple services into one response for the client
BFF (Backend For Frontend)
One gateway variant per client type — mobile BFF, web BFF, partner BFF. Each returns exactly the data shape that client needs, aggregating from the same underlying microservices. Avoids over-fetching without requiring GraphQL everywhere.

Trade-offs
The gateway is a single point of failure — it must be highly available and horizontally scaled. It also adds a network hop and becomes a potential bottleneck. Don't put business logic in it. Keep it as pure infrastructure.

Tools
AWS API Gateway, Kong, Nginx, Traefik, Envoy. For Kubernetes: Ingress controllers (Nginx ingress, Traefik ingress).

## Observability & Reliability
Observability (logs + metrics + traces)
The three pillars
You can't debug a distributed system you can't observe. The three pillars give you different lenses into the same system.

Logs
Time-stamped records of events. Use structured logging (JSON) so they're queryable, not just grep-able.
```
// Bad: hard to parse, query, or alert on
logger.info("Order 42 placed by user 100 for $59.99")
```
```
// Good: structured, filterable
logger.info("order_placed", extra={
    "order_id": 42,
    "user_id": 100,
    "amount": 59.99,
    "correlation_id": "abc-123"
})
```
Log levels: DEBUG (local dev only), INFO (normal operations), WARN (recoverable issues), ERROR (action needed). Never log PII or secrets.

Metrics
Numeric measurements aggregated over time. Four golden signals (Google SRE):

Latency: how long requests take (p50, p95, p99 — percentiles, not averages)
Traffic: requests per second
Errors: error rate (5xx / total requests)
Saturation: how full your resources are (CPU %, queue depth, DB connections)
// Prometheus-style metrics
http_requests_total{method="POST", path="/orders", status="200"} 1024
http_request_duration_seconds{quantile="0.99"} 0.432
db_pool_connections_active 18
Distributed traces
A trace follows a single request across multiple services. Each service creates a span (start time, end time, metadata). Spans are linked by a trace ID propagated in headers.
```
Request lands on API Gateway:  trace_id=abc, span_id=1
  → OrderService:              trace_id=abc, span_id=2, parent=1
    → InventoryService:        trace_id=abc, span_id=3, parent=2
    → PaymentService:          trace_id=abc, span_id=4, parent=2
```
Lets you see exactly where time is spent in a distributed call chain. Tools: Jaeger, Zipkin, AWS X-Ray, Datadog APM, OpenTelemetry (vendor-neutral instrumentation standard).

Correlation IDs
Generate a UUID at the entry point (API Gateway or first service) and pass it in every downstream call header (X-Correlation-ID). Log it in every service. Now you can pull all logs for a single user request across every service in seconds.

Alerting on metrics, not logs
Alert on error_rate > 1% or p99_latency > 2s — metric thresholds. Don't alert on individual log lines (too noisy). Use logs for investigation after an alert fires.

## CDN internals
What a CDN does
A CDN (Content Delivery Network) is a globally distributed network of edge servers that cache content close to users. A user in Mumbai hitting your CDN gets served from Mumbai — not your origin server in Virginia. Reduces latency from 200ms → 10ms for cached content.

Cache-Control headers
You control CDN behavior through HTTP headers from your origin:

// Immutable static assets (hash in filename, cache forever)
Cache-Control: public, max-age=31536000, immutable

// API responses (cache briefly at edge)
Cache-Control: public, max-age=60, stale-while-revalidate=300

// Private user data (never cache at CDN)
Cache-Control: private, no-store

// Cache key varies by device type
Vary: User-Agent
Origin pull vs push
Pull (lazy): CDN fetches from origin on first request, caches the response. Zero upfront setup. Edge cache fills gradually as users request content. Used for most web content.

Push: you explicitly upload content to the CDN. Guarantees all edges are warm before the first user request. Used for large file deployments, software downloads.

Cache invalidation
Two strategies: (1) Versioned URLs — embed a content hash in the filename (app.a3b4c5.js). Never invalidate, just deploy a new URL. Old versions expire naturally. (2) CDN purge — explicitly tell the CDN to evict a URL. Fast but costs money per purge and is difficult to purge patterns reliably.

CDN for dynamic content
Modern CDNs (Cloudflare Workers, AWS Lambda@Edge) run code at the edge. You can do authentication, A/B routing, personalization, and API responses at the edge — without hitting your origin at all. Sub-millisecond cold starts.

## Blob / object storage
What it is
Flat key-value store for unstructured binary data: images, videos, PDFs, backups. S3 (AWS), GCS (Google), Azure Blob. Infinitely scalable, cheap, durable (11 nines). Not a filesystem — no directories, no locks, no in-place edits.

Never proxy files through your backend
// Bad: file flows through your server, wastes memory + bandwidth
GET /api/files/photo.jpg → your server → S3 → your server → client

// Good: client downloads directly from S3
GET /api/files/photo.jpg/url → your server returns presigned URL
client → S3 directly (your server not involved)
Presigned URLs
Temporary URLs that grant time-limited access to a private S3 object. Generated by your backend, signed with your AWS credentials. Client uses the URL directly. No need to make objects public.

```
// Backend generates
url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'user/42/photo.jpg'},
    ExpiresIn=3600  # 1 hour
)
return {"url": url}
```

// Client uploads directly to S3 (PUT presigned URL)
// Client downloads directly from S3 (GET presigned URL)
Multipart upload
For files > 100MB. Split into parts (5MB–5GB each), upload in parallel, S3 assembles them. Allows resumable uploads — if one part fails, retry just that part. Maximum object size: 5TB.

Storage classes
S3 Standard (frequent access) → S3 Infrequent Access (monthly) → S3 Glacier (annual archiving) → Glacier Deep Archive (almost never accessed). Lifecycle policies automatically move objects between classes. Huge cost savings for old data.

## Deployment & Scaling
Feature flags
What they are
Runtime configuration that turns code paths on or off without a deployment. Decouple deployment (code is in production) from release (users see the feature). Ship code continuously, release intentionally.

```
if feature_flag("new_checkout_flow", user=request.user):
    return new_checkout_view()
else:
    return old_checkout_view()
```
Flag types
Release flags: short-lived. Turn on a new feature for everyone once it's stable. Delete the flag and old code path soon after.
Experiment flags (A/B): route % of users to variation A vs B. Measure conversion, then roll out winner.
Ops flags: kill switches. Instantly disable an expensive or broken feature under load without a deploy.
Permission flags: enable features for specific users, roles, or beta groups.
Targeting rules
Evaluate flags against context: user.plan == "pro", user.country in ["IN","US"], user.id % 100 < 10 (10% rollout). Gradual rollout: start at 1% → 5% → 25% → 100%, monitoring error rates at each step.

Flag debt
Every flag is tech debt. Flags that are never cleaned up become invisible conditional logic that nobody understands 6 months later. Enforce a policy: every flag has an owner and a TTL. Stale flags get deleted.

Tools
LaunchDarkly, Flagsmith, Unleash (self-hosted), AWS AppConfig, or a simple DB/Redis-backed implementation for small teams.

## Blue-green & canary deployments
The problem
A regular deployment takes down instances and replaces them. Even a "rolling" deploy means some users hit the new version and some hit the old version simultaneously, often causing errors during the transition.

Blue-green deployment
Run two identical production environments. Blue is live. Deploy new version to green. Smoke test green. Flip the load balancer to send all traffic to green in one instant. If something's wrong, flip back to blue in seconds.

Blue (live): v1.0 ←── all traffic
Green (idle): v1.1 (deploy here, test)

After validation:
Blue (idle): v1.0 (keep warm for rollback)
Green (live): v1.1 ←── all traffic switched instantly
Cost: requires 2x infrastructure at all times. Worth it for zero-downtime deployments on critical services. DB migrations are the hard part — must be backward-compatible with both versions.

Canary deployment
Gradually shift traffic to the new version. Start with 1–5% of users, monitor metrics, increase if stable, roll back if error rates spike.

```
Stage 1: 95% → v1.0, 5% → v1.1 (monitor 10 min)
Stage 2: 75% → v1.0, 25% → v1.1 (monitor 30 min)
Stage 3: 0%  → v1.0, 100% → v1.1 (complete)
```
Canary vs Blue-Green: Canary exposes real user traffic to the new version gradually — catches real-world issues that staging doesn't. Blue-green switches all at once but with an instant rollback path.

Kubernetes rolling update
The default k8s deployment strategy. Replace old pods one by one with new pods. Zero downtime but both versions run simultaneously during the transition. Configure maxUnavailable: 0 to ensure no capacity is lost.

## Horizontal vs vertical scaling
Vertical scaling (scale up)
Add more CPU/RAM/disk to the existing machine. Simple — no code changes. Hard limit: there's a maximum machine size. Single point of failure. Usually cheaper up to a point.

Before: 1 × 4 vCPU, 8GB RAM
After:  1 × 16 vCPU, 64GB RAM
Best for: DBs (initially), stateful services that are hard to distribute, when the bottleneck is a single process.

Horizontal scaling (scale out)
Add more instances. Requires stateless design — no instance-local state (use Redis for sessions, S3 for files). Theoretically unlimited. Resilient — losing one instance doesn't cause downtime.

Before: 2 × 2 vCPU instances
After:  10 × 2 vCPU instances (behind a load balancer)
Best for: stateless API servers, worker processes, anything behind a load balancer.

Scaling the DB
DBs are harder to scale horizontally. Common progression: (1) add indexes → (2) add read replicas → (3) add a connection pool (PgBouncer) → (4) vertical scale → (5) shard. Most apps never need step 5. Reach for read replicas and a query cache (Redis) before sharding.

Autoscaling
Automatically add/remove instances based on metrics: CPU %, request rate, queue depth. AWS Auto Scaling Groups, k8s HPA (Horizontal Pod Autoscaler). Key is the right metric — CPU is a lagging indicator. Request rate or queue depth reacts faster.

## Search
Search indexing
Why SQL LIKE is not search
WHERE name LIKE '%honda%' does a full table scan — O(n), no index can help with a leading wildcard. At 10M rows, this takes seconds. And it doesn't handle typos, stemming, synonyms, or relevance ranking.

Inverted index
The core data structure of search engines. Instead of document → words, stores word → list of documents containing it.

Forward index:
  doc1: "Honda City blue"
  doc2: "Honda Civic red"

Inverted index:
  "honda" → [doc1, doc2]
  "city"  → [doc1]
  "civic" → [doc2]
  "blue"  → [doc1]
  "red"   → [doc2]
Query "honda city" → intersect doc lists for "honda" and "city" → [doc1]. O(1) per term lookup.

Text analysis pipeline
Before indexing and before querying, text goes through:

Tokenization: split "Honda City 2024" → ["Honda", "City", "2024"]
Lowercasing: "Honda" → "honda"
Stop word removal: remove "the", "a", "is"
Stemming/lemmatization: "running", "runs", "ran" → "run"
Synonym expansion: "car" also matches "vehicle", "automobile"
Relevance scoring (BM25/TF-IDF)
TF (Term Frequency): how often the term appears in the document — more occurrences = more relevant. IDF (Inverse Document Frequency): how rare the term is across all documents — rare terms are more significant. BM25 is the modern refinement, used by Elasticsearch by default.

Elasticsearch architecture
Data is split into shards (primary + replica). Each shard is an independent Lucene index. Writes go to the primary shard then replicate. Reads can hit any shard copy. Near-real-time: new documents are searchable within ~1 second (configurable refresh interval).

Sync from DB to search index
Options: (1) Dual write in app code — write to DB and ES together (risk: partial failure). (2) CDC (Change Data Capture) — read the DB's WAL/binlog and sync changes to ES asynchronously. Most reliable — search index is eventually consistent with DB. Tools: Debezium, Logstash JDBC plugin.