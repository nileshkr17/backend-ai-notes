# System Design Master Interview Template
### One Template — HLD + LLD — Every Question

---

## TL;DR
> Use this template for every system design question. HLD = what components exist and how they talk. LLD = how each component is built internally. Most FAANG interviews want HLD. Some (Uber, Google SWE) go deep into LLD. Know which one the interviewer expects — ask in the first 2 minutes.

---

## How to Identify HLD vs LLD

| Signal | Type |
|---|---|
| "Design Twitter / YouTube / Uber" | HLD |
| "Design the database schema for X" | LLD leaning |
| "Design a rate limiter / cache / queue" | Both (component = LLD, at scale = HLD) |
| "Design the class structure for a parking lot" | LLD |
| "Design a URL shortener" | HLD (with some LLD for hash function) |
| "Walk me through the architecture of your current system" | HLD |

---

# PART A — HLD Template (High-Level Design)

---

## Step 1: Clarify Requirements (3–5 min)

**NEVER skip this. Interviewers give vague questions on purpose.**

### Functional Requirements — What the system DOES
Ask and write down:
```
□ What are the core features? (top 3 only — don't boil the ocean)
□ Who are the users? (consumers, businesses, internal services?)
□ Read-heavy or write-heavy?
□ Is real-time required? (chat, notifications, live updates)
□ Any special flows? (payment, auth, file upload)
```

### Non-Functional Requirements — How well it performs
```
□ Scale: DAU (Daily Active Users)? QPS (queries per second)?
□ Latency: p99 < X ms? Real-time or eventual consistency OK?
□ Availability: 99.9% (8.7h downtime/year) or 99.99% (52min/year)?
□ Durability: Can we lose data? (Never for payments, OK for analytics)
□ Consistency: Strong or eventual? (Bank = strong, social feed = eventual)
□ Data size: How much storage? How long retained?
```

### Out of Scope — Say it explicitly
```
□ "I'll skip auth for now unless you want me to cover it"
□ "Analytics and reporting are out of scope"
□ "I'll assume single region for now"
```

---

## Step 2: Back-of-Envelope Estimation (3–5 min)

**Do this BEFORE designing. Numbers drive every architectural decision.**

### Template Calculations

```
Given: 100M DAU

QPS (reads):
  100M users × 10 reads/day = 1B reads/day
  1B / 86,400 seconds       = ~12,000 QPS reads
  Peak = 3x average         = ~36,000 QPS

QPS (writes):
  100M users × 1 write/day  = 100M writes/day
  100M / 86,400             = ~1,200 QPS writes

Storage (per year):
  1,200 writes/sec × 500 bytes/write = 600 KB/sec
  600 KB × 86,400 × 365             = ~19 TB/year

Bandwidth:
  12,000 reads/sec × 500 bytes = 6 MB/sec read bandwidth
  Peak: 18 MB/sec

Cache size:
  20% of data = 80% of reads (Pareto principle)
  19 TB × 0.20 = ~4 TB hot data to cache
```

### Key Numbers to Memorize

| Thing | Value |
|---|---|
| Seconds in a day | 86,400 |
| Seconds in a year | ~31.5M |
| 1 char | 1 byte |
| 1 int | 4 bytes |
| 1 UUID | 16 bytes (36 chars as string) |
| 1 timestamp | 8 bytes |
| Average tweet/post | ~300 bytes |
| Average image | ~300 KB |
| Average video (1 min, 720p) | ~100 MB |
| L1 cache read | 0.5 ns |
| RAM read | ~100 ns |
| SSD read | ~100 µs |
| HDD seek | ~10 ms |
| Same DC network roundtrip | ~0.5 ms |
| Cross-region roundtrip | ~150 ms |

---

## Step 3: Define the API (3–5 min)

**APIs are the contract. Define them before the architecture.**

