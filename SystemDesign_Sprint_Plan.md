# System Design Interview Sprint Plan
### 13-Day Plan · 6–7 hrs/day · HLD + Distributed Systems · All Companies
### Local Repo: `/Desktop/resume/system-design-notes/` · Remote: [liquidslr/system-design-notes](https://github.com/liquidslr/system-design-notes)

---

## TL;DR

> You have 28 chapters locally. This plan maps each chapter to a day, adds real GitHub repos + engineering blog links for every topic, and ends with timed company-specific mock rounds. By Day 13 you can walk into any system design interview and draw a complete architecture from scratch.

---

## How to Use This Plan

- **AM block (3 hrs):** Read your local notes (`Readme.md` in each folder) + the linked GitHub repo
- **PM block (3–4 hrs):** Practice — draw the architecture on iPad (GoodNotes), then answer the mock questions at the bottom of each day
- **Daily ritual:** After PM block, write one page in GoodNotes: *"What are the 3 bottlenecks in today's system and how do you fix them?"*
- **Mark progress:** `[ ]` not started → `[~]` read → `[x]` can explain cold

---

## Phase Structure

| Phase | Days | Goal |
|---|---|---|
| **Phase 1 — Foundations** | Day 1–3 | Scaling, estimation, framework, core building blocks |
| **Phase 2 — Core Systems** | Day 4–8 | Rate limiter → Chat → Search → YouTube → Drive |
| **Phase 3 — Advanced Systems** | Day 9–11 | Payments, distributed queues, real-time, maps |
| **Phase 4 — Mock Rounds** | Day 12–13 | Timed company mocks, end-to-end walkthroughs |

---

## The 10 Building Blocks (Know These First)

Every system design answer assembles these. Before Day 1, make sure you can explain each in 2 sentences:

| Block | What it does | Key tool |
|---|---|---|
| **Load Balancer** | Distributes traffic across servers | Nginx, AWS ALB |
| **CDN** | Serves static content from edge nodes | CloudFront, Cloudflare |
| **Cache** | Reduces DB reads, stores hot data | Redis, Memcached |
| **Message Queue** | Decouples producers from consumers | Kafka, RabbitMQ, SQS |
| **SQL DB** | ACID transactions, structured data | PostgreSQL, MySQL |
| **NoSQL DB** | Flexible schema, horizontal scale | Cassandra, DynamoDB, MongoDB |
| **Blob Storage** | Unstructured files, images, videos | S3, GCS |
| **Search Index** | Full-text search, inverted index | Elasticsearch |
| **API Gateway** | Auth, rate limit, routing at entry | Kong, AWS API GW |
| **Consistent Hashing** | Distribute keys across nodes without full remap | Custom / Redis Cluster |

---

---

## PHASE 1 — Foundations (Day 1–3)

---

### Day 1 — Scaling + Back-of-Envelope Estimation

**Local notes:**
- [`01. Scaling/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/01.%20Scaling/)
- [`02. Back Of the Envelope Estimation/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/02.%20Back%20Of%20the%20Envelope%20Estimation/)

**What to master today:**
- Vertical vs horizontal scaling — when each breaks
- Stateless vs stateful servers — why stateless scales
- DB replication (master-replica), sharding strategies (range vs hash vs directory)
- CDN push vs pull
- Back-of-envelope: QPS, storage, bandwidth math

