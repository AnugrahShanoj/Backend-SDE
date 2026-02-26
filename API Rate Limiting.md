# 📘 Backend Deep Prep – Day 12  
Rate Limiting (API Protection & System Stability)

---

# 1️⃣ Why Rate Limiting Is Required?

Suppose NeoAegis exposes:

```
POST /api/sos
```

Now imagine an attacker sends:

👉 10,000 requests per second

Without rate limiting:

- Node processes become overloaded
- Worker threads get saturated
- Redis queue fills up
- SMS provider gets flooded
- Emergency contacts receive spam alerts
- System may crash

Even worse:

External providers like SMS gateway may:

- Block your account
- Increase billing cost

So system must protect itself from:

- Abuse
- Bot traffic
- Accidental overload
- DoS attacks

---

# 2️⃣ What Rate Limiting Does?

Backend enforces a limit like:

```
Max 5 SOS requests per minute per user
```

Each time a request arrives:

Backend checks:

- Who is making request? (UserID or IP)
- How many requests already made?
- Is the limit exceeded?

If within limit:

✔ Allow request

If exceeded:

❌ Reject request

Return:

```
429 Too Many Requests
```

---

# 3️⃣ Where Should Request Count Be Stored?

Never store in:

```js
let requestCount = {};
```

Because:

- Clustered apps have multiple worker processes
- Multi-machine systems have isolated memory

Each process would track count separately.

This leads to inconsistent rate limiting.

---

# 4️⃣ Using Redis For Distributed Rate Limiting

Redis acts as a shared counter store.

All API instances:

- Read from same Redis store
- Update same counter

Now:

Rate limit enforcement becomes consistent across:

- Clustered processes
- Multiple servers

Example:

Limit:

```
3 SOS requests per minute per user
```

User sends requests across:

Machine 1  
Machine 2  
Machine 3  

Redis ensures total request count is tracked globally.

After 3:

Remaining requests rejected.

---

# 5️⃣ Rate Limiting Strategies

## Fixed Window

Limit requests per time window:

```
5 requests per minute
```

Window resets every minute.

Problem:

User may send:

- 5 requests at 12:00:59
- 5 more at 12:01:00

Allows burst traffic.

---

## Sliding Window (Better)

Track requests over:

Last 60 seconds continuously.

Prevents sudden burst.

Provides smoother limiting.

---

## Token Bucket

User receives:

```
5 tokens
```

Each request consumes 1 token.

Tokens refill gradually.

Allows:

- Controlled bursts
- Stable request flow

Widely used in production systems.

---

# 6️⃣ Per-IP vs Per-User Limiting

Per-IP:

- Protects from bot traffic
- Useful for unauthenticated endpoints

Per-User:

- Protects logical abuse
- Useful for authenticated endpoints

Production systems combine both.

---

# 7️⃣ Where To Apply Rate Limiting?

Can be applied at:

- API Gateway
- Reverse Proxy (Nginx)
- Backend Application Layer

Early rejection improves performance.

---

# 🔥 NeoAegis Mapping

Limit:

```
POST /api/sos
```

to:

```
3 requests per minute per user
```

Prevents:

- SOS spam
- SMS flooding
- Queue overload

---