```
# REST API pattern:
POST   /v1/resource          → create
GET    /v1/resource/{id}     → read
PUT    /v1/resource/{id}     → full update
PATCH  /v1/resource/{id}     → partial update
DELETE /v1/resource/{id}     → delete
GET    /v1/resource?filter=X → list with filters

# Example — URL Shortener:
POST /v1/urls
  Request:  { "longUrl": "https://...", "expiresAt": "2027-01-01" }
  Response: { "shortUrl": "https://tiny.ly/abc123", "shortCode": "abc123" }

GET /v1/urls/{shortCode}
  Response: 302 Redirect → longUrl

DELETE /v1/urls/{shortCode}
  Response: 204 No Content

# Pagination — always cursor-based (NOT offset):
GET /v1/feed?userId=123&cursor=<token>&limit=20
  Response: { "items": [...], "nextCursor": "<token>", "hasMore": true }
  # Offset pagination breaks at scale — cursor is O(1) regardless of position
```

---

## Step 4: High-Level Architecture (10–15 min)

### Standard Skeleton — Draw This First

```
                    ┌──────────┐
    Mobile/Web ───► │  DNS /   │
                    │  CDN     │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │  Load    │
                    │ Balancer │
                    └────┬─────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
    ┌──────────┐   ┌──────────┐  ┌──────────┐
    │  API     │   │  API     │  │  API     │  (stateless — horizontal scale)
    │ Server 1 │   │ Server 2 │  │ Server N │
    └─────┬────┘   └─────┬────┘  └─────┬────┘
          └──────────────┼──────────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
    ┌──────────┐   ┌──────────┐  ┌──────────┐
    │  Cache   │   │ Primary  │  │ Message  │
    │ (Redis)  │   │   DB     │  │  Queue   │
    └──────────┘   └─────┬────┘  └──────────┘
                         │
                   ┌─────▼────┐
                   │ Replica  │
                   │   DB     │
                   └──────────┘
```

### Component Decision Checklist

**CDN:**
```
□ Use for: static assets (JS/CSS/images), video streaming, geo-distributed users
□ Pull CDN: lazy — loads on first request, cached after
□ Push CDN: pre-populate — good for known large content (videos)
□ Cache-Control headers set TTL
```

**Load Balancer:**
```
□ L4 (TCP): fast, good for non-HTTP (gaming, raw TCP)
□ L7 (HTTP): route by URL/header/cookie — most common
□ Algorithms: Round Robin, Least Connections, IP Hash (sticky), Weighted
□ Health checks remove unhealthy instances automatically
```

**API Gateway (vs plain LB):**
```
□ Use when you need: auth, rate limiting, SSL termination, request routing to microservices
□ API GW = LB + Auth + Rate Limit + Routing in one
□ Examples: AWS API Gateway, Kong, NGINX, Envoy
```

**Database Selection:**
```
□ Relational (PostgreSQL, MySQL):
    - ACID transactions, complex joins, strong schema
    - Financial data, user accounts, orders

□ Document (MongoDB):
    - Flexible schema, nested objects, no complex joins
    - Product catalogs, user profiles, CMS content

□ Wide-column (Cassandra, HBase):
    - Massive write throughput, known access patterns
    - Time-series, events, IoT, activity logs

□ Key-Value (Redis, DynamoDB):
    - Simple key lookups, O(1) access
    - Cache, sessions, leaderboards, rate limiting

□ Search (Elasticsearch):
    - Full-text search, fuzzy matching, autocomplete
    - Log aggregation, product search

□ Graph (Neo4j):
    - Relationships are first-class, multi-hop traversals
    - Social networks, fraud detection, recommendations

□ Time-series (InfluxDB, TimescaleDB):
    - Metrics, monitoring, stock prices
    - Optimized for time-range queries + aggregations
```

**Cache:**
```
□ Cache-aside (lazy): app checks cache → miss → load from DB → store in cache
□ Write-through: write to cache AND DB simultaneously (consistent, slower writes)
□ Write-back: write to cache only, async flush to DB (fast writes, risk of data loss)
□ Eviction: LRU (most common), LFU, TTL-based
□ Redis: in-memory, persistent, supports sorted sets, lists, pub/sub
```

**Message Queue:**
```
□ Use for: async processing, decoupling producers/consumers, spike buffering
□ Kafka: high throughput, persistent, replayable, ordered within partition
    → event streaming, audit log, analytics pipeline
□ RabbitMQ: routing, fanout, priority queues, simpler setup
    → task queues, RPC, complex routing
□ SQS: managed AWS, at-least-once delivery
□ Key concepts: partition, consumer group, offset (replay), retention period
```