**Reference repos + resources:**
- [System Design Primer (donnemartin)](https://github.com/donnemartin/system-design-primer) — the most comprehensive open-source SD repo, 280k stars
- [Scalability for Dummies](https://www.lecloud.net/tagged/scalability) — classic blog, 4 parts
- [High Scalability](http://highscalability.com) — real architecture case studies

**Numbers to memorize (write in GoodNotes):**

| Metric | Value |
|---|---|
| L1 cache ref | 0.5 ns |
| L2 cache ref | 7 ns |
| RAM read | 100 ns |
| SSD random read | 150 µs |
| Network roundtrip (same DC) | 0.5 ms |
| Network roundtrip (CA → NL) | 150 ms |
| 1 million requests/day | ~12 QPS |
| 1 billion requests/day | ~12,000 QPS |

**Mock questions (answer out loud, 10 min each):**
1. How would you scale a web server from 1 user to 1 million?
2. A system gets 10M DAU. Each user uploads 1 photo/day (avg 300KB). How much storage per year?
3. Your DB is slow. Walk me through every option you'd try before sharding.

---

### Day 2 — System Design Framework + Consistent Hashing

**Local notes:**
- [`03. System Design Framework/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/03.%20System%20Design%20Framework/)
- [`05. Consistent Hashing/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/05.%20Consistent%20Hashing/)

**What to master today:**
- The 4-step framework: Clarify → High-level design → Deep-dive → Wrap up
- Consistent hashing: virtual nodes, why it solves hotspot problem
- What happens when a node joins/leaves a hash ring

**Reference repos + resources:**
- [Consistent Hashing implementation (Java)](https://github.com/Jaskey/ConsistentHash) — clean Java implementation with virtual nodes
- [Consistent Hashing (Python)](https://github.com/Doist/uhashring) — production-grade
- [Stanford lecture on Consistent Hashing](http://theory.stanford.edu/~tim/s16/l/l1.pdf) — theory foundation
- [Google Maglev paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44824.pdf) — how Google uses it in load balancers

**Framework template (write this on iPad — use every interview):**

```
Step 1 — Clarify (5 min)
  → Who are the users? Scale? Read-heavy or write-heavy?
  → Functional requirements (what it does)
  → Non-functional requirements (latency, availability, consistency)

Step 2 — High-level boxes (10 min)
  → Client → API Gateway → Services → DB/Cache → Queue

Step 3 — Deep-dive on 2–3 components (20 min)
  → The interviewer will guide you here
  → Typical: DB schema, caching strategy, handling failures

Step 4 — Wrap up (5 min)
  → Bottlenecks, monitoring, future scale
```

**Mock questions:**
1. Walk me through how you'd approach designing Instagram from scratch.
2. A node in your Redis cluster goes down. How does consistent hashing minimize data loss?
3. Why do you need virtual nodes in consistent hashing? What problem do they solve?

---

### Day 3 — Key-Value Store + Unique ID Generator

**Local notes:**
- [`06. Key-Value Store/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/06.%20Key-Value%20Store/)
- [`07. Unique-Id Generator/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/07.%20Unique-Id%20Generator/)

**What to master today:**
- CAP theorem — pick 2: consistency, availability, partition tolerance
- Gossip protocol for node failure detection
- Merkle trees for data inconsistency detection
- Snowflake ID: 41-bit timestamp + 10-bit machine ID + 12-bit sequence = 64-bit
- UUID vs Snowflake vs Ticket Server — trade-offs

**Reference repos + resources:**
- [Twitter Snowflake (original)](https://github.com/twitter-archive/snowflake) — archived but readable
- [Snowflake Java impl](https://github.com/callicoder/java-snowflake) — clean Java version
- [Amazon Dynamo paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — foundational paper
- [Apache Cassandra architecture](https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/architecture/archIntro.html) — gossip + consistent hashing in practice
- [Ticket server (Flickr)](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/) — simple distributed ID approach

**Mock questions:**
1. Design a key-value store like Redis. What happens when a node fails?
2. Your system needs globally unique IDs across 100 servers. Walk me through 3 approaches.
3. Choose between Cassandra and DynamoDB for a social media app. Justify.

---

---

## PHASE 2 — Core Systems (Day 4–8)

---

### Day 4 — Rate Limiter + URL Shortener

**Local notes:**
- [`04. Rate Limiter/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/04.%20Rate%20Limiter/)
- [`08. URL Shortener/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/08.%20URL%20Shortener/)

**What to master today:**

**Rate Limiter:**
- Token Bucket vs Leaking Bucket vs Sliding Window Counter — know when each is used
- Where to place it: client vs server vs middleware (API Gateway is best)
- Redis INCR + EXPIRE for distributed rate limiting
- Race condition in distributed rate limit → Lua scripts in Redis (atomic)

**URL Shortener:**
- Hash function: MD5 (128-bit) → take first 7 chars → 3.5 trillion combinations
- Collision handling: check DB, if exists append predefined string + rehash
- 301 (permanent, browser caches) vs 302 (temporary, goes through server every time) — **302 for analytics**
- Read-heavy → cache popular URLs in Redis

**Reference repos + resources:**
- [Uber Rate Limiter (Go)](https://github.com/uber-go/ratelimit) — production leaky bucket
- [Redis rate limiting patterns](https://redis.io/docs/manual/patterns/distributed-locks/) — official Redis docs
- [Rate limiter in Java (Resilience4j)](https://github.com/resilience4j/resilience4j) — production Java library
- [URL shortener system design (Java)](https://github.com/qiangmzsx/Software-Engineering-at-Google) — design reference
- [Bitly engineering blog](https://word.bitly.com/post/28069468495/seven-databases-in-seven-weeks) — real case study

**Mock questions:**
1. Design a rate limiter for an API that allows 100 req/min per user. How do you handle distributed servers?
2. Design TinyURL. How do you guarantee uniqueness? What if two users submit the same long URL?
3. Your rate limiter uses Redis. Redis goes down. What happens? How do you handle it?

---

### Day 5 — Web Crawler + Notification System

**Local notes:**
- [`09. Web Crawler/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/09.%20Web%20Crawler/)
- [`10. Notification System/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/10.%20Notification%20System/)

**What to master today:**

**Web Crawler:**
- BFS traversal of the web — URL frontier (priority queue)
- Politeness: respect robots.txt, rate limit per domain
- Dedup: BloomFilter for visited URLs (memory efficient)
- DNS caching to avoid repeated lookups
- Distributed crawling: partition URL space by domain hash

**Notification System:**
- Push (FCM/APNs) vs SMS (Twilio) vs Email (SendGrid/Mailchimp)
- Fanout on write (push model) vs fanout on read (pull model)
- Retry with exponential backoff for failed deliveries
- Queue per channel: Kafka topic per notification type
- Rate limiting notifications per user (don't spam)

**Reference repos + resources:**
- [Scrapy (Python crawler)](https://github.com/scrapy/scrapy) — 51k stars, production crawler
- [Apache Nutch](https://github.com/apache/nutch) — distributed Java crawler
- [BloomFilter Java](https://github.com/google/guava/blob/master/guava/src/com/google/common/hash/BloomFilter.java) — Guava's implementation
- [FCM docs](https://firebase.google.com/docs/cloud-messaging) — Google push notification
- [Notify.js](https://github.com/nicowillis/Notify.js) — lightweight notification reference

**Mock questions:**
1. Design a web crawler for Google. How do you prioritize important pages?
2. Design a push notification system for 100M users. A celebrity posts — how do you fan out?
3. Your notification service is sending duplicate notifications. What could cause this? How do you fix it?

---

### Day 6 — News Feed + Chat System

**Local notes:**
- [`11. News Feed System/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/11.%20News%20Feed%20System/)
- [`12. Chat System/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/12.%20Chat%20System/)

**What to master today:**

**News Feed:**
- Fanout on write: pre-compute feed on post → fast read, slow write, waste for inactive users
- Fanout on read: compute on request → slow read, no waste for inactive users
- **Hybrid**: fanout on write for normal users, fanout on read for celebrities (>X followers)
- Feed stored in Redis sorted set (score = timestamp)
- Pagination: cursor-based (not offset — offset is O(n))

**Chat System:**
- WebSocket for real-time bidirectional communication (vs HTTP polling vs long polling)
- Message storage: Cassandra (append-only, high write throughput, row key = channel_id)
- 1:1 chat vs group chat vs channels — different fanout logic
- Online presence: heartbeat every 5s → update Redis TTL
- Discord's architecture: 1B+ messages/day in Cassandra, message_id = Snowflake

**Reference repos + resources:**
- [Facebook Messenger system design](https://engineering.fb.com/2018/01/26/core-data/messenger/) — real architecture
- [Discord stores billions of messages](https://discord.com/blog/how-discord-stores-billions-of-messages) — Cassandra deep dive ⭐
- [Slack Flannel edge cache](https://slack.engineering/flannel-an-application-level-edge-cache-to-make-slack-scale/) — how Slack scales reads
- [WebSocket vs SSE vs Long Polling](https://ably.com/blog/websockets-vs-long-polling) — definitive comparison
- [Socket.IO (JS)](https://github.com/socketio/socket.io) — 60k stars, WebSocket abstraction
- [News feed system design (reference)](https://github.com/donnemartin/system-design-primer#design-the-facebook-news-feed) — system design primer section

**Mock questions:**
1. Design Facebook News Feed. How do you handle users with 10M followers posting?
2. Design WhatsApp. How do you guarantee message delivery? What's your storage schema?
3. A user sends a message but goes offline before receiving the ACK. What happens?

---

### Day 7 — Search Autocomplete + YouTube

**Local notes:**
- [`13. Search Autocomplete/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/13.%20Search%20Autocomplete/)
- [`14. Youtube/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/14.%20Youtube/)

**What to master today:**

**Search Autocomplete:**
- Trie for prefix matching — but doesn't scale at 10B entries
- Trie + top-K results stored at each node (pre-computed popularity)
- Sharding trie by first character → 26 shards
- Update: weekly batch rebuild OR real-time stream with Kafka → aggregate → update
- BloomFilter to filter offensive terms before suggestions

**YouTube:**
- Video upload pipeline: S3 → transcoding workers → multiple formats (360p/720p/1080p/4K)
- CDN for video delivery (pre-signed URLs, edge caching)
- Adaptive bitrate streaming (ABR): HLS/DASH — auto-switch quality based on bandwidth
- Metadata in MySQL (video_id, user_id, title), view counts in Redis (eventually consistent to DB)
- Netflix uses per-shot encoding — not per-title

**Reference repos + resources:**
- [How We Built Prefixy (autocomplete at scale)](https://medium.com/@prefixyteam/how-we-built-prefixy-a-scalable-prefix-search-service-for-powering-autocomplete-c20f98e2eff1) — real case study ⭐
- [Prefix Hash Tree paper](https://people.eecs.berkeley.edu/~sylvia/papers/pht.pdf) — academic foundation
- [Trie implementation Java](https://github.com/eugenp/tutorials/tree/master/data-structures/src/main/java/com/baeldung/trie) — Baeldung reference
- [YouTube architecture (2012)](https://www.youtube.com/watch?v=w5WVu624fY8) — classic talk ⭐
- [Netflix video encoding at scale](https://netflixtechblog.com/high-quality-video-encoding-at-scale-d159db052746) — real transcoding pipeline
- [Netflix shot-based encoding](https://netflixtechblog.com/optimized-shot-based-encodes-now-streaming-4b9464204830) — advanced
- [FFmpeg](https://github.com/FFmpeg/FFmpeg) — actual transcoding tool used everywhere

**Mock questions:**
1. Design Google Search autocomplete. How do you update suggestions in real time?
2. Design YouTube. A user uploads a 4K video. Walk me through what happens end-to-end.
3. Your CDN is serving stale video content. How do you handle cache invalidation?

---

### Day 8 — Google Drive + Proximity Service

**Local notes:**
- [`15. Google Drive/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/15.%20Google%20Drive/)
- [`16. Proximity Service/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/16.%20Proximity%20Service/)

**What to master today:**

**Google Drive:**
- Chunking: split files into 4MB blocks → upload only changed chunks (delta sync)
- Differential sync: operational transform or CRDT for concurrent edits
- Block storage: deduplicate identical blocks via content hash (SHA-256) → massive space savings
- Metadata DB (MySQL): file_id, parent_folder_id, owner_id, version
- Conflict resolution: last-write-wins OR version vectors OR merge

**Proximity Service:**
- Geohash: encode lat/lng as string → prefix match = neighbors
- Quadtree: partition space recursively until ≤100 POIs per cell
- Redis GEOADD + GEORADIUS for simple proximity queries
- Business data: read-heavy → cache in Redis, update nightly batch

**Reference repos + resources:**
- [Differential Synchronization (Neil Fraser)](https://neil.fraser.name/writing/sync/) — the algorithm behind Google Docs ⭐
- [How Dropbox scaled](https://www.youtube.com/watch?v=PE4gwstWhmc) — real engineering talk ⭐
- [Geohash library (Java)](https://github.com/kungfoo/geohash-java) — production geohash
- [Redis GEO commands](https://redis.io/docs/data-types/geo/) — official docs
- [Quadtree visualization](https://github.com/timohausmann/quadtree-js) — helps understand spatial partitioning
- [Yelp engineering: proximity search](https://engineeringblog.yelp.com/2015/09/elasticsearch-at-yelp.html) — real case

**Mock questions:**
1. Design Google Drive. Two users edit the same doc simultaneously. What happens?
2. Design Yelp (find restaurants near me). How do you index 500M businesses by location?
3. You need sub-100ms proximity queries at 10k QPS. Walk through your data store choice.

---

---

## PHASE 3 — Advanced Systems (Day 9–11)

---

### Day 9 — Distributed Message Queue + Metrics Monitoring

**Local notes:**
- [`19. Distributed Message Queue/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/19.%20Distributed%20Message%20Queue/)
- [`20. Metrics Monitoring and Alerting System/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/20.%20Metrics%20Monitoring%20and%20Alerting%20System/)

**What to master today:**

**Distributed Message Queue (Kafka-level):**
- Partition = unit of parallelism. More partitions = more consumers
- Offset management: auto-commit (at-least-once) vs manual (exactly-once with idempotent producers)
- Consumer group: each partition consumed by exactly one consumer in the group
- Replication factor = 3: leader handles reads/writes, followers replicate
- Compacted topics: keep only latest value per key (like a changelog)
- Dead Letter Topic: failed messages go here after N retries

**Metrics Monitoring:**
- Time-series DB: InfluxDB, Prometheus, TimescaleDB — optimized for `(metric, timestamp, value)`
- Push vs pull model: Prometheus pulls (better for service discovery), StatsD pushes
- Aggregation: raw data at 1s → roll up to 1m → 1h → 1d (reduce storage)
- Alerting: threshold + anomaly detection (moving average, 3-sigma)
- Four Golden Signals: Latency, Traffic, Errors, Saturation

**Reference repos + resources:**
- [Apache Kafka](https://github.com/apache/kafka) — the source ⭐
- [Kafka Java client](https://github.com/confluentinc/kafka) — Confluent's maintained fork
- [Kafka in 5 minutes](https://kafka.apache.org/documentation/) — official docs
- [Prometheus](https://github.com/prometheus/prometheus) — 55k stars, the metrics standard
- [Grafana](https://github.com/grafana/grafana) — 62k stars, dashboards
- [InfluxDB](https://github.com/influxdata/influxdb) — time-series DB
- [Designing Kafka (Jay Kreps)](https://notes.stephenholiday.com/Kafka.pdf) — original paper ⭐

**Mock questions:**
1. Design a Kafka-like message queue. How do you guarantee exactly-once delivery?
2. Design a metrics system for 10k microservices. How do you store 1B data points/day efficiently?
3. Your consumer is 2 hours behind on a Kafka partition. Walk through root causes and fixes.

---

### Day 10 — Payment System + Ad Click Aggregation

**Local notes:**
- [`26. Payment System/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/26.%20Payment%20System/)
- [`21. Ad Click Event Aggregation/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/21.%20Ad%20Click%20Event%20Aggregation/)

**What to master today:**

**Payment System:**
- Idempotency key: client sends UUID with each request → server dedupes → safe to retry
- Double-entry bookkeeping: every transaction = debit one account + credit another (sum = 0)
- Saga pattern: distributed transaction across services — compensating transactions on failure
- Two-phase commit (2PC): coordinator + participants — slow but consistent
- PSP (Payment Service Provider): Stripe, Adyen handle card networks — you call their API
- Reconciliation: end-of-day batch compare your DB vs PSP records → fix mismatches

**Ad Click Aggregation:**
- High write volume: Kafka → stream processor (Flink/Spark) → aggregate → store
- Lambda architecture: batch layer (accurate) + speed layer (real-time approximate) → merge
- Kappa architecture: one Kafka stream handles both — simpler, preferred now
- Watermarking: handle late-arriving events (event time vs processing time)
- Deduplication: same click event arriving twice → use click_id as idempotency key

**Reference repos + resources:**
- [Stripe API docs](https://stripe.com/docs/api) — how production payments work
- [Saga pattern (microservices.io)](https://microservices.io/patterns/data/saga.html) — the canonical explanation ⭐
- [Outbox pattern](https://microservices.io/patterns/data/transactional-outbox.html) — reliable event publishing ⭐
- [Apache Flink](https://github.com/apache/flink) — stream processing for ad aggregation
- [Apache Spark](https://github.com/apache/spark) — batch + stream processing
- [Martin Kleppmann - Designing Data-Intensive Applications](https://github.com/ept/ddia-references) — references from the book ⭐
- [Square payment system engineering](https://developer.squareup.com/blog/) — real case studies

**Mock questions:**
1. Design a payment system like PayPal. How do you prevent double charges?
2. Design an ad click aggregation system. 1B clicks/day. Count clicks per ad per minute.
3. A payment goes through on the PSP but your DB write fails. What's your recovery strategy?

---

### Day 11 — Google Maps + Nearby Friends + Stock Exchange

**Local notes:**
- [`18. Google Maps/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/18.%20Google%20Maps/)
- [`17. Nearby Friends/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/17.%20Nearby%20Friends/)
- [`28. Stock Exchange/Readme.md`](https://github.com/liquidslr/system-design-notes/tree/main/28.%20Stock%20Exchange/)

**What to master today:**

**Google Maps:**
- Map tiles: pre-rendered PNGs at zoom levels → CDN cached
- Routing: graph of road segments → Dijkstra/A* for shortest path
- Graph too large → partition into cells → inter-cell routing hierarchy
- ETA: historical speed data per road segment per time-of-day → weighted Dijkstra
- Live traffic: GPS pings from mobile apps → aggregate → update road weights

**Nearby Friends:**
- Location update: mobile → WebSocket server → Redis (user_id → lat/lng)
- Fan out: when you update location, push to all friends via pub/sub
- Pub/Sub: Redis pub/sub per user_id channel
- Efficiency: only compute distance for friends within bounding box first

**Stock Exchange:**
- Matching engine: price-time priority order book
- Order types: market (execute now) vs limit (execute at price) vs stop
- LMAX Disruptor pattern: single-threaded matching engine + ring buffer = microsecond latency
- Eventual consistency for portfolio display, strong consistency for order matching
- Market data feed: separate read path — broadcast via UDP multicast to subscribers

**Reference repos + resources:**
- [OSRM (open source routing)](https://github.com/Project-OSRM/osrm-backend) — production road routing engine ⭐
- [OpenStreetMap](https://github.com/openstreetmap) — the data behind maps
- [LMAX Disruptor](https://github.com/LMAX-Exchange/disruptor) — the actual exchange ring buffer ⭐
- [How LMAX works](https://lmax-exchange.github.io/disruptor/disruptor.html) — mechanical sympathy blog
- [Coinbase matching engine post](https://www.coinbase.com/blog/how-we-scaled-coinbase-matching-engine) — real case study ⭐
- [Redis pub/sub docs](https://redis.io/docs/manual/pubsub/) — for Nearby Friends pattern

**Mock questions:**
1. Design Google Maps routing. How do you compute fastest path across 100M road nodes?
2. Design a stock exchange matching engine. How do you achieve microsecond latency?
3. Design Nearby Friends (like Snapchat). 100M users, location updates every 5s. How do you fan out?

---

---

## PHASE 4 — Company Mock Rounds (Day 12–13)

### Mock Format
```
Total: 45 min per mock (real interview pace)
  → 5 min:  Clarify requirements (ask 3–4 questions)
  → 15 min: Draw high-level architecture (boxes + arrows)
  → 15 min: Deep-dive on 2 components the interviewer picks
  → 5 min:  Failure modes + monitoring + scale
  → 5 min:  Self-review — what did you skip?
```

---

### Day 12 — JusPay + JPMorgan Mocks

**JusPay focuses on:** Payment flows, distributed systems, Kafka, idempotency, ledger design

| Mock # | Question | Key concepts to hit |
|---|---|---|
| Mock 1 | Design a payment gateway (like JusPay) | Idempotency key, Saga pattern, PSP integration, retry with backoff |
| Mock 2 | Design a digital wallet | Double-entry bookkeeping, ACID transactions, event sourcing |
| Mock 3 | Design a real-time fraud detection system | Kafka stream, feature store, ML scoring, sub-100ms latency |

**Local notes for reference:**
- [`26. Payment System/`](https://github.com/liquidslr/system-design-notes/tree/main/26.%20Payment%20System/)
- [`27. Digital Wallet/`](https://github.com/liquidslr/system-design-notes/tree/main/27.%20%20Digital%20Wallet/)

---

**JPMorgan focuses on:** Distributed systems, data consistency, stock/trading systems, resilience

| Mock # | Question | Key concepts to hit |
|---|---|---|
| Mock 1 | Design a stock exchange order matching system | Order book, LMAX Disruptor, price-time priority, market data feed |
| Mock 2 | Design a real-time metrics dashboard for trading | Time-series DB, WebSocket push, aggregation pipeline |
| Mock 3 | Design a distributed transaction system | 2PC vs Saga, Outbox pattern, idempotency, compensating transactions |

**Local notes for reference:**
- [`28. Stock Exchange/`](https://github.com/liquidslr/system-design-notes/tree/main/28.%20Stock%20Exchange/)
- [`19. Distributed Message Queue/`](https://github.com/liquidslr/system-design-notes/tree/main/19.%20Distributed%20Message%20Queue/)

---

### Day 13 — Amazon + Microsoft + Salesforce Mocks

**Amazon focuses on:** Scale, fault tolerance, leadership principles woven into design

| Mock # | Question | Key concepts to hit |
|---|---|---|
| Mock 1 | Design Amazon's notification system | Fanout, Kafka, FCM/APNs/SMS, retry, dedup |
| Mock 2 | Design S3-like object storage | Chunk → replicate → consistent hashing, metadata service, pre-signed URLs |
| Mock 3 | Design a distributed job scheduler | Priority queue, worker pool, at-least-once, idempotent jobs, cron |

**Local notes for reference:**
- [`24. S3-like Object Storage/`](https://github.com/liquidslr/system-design-notes/tree/main/24.%20S3-like%20Object%20Storage/)
- [`10. Notification System/`](https://github.com/liquidslr/system-design-notes/tree/main/10.%20Notification%20System/)

---

**Microsoft focuses on:** Deep technical correctness, edge cases, distributed consensus

| Mock # | Question | Key concepts to hit |
|---|---|---|
| Mock 1 | Design a distributed key-value store | Consistent hashing, replication, conflict resolution (vector clocks) |
| Mock 2 | Design a collaborative document editor | OT / CRDT, WebSocket, version history, conflict merge |
| Mock 3 | Design Azure Service Bus (message queue) | Kafka internals, partition, consumer group, exactly-once |

---

**Salesforce focuses on:** Multi-tenancy, API design, clean architecture, CRM scale

| Mock # | Question | Key concepts to hit |
|---|---|---|
| Mock 1 | Design a multi-tenant SaaS platform | Tenant isolation (schema-per-tenant vs row-level), rate limiting per tenant |
| Mock 2 | Design a search system for CRM data | Elasticsearch, inverted index, real-time indexing via Kafka |
| Mock 3 | Design a notification/email system | Fanout, dedup, unsubscribe handling, bounce tracking |

---

---

## Quick Reference: Topic → Local Folder → Key Repo

| Topic | Local Folder | GitHub Repo |
|---|---|---|
| Scaling | [`01. Scaling/`](https://github.com/liquidslr/system-design-notes/tree/main/01.%20Scaling/) | [system-design-primer](https://github.com/donnemartin/system-design-primer) |
| Back-of-envelope | [`02. Back Of the Envelope Estimation/`](https://github.com/liquidslr/system-design-notes/tree/main/02.%20Back%20Of%20the%20Envelope%20Estimation/) | [system-design-primer](https://github.com/donnemartin/system-design-primer) |
| SD Framework | [`03. System Design Framework/`](https://github.com/liquidslr/system-design-notes/tree/main/03.%20System%20Design%20Framework/) | [Grokking SD](https://github.com/Sairyss/system-design-patterns) |
| Rate Limiter | [`04. Rate Limiter/`](https://github.com/liquidslr/system-design-notes/tree/main/04.%20Rate%20Limiter/) | [uber-go/ratelimit](https://github.com/uber-go/ratelimit) |
| Consistent Hashing | [`05. Consistent Hashing/`](https://github.com/liquidslr/system-design-notes/tree/main/05.%20Consistent%20Hashing/) | [Jaskey/ConsistentHash](https://github.com/Jaskey/ConsistentHash) |
| Key-Value Store | [`06. Key-Value Store/`](https://github.com/liquidslr/system-design-notes/tree/main/06.%20Key-Value%20Store/) | [Twitter Snowflake](https://github.com/twitter-archive/snowflake) |
| Unique ID | [`07. Unique-Id Generator/`](https://github.com/liquidslr/system-design-notes/tree/main/07.%20Unique-Id%20Generator/) | [callicoder/java-snowflake](https://github.com/callicoder/java-snowflake) |
| URL Shortener | [`08. URL Shortener/`](https://github.com/liquidslr/system-design-notes/tree/main/08.%20URL%20Shortener/) | [system-design-primer](https://github.com/donnemartin/system-design-primer#design-pastebin) |
| Web Crawler | [`09. Web Crawler/`](https://github.com/liquidslr/system-design-notes/tree/main/09.%20Web%20Crawler/) | [scrapy/scrapy](https://github.com/scrapy/scrapy) |
| Notification System | [`10. Notification System/`](https://github.com/liquidslr/system-design-notes/tree/main/10.%20Notification%20System/) | [firebase/firebase-admin-java](https://github.com/firebase/firebase-admin-java) |
| News Feed | [`11. News Feed System/`](https://github.com/liquidslr/system-design-notes/tree/main/11.%20News%20Feed%20System/) | [system-design-primer#news-feed](https://github.com/donnemartin/system-design-primer#design-the-facebook-news-feed) |
| Chat System | [`12. Chat System/`](https://github.com/liquidslr/system-design-notes/tree/main/12.%20Chat%20System/) | [socketio/socket.io](https://github.com/socketio/socket.io) |
| Search Autocomplete | [`13. Search Autocomplete/`](https://github.com/liquidslr/system-design-notes/tree/main/13.%20Search%20Autocomplete/) | [eugenp/tutorials trie](https://github.com/eugenp/tutorials/tree/master/data-structures) |
| YouTube | [`14. Youtube/`](https://github.com/liquidslr/system-design-notes/tree/main/14.%20Youtube/) | [FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg) |
| Google Drive | [`15. Google Drive/`](https://github.com/liquidslr/system-design-notes/tree/main/15.%20Google%20Drive/) | [Differential sync](https://neil.fraser.name/writing/sync/) |
| Proximity Service | [`16. Proximity Service/`](https://github.com/liquidslr/system-design-notes/tree/main/16.%20Proximity%20Service/) | [kungfoo/geohash-java](https://github.com/kungfoo/geohash-java) |
| Nearby Friends | [`17. Nearby Friends/`](https://github.com/liquidslr/system-design-notes/tree/main/17.%20Nearby%20Friends/) | [Redis GEO](https://redis.io/docs/data-types/geo/) |
| Google Maps | [`18. Google Maps/`](https://github.com/liquidslr/system-design-notes/tree/main/18.%20Google%20Maps/) | [Project-OSRM](https://github.com/Project-OSRM/osrm-backend) |
| Message Queue | [`19. Distributed Message Queue/`](https://github.com/liquidslr/system-design-notes/tree/main/19.%20Distributed%20Message%20Queue/) | [apache/kafka](https://github.com/apache/kafka) |
| Metrics Monitoring | [`20. Metrics Monitoring and Alerting System/`](https://github.com/liquidslr/system-design-notes/tree/main/20.%20Metrics%20Monitoring%20and%20Alerting%20System/) | [prometheus/prometheus](https://github.com/prometheus/prometheus) |
| Ad Click Aggregation | [`21. Ad Click Event Aggregation/`](https://github.com/liquidslr/system-design-notes/tree/main/21.%20Ad%20Click%20Event%20Aggregation/) | [apache/flink](https://github.com/apache/flink) |
| Hotel Reservation | [`22. Hotel Reservation System/`](https://github.com/liquidslr/system-design-notes/tree/main/22.%20Hotel%20Reservation%20System/) | [Saga pattern](https://microservices.io/patterns/data/saga.html) |
| Distributed Email | [`23. Distributed Email Service/`](https://github.com/liquidslr/system-design-notes/tree/main/23.%20Distributed%20Email%20Service/) | [mailhog/MailHog](https://github.com/mailhog/MailHog) |
| S3 Object Storage | [`24. S3-like Object Storage/`](https://github.com/liquidslr/system-design-notes/tree/main/24.%20S3-like%20Object%20Storage/) | [minio/minio](https://github.com/minio/minio) |
| Gaming Leaderboard | [`25. Real-time Gaming Leaderboard/`](https://github.com/liquidslr/system-design-notes/tree/main/25.%20Real-time%20Gaming%20Leaderboard/) | [Redis sorted sets](https://redis.io/docs/data-types/sorted-sets/) |
| Payment System | [`26. Payment System/`](https://github.com/liquidslr/system-design-notes/tree/main/26.%20Payment%20System/) | [Outbox pattern](https://microservices.io/patterns/data/transactional-outbox.html) |
| Digital Wallet | [`27. Digital Wallet/`](https://github.com/liquidslr/system-design-notes/tree/main/27.%20%20Digital%20Wallet/) | [stripe/stripe-java](https://github.com/stripe/stripe-java) |
| Stock Exchange | [`28. Stock Exchange/`](https://github.com/liquidslr/system-design-notes/tree/main/28.%20Stock%20Exchange/) | [LMAX-Exchange/disruptor](https://github.com/LMAX-Exchange/disruptor) |

---

## GoodNotes iPad Setup for System Design

**Create this notebook: "System Design Sprint"**

```
📓 System Design Sprint
├── 📄 10 Building Blocks (one block per page — draw the component + when to use it)
├── 📄 Interview Framework (the 4-step template — memorize this)
├── 📄 Numbers Cheat Sheet (latency numbers + storage math)
├── 📄 Pattern Library
│   ├── Fanout patterns (write vs read vs hybrid)
│   ├── Caching patterns (cache-aside, write-through, write-behind)
│   ├── DB patterns (sharding, replication, CQRS)
│   └── Consistency patterns (eventual, strong, causal)
└── 📄 Mock Round Notes (one page per mock — what you drew, what you missed)
```

**For each system you study, draw:**
```
1. Box diagram (client → LB → service → DB/cache/queue)
2. Write path (what happens when user creates data)
3. Read path (what happens when user queries data)
4. Failure scenario (what breaks first, how you recover)
```

---

## Summary Stats

| Phase | Days | Systems covered |
|---|---|---|
| Foundations | 1–3 | Scaling, Estimation, Framework, Hashing, KV Store, IDs |
| Core Systems | 4–8 | Rate Limiter, URL Shortener, Crawler, Notification, Feed, Chat, Autocomplete, YouTube, Drive, Proximity |
| Advanced | 9–11 | Message Queue, Metrics, Payments, Ad Aggregation, Maps, Friends, Stock Exchange |
| Mocks | 12–13 | JusPay · JPMorgan · Amazon · Microsoft · Salesforce |
| **Total** | **13 days** | **28 systems · 5 companies** |
