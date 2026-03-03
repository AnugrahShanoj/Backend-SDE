# 📘 Backend Deep Prep – Day 13  
# Caching Strategy (Deep System-Level Understanding)

---

# 1️⃣ Why Caching Becomes Mandatory at Scale

Database access is expensive compared to memory access.

Approximate comparison:

- RAM access → Nanoseconds  
- Redis (in-memory over network) → Microseconds  
- Database query (disk + parsing + network) → Milliseconds  

Now imagine:

If a DB query takes **10ms**  
And system receives **10,000 requests per second**

You would require:

100 seconds of DB time per second.

Impossible.

Database becomes the bottleneck.

Caching reduces repeated DB hits and improves:

- Latency  
- Throughput  
- Scalability  

---

# 2️⃣ What Caching Actually Solves

Caching solves:

> Repeated computation or repeated database reads.

Example (NeoAegis):

```
GET /api/sos/history
```

If the same user refreshes dashboard multiple times within 30 seconds:

Why query DB every time?

Instead:

- Store result in Redis
- Return cached data instantly
- Avoid DB pressure

---

# 3️⃣ Classifying Backend Data

Before caching, classify data correctly.

### A) Hot Data (Good for Caching)

- Frequently accessed
- Rarely updated

Examples:
- User profile
- SOS history summary
- Alert counts

---

### B) Highly Dynamic Data (Be Careful)

- Frequently updated
- Requires freshness

Examples:
- Real-time tracking location
- Payment transaction status

Caching dynamic data may cause stale responses.

---

# 4️⃣ Caching Layers in Production Systems

Caching does not exist only inside application.

There are 3 major layers:

---

## 1️⃣ Application-Level Cache (Redis)

Architecture:

```
API → Redis → DB
```

Used for:

- Per-user caching
- Authenticated endpoints
- Rate limiting counters
- Session storage

Redis is:

- In-memory
- Shared across cluster
- Extremely fast
- Supports TTL

---

## 2️⃣ Reverse Proxy Cache (Nginx)

Nginx can cache entire HTTP responses.

Flow:

```
Client → Nginx (cache hit) → Response
```

Node server is not even touched.

Used for:

- Public APIs
- Read-heavy endpoints
- Non-authenticated content

---

## 3️⃣ CDN Cache

Used for:

- Static assets
- Images
- Frontend bundles
- Public JSON responses

Reduces:

- Geographic latency
- Backend traffic load

---

# 5️⃣ Cache Aside Pattern (Most Common & Interview Favorite)

Also called:

> Lazy Loading Cache

Flow:

1. Client sends request.
2. API checks Redis.
3. If cache hit → return.
4. If cache miss:
   - Query DB
   - Store result in Redis (with TTL)
   - Return response

Example:

```
redis.get("user:123:sosHistory")
```

If exists → return.  
If not → fetch DB → cache → return.

Advantages:

- Simple
- Flexible
- Application controls caching logic

Disadvantage:

- Risk of stale data

---

# 6️⃣ Other Caching Patterns

---

## Write-Through

Whenever DB is updated:

- Update cache immediately.

Flow:

DB write → Cache updated simultaneously.

Pros:
- Cache always fresh

Cons:
- Increased write overhead
- More complexity

---

## Write-Behind (Write-Back)

Write to cache first.  
DB update happens asynchronously later.

Used in:

- Logging systems
- Analytics pipelines

Risk:

- Data loss if cache crashes before DB write.

---

# 7️⃣ Cache Invalidation (Hardest Problem)

Classic engineering quote:

> “There are only two hard things in computer science: cache invalidation and naming things.”

Why?

Because when data changes:

Cache must reflect that change.

Example:

```
POST /api/sos
```

Now cached:

```
GET /api/sos/history
```

is stale.

Two strategies:

---

## A) Invalidate Cache (Safer)

Delete cache key:

```
redis.del("user:123:sosHistory")
```

Next request rebuilds cache.

Simple and reliable.

---

## B) Update Cache (Optimized)

Manually modify cached data.

Faster but:

- Complex
- Error-prone

---

# 8️⃣ TTL (Time To Live)

Cache must not live forever.

Example:

```
TTL = 60 seconds
```

After expiry:

- Cache removed
- Fresh DB fetch on next request

Trade-off:

Short TTL → Fresh data, more DB load  
Long TTL → Less DB load, more stale risk  

Choosing TTL is a balance between:

- Freshness
- Performance

---

# 9️⃣ Cache Stampede Problem

Scenario:

Cache expires at 12:00.

At 12:00:

10,000 requests arrive.

All detect cache miss.

All hit DB simultaneously.

Database crashes.

This is called:

> Cache Stampede

Solutions:

- Add random expiry offsets
- Use distributed locking
- Allow only one request to rebuild cache
- Pre-warm cache before expiry

---

# 🔟 Why Not Use In-Memory JS Object For Cache?

Because in distributed systems:

- Each cluster worker has its own memory
- Each machine has isolated memory

Example:

Worker 1 cache ≠ Worker 2 cache  
Machine 1 cache ≠ Machine 2 cache  

This causes inconsistency.

Redis solves:

- Shared centralized cache
- Consistent data across all instances
- Supports TTL and atomic operations

---

# 🔥 NeoAegis Practical Caching Example

Good candidates:

```
GET /api/user/profile
GET /api/sos/history
GET /api/alerts
```

Bad candidates:

```
POST /api/sos
Real-time tracking location
Payment verification
```

---

# 🧠 Deep System Insight

Caching:

- Reduces DB load
- Improves latency
- Improves throughput
- Enables horizontal scaling

But introduces:

- Consistency challenges
- Invalidation complexity
- Stampede risk

Good backend engineers:

Understand both benefits and trade-offs.

---