---

## Step 5: Deep Dive on 2–3 Components (10–15 min)

### Database Scaling

```
Replication:
  Primary → Replica(s): writes to primary, reads from replicas
  Replication lag: stale reads possible on replicas (eventual consistency)
  Fix stale reads: route user's reads to primary for X seconds after their write

Sharding (horizontal partitioning):
  Shard key choice is CRITICAL — bad key = hot shard = bottleneck

  Range sharding:  userId 0–1M → shard1, 1M–2M → shard2
    ✓ Range queries easy
    ✗ Hot shard if data clusters (new users all on last shard)

  Hash sharding:  hash(userId) % numShards
    ✓ Even distribution
    ✗ Range queries hard, resharding requires data movement

  Consistent hashing: virtual ring — add/remove nodes with minimal data movement
    ✓ Only K/N keys remapped when node added (K=keys, N=nodes)
    ✓ Virtual nodes per physical node → better even distribution

Indexes:
  B-tree: default, good for range + equality queries
  Composite: column ORDER matters — leftmost prefix rule
  Covering index: includes all query columns — no table lookup needed
  Partial index: WHERE clause on index — smaller, faster
```

### Caching Deep Dive

```
Cache invalidation:
  TTL: simple, stale data possible until expiry
  Event-based: publish invalidation event on every write (fan-out invalidation)
  Version key: user:123:v5 — old version key expires naturally

Problems + fixes:
  Cache stampede: many requests hit DB when popular key expires simultaneously
    Fix: probabilistic early expiration, mutex on miss, background refresh

  Hot key: one key gets disproportionate traffic (celebrity user, trending post)
    Fix: local in-process cache + Redis, or shard the key (user:123:shard:0..N)

  Cache penetration: requests for keys that don't exist in DB (attack or bug)
    Fix: cache null results with short TTL, Bloom filter at entry point

  Cache avalanche: many keys expire at same time → DB overwhelmed
    Fix: jitter on TTL (expiry ± random offset), pre-warming
```

### Real-Time / Push Notifications

```
Server → Client options:
  Short polling:  client polls every N seconds — simple, wasteful, high latency
  Long polling:   client holds connection open — better latency, more complex
  WebSocket:      full-duplex TCP — best for chat, live data, gaming
  SSE:            one-way server→client, HTTP-based, auto-reconnect — good for feeds

WebSocket at scale:
  Connection is stateful — user pinned to specific server
  Problem: user A on server1, user B on server2 — how does A's message reach B?
  Solution: Redis pub/sub — server2 publishes to channel, server1 subscribes → delivers to A

Notification fan-out:
  Fan-out on write: precompute notification list on write
    ✓ Fast reads        ✗ Expensive for celebrities (10M followers = 10M writes)
  Fan-out on read: compute recipients at read time
    ✓ No write amplification  ✗ Slower reads
  Hybrid: fan-out on write for normal users, fan-out on read for celebrities
```

---

## Step 6: Non-Functional Requirements

### Availability
```
Eliminate every Single Point of Failure (SPOF):
  □ Multiple API servers behind LB
  □ DB primary + replica (failover)
  □ Multi-AZ deployment
  □ Circuit breaker for downstream calls
  □ Health checks + auto-restart / auto-scaling

SLA math:
  Two services in series:   99.9% × 99.9% = 99.8%  (availability DROPS with each dep)
  Two services in parallel: 1 - (0.1% × 0.1%) = 99.9999%  (redundancy multiplies)
```

### Consistency
```
CAP Theorem: during network partition, choose Consistency OR Availability
  CP: consistent but may be unavailable (HBase, ZooKeeper, etcd)
  AP: available but may return stale data (Cassandra, DynamoDB, CouchDB)

Consistency levels (weakest → strongest):
  Eventual:          writes propagate eventually (DNS, social feeds — OK)
  Read-your-writes:  you always see your own writes
  Monotonic read:    never see older data after seeing newer
  Strong:            all reads see latest write (requires sync replication — expensive)
```

