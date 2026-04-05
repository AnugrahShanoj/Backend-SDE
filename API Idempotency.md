# 📘 Backend Deep Prep – Day 18  
# API Idempotency & Reliability (Deep System-Level Understanding)

# 1️⃣ The Core Problem: Duplicate Requests

In real-world systems, requests may be sent multiple times due to:

- Network failures  
- Client retries  
- Timeout + retry mechanisms  
- User double-clicking  

---

# 🔥 NeoAegis Scenario

```
POST /api/sos
```

User taps SOS:

- First request sent  
- Network delay → client retries  

Backend receives:

👉 Same request twice  

---

# ❌ Without Protection

System may:

- Create duplicate SOS entries  
- Send multiple SMS alerts  
- Trigger duplicate emergency workflows  

This leads to:

- Data inconsistency  
- Poor user experience  
- Operational issues  

---

# 2️⃣ What Is Idempotency?

An API is idempotent if:

> Multiple identical requests produce the same result as a single request.

---

## Example

### Idempotent

```
PUT /api/user/123
```

Calling multiple times:

👉 Same final state  

---

### Non-Idempotent

```
POST /api/sos
```

Each call:

👉 Creates a new resource  

---

# 3️⃣ Why POST Needs Protection

POST operations:

- Create new data  
- Trigger side effects  

Retries can lead to:

👉 Duplicate operations  

---

# 4️⃣ Solution: Idempotency Key

Client sends:

```
POST /api/sos
Idempotency-Key: abc123
```

---

## Backend Flow

1. Check if key exists  
2. If exists → return stored response  
3. If not:
   - Process request  
   - Store key + response  
   - Return response  

---

# 🧠 Key Concept

👉 Same idempotency key = same operation  

---

# 5️⃣ Where To Store Idempotency Data

Common choices:

- Redis (preferred)  
- Database  

Example:

```
Key: abc123
Value: {
   status: "success",
   sosId: "xyz"
}
```

---

# 6️⃣ TTL (Time To Live)

Idempotency keys should expire.

Example:

```
TTL = 5 minutes
```

---

## Why TTL Is Needed

- Prevent memory overflow  
- Keys only needed during retry window  

---

# 7️⃣ The Hidden Problem: Race Condition

Even with idempotency key, problem still exists.

---

## Scenario

Two identical requests arrive at same time:

```
Request A
Request B
```

---

### Step 1

Both check:

```
Key exists? → NO
```

---

### Step 2

Both proceed:

- A processes request  
- B also processes request  

---

### Step 3

Both store result:

👉 Duplicate execution occurs  

---

# 🚨 Root Cause

The flow:

```
Check → Process → Store
```

is **not atomic**

Between check and store:

👉 Another request can enter  

---

# 8️⃣ Solution: Atomic Locking (Redis)

Use:

```
SET key value NX
```

---

## What NX Means

👉 Set only if key does not exist  

---

## Correct Flow

### Request A

```
SET abc123 "processing" NX → SUCCESS
```

👉 Allowed to proceed  

---

### Request B

```
SET abc123 "processing" NX → FAIL
```

👉 Blocked or rejected  

---

## Result

- Only one request executes  
- No duplicate processing  

---

# 🧠 Key Insight

Idempotency requires:

👉 Atomic check + set  

Not:

❌ Separate check and set  
✔ Combined atomic operation  

---

# 9️⃣ Handling Processing State

While first request is running:

Key can be:

```
abc123 → "processing"
```

After completion:

```
abc123 → response
```

Second request can:

- Wait  
- Return cached response  

---

# 🔟 Idempotency vs Deduplication

---

## Idempotency

- Prevents duplicate execution  
- Happens before processing  

---

## Deduplication

- Removes duplicates after creation  
- Reactive approach  

---

# 🔥 NeoAegis Real Implementation

```
POST /api/sos
Idempotency-Key: user123-xyz
```

Flow:

- Check Redis  
- Acquire lock using SET NX  
- Create SOS record  
- Push SMS job to queue  
- Store result  
- Return response  

---

# 🧠 Deep System Insight

Idempotency ensures:

- Safe retries  
- Reliability  
- Consistent behavior  

In distributed systems:

👉 Retries are inevitable  

So idempotency is mandatory.

---

# 🎯 Interview-Ready Summary

Idempotency ensures that repeated requests with the same identifier produce the same result, but to prevent race conditions, the check and storage of idempotency keys must be atomic, typically implemented using mechanisms like Redis SET NX.

---