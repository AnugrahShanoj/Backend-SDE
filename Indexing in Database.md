# 📘 Backend Deep Prep – Day 15  
# Database Indexing (Deep Performance & Query Optimization)

# 1️⃣ The Core Problem: Why Queries Become Slow

Consider a query in NeoAegis:

```js
Users.find({ email: "abc@gmail.com" })
```

If your collection has:

👉 1 million users

Without index, MongoDB must:

```
Scan every document one by one
```

This is called:

> **Collection Scan (COLLSCAN)**

Time complexity:

👉 O(n)

Meaning:

- 100 records → fast  
- 1 million → slow  
- 10 million → very slow  

---

# 🔥 Production Impact

Without indexing:

- Login becomes slow  
- Search APIs lag  
- Dashboard loads slowly  
- DB CPU usage increases  
- Entire system performance degrades  

Database becomes the bottleneck.

---

# 2️⃣ What Is Indexing?

Index is:

> A data structure that allows fast lookup of records without scanning entire collection.

Analogy:

- Without index → read full book  
- With index → jump directly to page  

---

# 3️⃣ How Index Works Internally (Conceptual)

MongoDB uses:

👉 **B-Tree (Balanced Tree)**

Simplified structure:

```
          [M]
        /     \
     [A-H]   [N-Z]
```

Searching:

```js
email = "abc@gmail.com"
```

Instead of scanning all records:

- Traverse tree  
- Reach correct node  
- Fetch result  

Time complexity:

👉 O(log n)

Huge performance improvement.

---

# 4️⃣ Without Index vs With Index

Without index:

```
Scan 1 million records → slow
```

With index:

```js
db.users.createIndex({ email: 1 })
```

```
Direct lookup → fast
```

---

# 5️⃣ Types of Indexes (MongoDB)

---

## 1️⃣ Single Field Index

```js
{ email: 1 }
```

Used for:

```js
find({ email })
```

---

## 2️⃣ Compound Index (Very Important)

```js
{ userId: 1, createdAt: -1 }
```

Used for:

```js
find({ userId }).sort({ createdAt: -1 })
```

---

## ⚠️ Key Rule (Most Important)

👉 Compound index works **left → right**

For index:

```js
{ A, B, C }
```

Valid usage:

✔ A  
✔ A + B  
✔ A + B + C  

Invalid:

❌ B alone  
❌ C alone  
❌ B + C  

---

## 3️⃣ Unique Index

```js
{ email: 1 }, { unique: true }
```

Ensures:

- No duplicate values

Used for:

- Email
- Username

---

## 4️⃣ Text Index

```js
{ name: "text" }
```

Used for:

- Search functionality

---

# 6️⃣ Query Pattern Matters (Interview Focus)

Indexes must match:

👉 How you query data

Example:

```js
find({ userId }).sort({ createdAt: -1 })
```

Correct index:

```js
{ userId: 1, createdAt: -1 }
```

Wrong index:

```js
{ createdAt: -1, userId: 1 }
```

Because:

MongoDB uses index from left to right.

---

# 7️⃣ Why Order Matters (Deep Insight)

Index:

```js
{ userId: 1, createdAt: -1 }
```

Structure:

```
userId = A
   createdAt sorted

userId = B
   createdAt sorted
```

If query starts with `createdAt`:

MongoDB cannot jump into middle of index.

It must scan.

---

# 8️⃣ Real NeoAegis Indexing Strategy

---

## Login

```js
find({ email })
```

Index:

```js
{ email: 1 }
```

---

## SOS History

```js
find({ userId }).sort({ createdAt: -1 })
```

Index:

```js
{ userId: 1, createdAt: -1 }
```

---

## Active Users

```js
find({ isActive: true })
```

Index:

```js
{ isActive: 1 }
```

---

# 9️⃣ Index Trade-Offs (Critical)

Indexes are NOT free.

---

## Advantages

- Faster reads  
- Faster search  
- Efficient sorting  

---

## Disadvantages

- Slower writes (index must update)  
- Increased memory usage  
- Increased storage  
- Too many indexes degrade performance  

---

# 🔥 Key Insight

> Indexes improve READ performance but degrade WRITE performance.

---

# 🔟 Over-Indexing Problem

If you create too many indexes:

- Insert becomes slow  
- Update becomes slow  
- Memory consumption increases  

Best practice:

👉 Only index fields used in queries

---

# 1️⃣1️⃣ When Index Is NOT Used

Index may not be used when:

- Query skips first field in compound index  
- Query uses wrong field order  
- Low selectivity fields (e.g., boolean)  
- Query applies functions on field  

---

# 1️⃣2️⃣ Deep Mental Model

Without index:

👉 Scan everything  

With index:

👉 Navigate directly  

---

# 🎯 Interview-Ready Summary

Indexing improves query performance by allowing the database to locate data using a structured lookup instead of scanning the entire collection, but it must be designed based on query patterns and comes with trade-offs in write performance and memory usage.

---