### Security
```
□ HTTPS everywhere — no plain HTTP
□ Auth: JWT (stateless) or sessions (stateful, needs Redis store)
□ Rate limiting: per IP, per user, per API key
□ Input validation: SQL injection, XSS prevention
□ Encrypt at rest (AES-256) and in transit (TLS 1.3)
□ Principle of least privilege: services only access what they need
□ Secrets management: never hardcode keys — use Vault, AWS Secrets Manager
```

---

## Step 7: Wrap-Up (3–5 min)

```
□ Recap:        "We designed X. Key path: Client → CDN → LB → API → Cache/DB"
□ Bottlenecks:  "Primary bottleneck at scale is DB writes — solved with sharding on userId"
□ Trade-offs:   "We chose eventual consistency for feed — users may see stale data for ~seconds"
□ Missing:      "In prod I'd add: distributed tracing, alerting, DR plan, data retention policy"
□ Scale path:   "This handles 10K QPS. For 100K QPS: add read replicas + shard the DB"
```

---

# PART B — LLD Template (Low-Level Design)

---

## LLD Interview Structure (45 min)

```
0–5  min:  Clarify requirements, identify entities
5–10 min:  Define interfaces / API
10–25 min: Class diagram — entities, relationships, key methods
25–35 min: Implement core logic (interviewer-specified part)
35–40 min: Edge cases, error handling, concurrency
40–45 min: Design patterns used, scalability discussion
```

---

## Step 1: Entity Extraction

```
From requirements, extract:
  Nouns      → Classes / Entities
  Verbs      → Methods / Operations
  Adjectives → Attributes / Properties

Example: "Design a parking lot"
  Nouns:      ParkingLot, Floor, Spot, Vehicle, Ticket, Payment, Gate
  Verbs:      park, unpark, findSpot, issueTicket, calculateFee, processPayment
  Adjectives: available/occupied (spot), compact/large/regular (spot type)
```

---

## Step 2: Define Interfaces First

```java
// Always start with interfaces — not implementations
// This enforces SOLID from the beginning

public interface ParkingLotService {
    Ticket park(Vehicle vehicle);
    Receipt unpark(String ticketId);
    int availableSpots(VehicleType type);
}

public interface PricingStrategy {
    BigDecimal calculate(Ticket ticket);
}

public interface SpotAllocationStrategy {
    Optional<ParkingSpot> findSpot(List<ParkingSpot> spots, Vehicle vehicle);
}
```

---

## Step 3: Core Class Template

