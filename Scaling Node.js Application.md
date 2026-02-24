# 📘 Backend Deep Prep – Day 10  
Scaling Node.js Applications (Cluster + Horizontal Scaling + Load Balancer)

---

# 1️⃣ The Real Limitation of Node.js

Node.js runs JavaScript on:

👉 A single main thread

All of these run on the same thread:

- Express middleware
- Route handlers
- JWT verification
- bcrypt hashing
- JSON parsing
- Business logic

If CPU-heavy operations increase:

- Event loop gets blocked
- Incoming requests wait in queue
- Latency increases
- Throughput decreases

Example:

During NeoAegis login:

```js
bcrypt.compare(password)
```

bcrypt is CPU-bound.

Even async bcrypt uses:

👉 libuv thread pool

Default thread pool size:

```
4
```

So only 4 hashing operations run concurrently.

Remaining requests must wait.

---

# 2️⃣ Can Increasing Thread Pool Solve This?

You can increase thread pool size:

```bash
UV_THREADPOOL_SIZE=8
```

But:

- CPU cores remain same
- Tasks compete for same CPU
- Not true horizontal scaling

Thread pool is meant for async I/O support,  
not for application-level scaling.

---

# 3️⃣ Process-Level Scaling Using Cluster

Node provides:

👉 cluster module

Cluster allows:

- Creating multiple worker processes
- Each having its own event loop
- Each using a separate CPU core

Architecture becomes:

```
Primary Process
      ↓
Worker 1
Worker 2
Worker 3
Worker 4
```

Each worker:

- Is a separate Node instance
- Handles different requests

---

# 4️⃣ Production Issue With Cluster

Each worker has:

- Separate memory
- Separate heap
- Separate event loop

They DO NOT share variables.

Example:

```js
let activeSOSUsers = [];
```

User A triggers SOS.

Request handled by Worker 1:

```
Worker 1:
activeSOSUsers = [A]
```

Now another request:

```
GET /api/active-sos
```

handled by Worker 3.

Worker 3 memory:

```
activeSOSUsers = []
```

So response becomes inconsistent.

This causes:

- Wrong tracking data
- Session inconsistency
- Bad user experience

---

# 5️⃣ Solution — Shared External Store

Instead of in-memory state:

Use:

- Redis
- Database
- Distributed cache

Now:

All workers read/write from same store.

```
Worker 1 ─┐
Worker 2 ─┼──→ Redis (shared memory)
Worker 3 ─┤
Worker 4 ─┘
```

State becomes consistent across processes.

Important Rule:

Never store shared application state in memory when using cluster.

---

# 6️⃣ Cluster Solves Only Single Machine Limitation

Cluster scales:

👉 Within one machine

But traffic may exceed machine capacity.

So we add:

👉 More machines

This is called:

Horizontal Scaling

---

# 7️⃣ Multi-Machine Architecture

Now system becomes:

```
Machine 1 → Node cluster
Machine 2 → Node cluster
Machine 3 → Node cluster
Machine 4 → Node cluster
```

But new problem:

👉 How does client know which machine to send request to?

---

# 8️⃣ Role of Load Balancer

Load balancer sits in front of all backend servers.

Client sends request to:

```
api.neo.com
```

DNS resolves to:

👉 Load Balancer

Request flow becomes:

```
Client
   ↓
Load Balancer
   ↓
Machine 1 / 2 / 3 / 4
   ↓
Worker Process
```

Load balancer distributes incoming requests among machines.

---

# 9️⃣ Load Balancing Strategies

Common strategies:

Round Robin:

```
Req1 → Machine 1
Req2 → Machine 2
Req3 → Machine 3
Req4 → Machine 4
```

Least Connections:

Request sent to server with minimum active connections.

IP Hash:

Same client IP routed to same server.

---

# 🔟 Why In-Memory Session Fails Here?

User logs in via:

```
Machine 1
```

Session stored in memory of Machine 1.

Next request routed to:

```
Machine 3
```

Machine 3 does not have that session.

User appears logged out.

Hence:

Shared session store required.

---

# 1️⃣1️⃣ Nginx As Load Balancer

In production:

👉 Nginx is commonly used as reverse proxy + load balancer.

Architecture:

```
Client
   ↓
Nginx
   ↓
Multiple Node Servers
```

Nginx:

- Distributes traffic
- Terminates SSL (HTTPS)
- Protects backend from direct exposure
- Improves performance

---

# 🔁 Final Production Architecture

```
Client
   ↓
Nginx (Load Balancer)
   ↓
Machine 1 (Cluster)
Machine 2 (Cluster)
Machine 3 (Cluster)
Machine 4 (Cluster)
   ↓
Redis (Shared State)
```

---
