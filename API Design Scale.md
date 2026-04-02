# 📘 Backend Deep Prep – Day 17  
# API Design at Scale (Pagination, Filtering, Sorting – Deep Understanding)
---

# 1️⃣ The Core Problem: Large Data Responses

Consider NeoAegis:

```
GET /api/sos/history
```

If user has:

👉 50,000 SOS records  

And API returns all:

```
[ ...50,000 records... ]
```

Problems:

- High response size  
- Increased latency  
- High memory usage (server + client)  
- Slow frontend rendering  
- DB load increases  

---

# 🔥 Production Impact

Without proper API design:

- APIs become slow  
- System becomes unstable under load  
- Poor user experience  

---

# 2️⃣ Pagination (Core Solution)

Pagination means:

> Returning data in smaller chunks instead of full dataset

---

## Example

```
GET /api/sos?page=1&limit=10
```

Returns:

👉 Only 10 records  

---

# 3️⃣ Offset-Based Pagination

### Query

```js
db.sos.find().skip(1000).limit(10)
```

---

## How It Works

Database:

- Reads first 1000 records  
- Skips them  
- Returns next 10  

---

## Problem

Time complexity:

👉 O(n)

Large page number → slower query

---

## Issue

Page 1000:

```
skip(9990)
```

DB scans 9990 records unnecessarily.

---

# 4️⃣ Cursor-Based Pagination (Efficient Approach)

Instead of page numbers:

Use:

👉 Last seen value (cursor)

---

## First Request

```js
db.sos.find()
.sort({ createdAt: -1 })
.limit(3)
```

Result:

```
2024-01-10
2024-01-09
2024-01-08
```

Cursor:

```
2024-01-08
```

---

## Next Request

```js
db.sos.find({
  createdAt: { $lt: "2024-01-08" }
})
.sort({ createdAt: -1 })
.limit(3)
```

---

## Step-by-Step Execution

### 1️⃣ Filter

```js
createdAt < cursor
```

Removes already seen data.

---

### 2️⃣ Sort

```js
.sort({ createdAt: -1 })
```

Maintains consistent order.

---

### 3️⃣ Limit

```js
.limit(3)
```

Returns next chunk.

---

# 🧠 Mental Model

Cursor represents:

👉 Position in dataset  

Query means:

> “Fetch next records after this position”

---

# 🔁 Important Rule

If sorting is:

```js
createdAt: -1
```

Use:

```js
$lt (less than)
```

If sorting is:

```js
createdAt: 1
```

Use:

```js
$gt (greater than)
```

---

# 5️⃣ Why Cursor-Based Pagination Is Efficient

Offset pagination:

👉 Scan → Skip → Return  

Cursor pagination:

👉 Seek → Return  

Time complexity:

- Offset → O(n)  
- Cursor → O(log n)  

---

# 6️⃣ Sorting

Example:

```
GET /api/sos?sort=createdAt_desc
```

Query:

```js
.sort({ createdAt: -1 })
```

---

## Important Rule

Sorting must align with index.

Otherwise:

👉 MongoDB performs in-memory sort → slow

---

# 7️⃣ Filtering

Example:

```
GET /api/sos?status=active
```

Query:

```js
find({ status: "active" })
```

---

# 8️⃣ Combining Filtering + Sorting + Pagination

Example:

```
GET /api/sos?status=active&cursor=xyz&limit=10
```

Query:

```js
find({
  status: "active",
  createdAt: { $lt: cursor }
})
.sort({ createdAt: -1 })
.limit(10)
```

---

# 9️⃣ Index Strategy (Very Important)

Correct index:

```js
{ status: 1, createdAt: -1 }
```

Why:

- Filter → status  
- Sort → createdAt  

---

## ❌ Wrong Index

```js
{ createdAt: -1, status: 1 }
```

Because:

👉 Query starts with status  

Index must follow query pattern.

---

# 🔟 API Design Best Practices

---

## Limit Maximum Page Size

Never allow:

```
limit=10000
```

Use:

👉 max limit = 50 or 100  

---

## Return Metadata

```json
{
  "data": [...],
  "nextCursor": "abc123",
  "hasMore": true
}
```

---

## Avoid Deep Pagination

Offset pagination for large pages:

❌ Inefficient  

Cursor pagination:

✔ Efficient  

---

# 1️⃣1️⃣ Additional Advantages of Cursor Pagination

- Prevents duplicate records  
- Prevents missing records during concurrent inserts  
- Maintains consistent ordering  

---

# 🧠 Deep System Insight

Cursor pagination is essentially:

> A range query on an indexed field

---

# 🎯 Interview-Ready Summary

Cursor-based pagination is more efficient than offset-based pagination because it uses indexed range queries to directly locate the starting point, avoiding scanning and skipping large numbers of records, thereby improving performance and consistency at scale.