```java
// ─── ENUMS ───────────────────────────────────────────────────────
public enum VehicleType  { MOTORCYCLE, CAR, TRUCK }
public enum SpotType     { COMPACT, REGULAR, LARGE }
public enum TicketStatus { ISSUED, ACTIVE, PAID, CLOSED }

// ─── VALUE OBJECTS (immutable) ───────────────────────────────────
public record Vehicle(String plateNumber, VehicleType type) {}

public record Ticket(
    String ticketId,
    Vehicle vehicle,
    ParkingSpot spot,
    LocalDateTime entryTime,
    TicketStatus status
) {
    public static Ticket issue(Vehicle vehicle, ParkingSpot spot) {
        return new Ticket(UUID.randomUUID().toString(), vehicle, spot,
                          LocalDateTime.now(), TicketStatus.ISSUED);
    }
}

// ─── ENTITIES (mutable state) ────────────────────────────────────
public class ParkingSpot {
    private final String spotId;
    private final SpotType type;
    private final int floor;
    private boolean occupied;

    public synchronized boolean occupy() {
        if (occupied) return false;
        this.occupied = true;
        return true;
    }
    public synchronized void vacate()  { this.occupied = false; }
    public boolean isAvailable()       { return !occupied; }
    public SpotType getType()          { return type; }
    public int getFloor()              { return floor; }
}

// ─── STRATEGY: PRICING ───────────────────────────────────────────
public class HourlyPricingStrategy implements PricingStrategy {
    private static final Map<SpotType, BigDecimal> RATES = Map.of(
        SpotType.COMPACT, new BigDecimal("2.00"),
        SpotType.REGULAR, new BigDecimal("3.00"),
        SpotType.LARGE,   new BigDecimal("5.00")
    );

    @Override
    public BigDecimal calculate(Ticket ticket) {
        long hours = ChronoUnit.HOURS.between(ticket.entryTime(), LocalDateTime.now());
        hours = Math.max(1, hours); // Minimum 1 hour
        return RATES.get(ticket.spot().type()).multiply(BigDecimal.valueOf(hours));
    }
}

// ─── STRATEGY: SPOT ALLOCATION ───────────────────────────────────
public class NearestSpotStrategy implements SpotAllocationStrategy {
    @Override
    public Optional<ParkingSpot> findSpot(List<ParkingSpot> spots, Vehicle vehicle) {
        SpotType required = mapVehicleToSpot(vehicle.type());
        return spots.stream()
            .filter(s -> s.getType() == required && s.isAvailable())
            .min(Comparator.comparingInt(ParkingSpot::getFloor));
    }

    private SpotType mapVehicleToSpot(VehicleType type) {
        return switch (type) {
            case MOTORCYCLE -> SpotType.COMPACT;
            case CAR        -> SpotType.REGULAR;
            case TRUCK      -> SpotType.LARGE;
        };
    }
}

// ─── CORE SERVICE ────────────────────────────────────────────────
public class ParkingLotServiceImpl implements ParkingLotService {
    private final List<ParkingSpot> spots;
    private final SpotAllocationStrategy allocationStrategy;
    private final PricingStrategy pricingStrategy;
    private final Map<String, Ticket> activeTickets = new ConcurrentHashMap<>();

    public ParkingLotServiceImpl(List<ParkingSpot> spots,
                                 SpotAllocationStrategy allocationStrategy,
                                 PricingStrategy pricingStrategy) {
        this.spots = spots;
        this.allocationStrategy = allocationStrategy;
        this.pricingStrategy = pricingStrategy;
    }

    @Override
    public Ticket park(Vehicle vehicle) {
        ParkingSpot spot = allocationStrategy.findSpot(spots, vehicle)
            .orElseThrow(() -> new ParkingFullException("No spot for " + vehicle.type()));

        if (!spot.occupy())
            throw new SpotAlreadyOccupiedException(spot.getSpotId());

        Ticket ticket = Ticket.issue(vehicle, spot);
        activeTickets.put(ticket.ticketId(), ticket);
        return ticket;
    }

    @Override
    public Receipt unpark(String ticketId) {
        Ticket ticket = activeTickets.remove(ticketId);
        if (ticket == null) throw new InvalidTicketException(ticketId);

        BigDecimal fee = pricingStrategy.calculate(ticket);
        ticket.spot().vacate();
        return new Receipt(ticketId, fee, LocalDateTime.now());
    }

    @Override
    public int availableSpots(VehicleType type) {
        SpotType required = new NearestSpotStrategy().mapVehicleToSpot(type);
        return (int) spots.stream()
            .filter(s -> s.getType() == required && s.isAvailable())
            .count();
    }
}
```

---

## Step 4: LLD Edge Cases Checklist

```
□ Concurrency:         Two threads occupy same spot? → synchronized on spot.occupy()
□ Invalid input:       Null vehicle, empty ticketId → validate at boundary
□ Resource exhausted:  No spots left → throw meaningful exception, not NPE
□ Invalid transitions: Can CLOSED ticket be reopened? → state machine check
□ Orphaned resources:  System crash after park but before ticket saved → idempotency key
□ Large scale hook:    ConcurrentHashMap, avoid synchronized on entire list
□ Time zones:          Store all times as UTC, convert at display layer
```

---

## Step 5: Design Patterns Cheat Sheet for LLD

| Pattern | When to use in LLD | Example |
|---|---|---|
| **Strategy** | Interchangeable algorithms | Pricing, spot allocation, routing |
| **Factory** | Decouple object creation | SpotFactory, VehicleFactory |
| **Builder** | Many optional constructor params | Ticket.builder().vehicle().spot().build() |
| **Observer** | React to state changes | On park → notify billing, security |
| **State** | Object changes behavior with state | Ticket: ISSUED→ACTIVE→PAID→CLOSED |
| **Singleton** | One shared resource | Config, connection pool |
| **Decorator** | Wrap to add behavior | LoggingPricingStrategy wraps Hourly |
| **Template Method** | Common algorithm, variable steps | AbstractPricingStrategy |
| **Command** | Undo/redo, queue operations | AdminPriceOverrideCommand |
| **Repository** | Abstract data access | TicketRepository, SpotRepository |

---

## Common LLD Questions — Entity Map

| Question | Core Entities | Key Patterns |
|---|---|---|
| Parking Lot | Lot, Floor, Spot, Vehicle, Ticket | Strategy (pricing, allocation), State (ticket) |
| Library Management | Book, Member, Loan, Fine, Catalog | State (loan), Observer (overdue notify) |
| Hotel Reservation | Hotel, Room, Booking, Guest, Payment | Strategy (pricing), State (booking lifecycle) |
| Elevator System | Elevator, Floor, Request, Scheduler | State (idle/moving/open), Strategy (scheduling) |
| ATM Machine | ATM, Card, Account, Transaction | State (idle/card/pin/txn), Chain of Responsibility |
| Chess Game | Board, Piece, Player, Move, Game | Strategy (move rules per piece), State (game) |
| Food Delivery | Restaurant, Menu, Order, Driver | State (order lifecycle), Observer (tracking) |
| Movie Booking | Cinema, Hall, Seat, Show, Booking | State (seat: available/locked/booked) |
| Ride Sharing | Driver, Rider, Trip, Pricing | Strategy (pricing, matching), Observer |
| Vending Machine | Machine, Slot, Product, Coin | State pattern (the core of the problem) |

---

# PART C — Universal Cheat Sheet

---

## Scaling Levels

```
Level 1 — 1K QPS:    Single server + single DB
Level 2 — 10K QPS:   Add Redis cache + read replicas + CDN
Level 3 — 100K QPS:  LB + multiple stateless API servers + connection pooling
Level 4 — 1M QPS:    DB sharding + message queues + async processing
Level 5 — 10M+ QPS:  Geo-distribution + microservices + event sourcing + global CDN
```

## Trade-offs to Always Mention Out Loud

```
□ SQL vs NoSQL:               consistency + schema  vs  flexibility + scale
□ Cache:                      performance           vs  staleness
□ Sync vs Async:              latency               vs  throughput
□ Strong vs Eventual:         correctness           vs  availability
□ Monolith vs Microservices:  simplicity            vs  independent scaling
□ Fan-out on write vs read:   fast reads            vs  expensive writes
□ Push vs Pull:               server overhead       vs  client polling
□ Normalization vs Denorm:    consistency           vs  query performance
```

## Availability Numbers

| SLA | Downtime per year |
|---|---|
| 99% | 3.65 days |
| 99.9% | 8.7 hours |
| 99.99% | 52 minutes |
| 99.999% | 5 minutes |

## Single Machine Limits (rough)

| Component | Throughput |
|---|---|
| MySQL (writes) | ~5K–10K writes/sec |
| PostgreSQL | ~10K–20K writes/sec |
| Redis | ~100K–1M ops/sec |
| Kafka | ~1M+ messages/sec |
| Nginx | ~50K–100K req/sec |

## Interview Time Box (45 min)

```
[0–5  min]  Clarify: functional + non-functional requirements
[5–8  min]  Back-of-envelope estimation
[8–12 min]  API design
[12–25 min] High-level architecture + component decisions
[25–38 min] Deep dive on 2 components (interviewer-driven)
[38–43 min] NFRs: availability, consistency, security
[43–45 min] Wrap-up: bottlenecks, trade-offs, what's missing
```

## Red Flags to Avoid

```
❌ Jumping to solution without clarifying requirements
❌ Single server with no replication (always a SPOF)
❌ Offset-based pagination for large datasets (use cursor)
❌ Not mentioning failure modes (what if DB goes down?)
❌ Over-engineering LLD (don't build a framework for a parking lot)
❌ Under-engineering HLD (don't say "just use a DB" for 1M QPS)
❌ Forgetting API design before architecture
❌ Not saying trade-offs out loud — every choice has a cost
❌ Going silent for more than 30 seconds — narrate your thinking
```
