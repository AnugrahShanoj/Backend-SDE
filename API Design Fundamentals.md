# 📘 Backend Deep Prep – Day 9  
API Design Fundamentals (REST Thinking for Backend Interviews)

---

# 1️⃣ API Is A Contract

Whenever frontend calls an endpoint like:

```
POST /api/sos
```

It creates a contract between:

- Frontend (client)
- Backend (server)

This contract defines:

- Request format
- Response structure
- Authentication requirement
- Status codes

Once this API is used in production:

👉 You cannot freely change it later

Because:

- Mobile apps may depend on it
- Web apps may depend on it
- Third-party integrations may depend on it

Breaking API contract leads to production failures.

So APIs must be designed carefully with future growth in mind.

---

# 2️⃣ Resource-Oriented Design (REST Thinking)

In REST:

👉 Design APIs around **resources (nouns)**  
Not actions (verbs)

Bad Design:

```
POST /triggerSOS
```

This represents an action.

Better Design:

```
POST /api/sos
```

Meaning:

👉 Create a new SOS resource

This makes API intuitive and scalable.

---

# 3️⃣ Correct Use of HTTP Methods

Each HTTP method represents a type of operation:

| Method | Meaning |
|--------|---------|
GET | Retrieve resource |
POST | Create new resource |
PUT | Replace entire resource |
PATCH | Partially update resource |
DELETE | Remove resource |

NeoAegis examples:

Trigger SOS:

```
POST /api/sos
```

Update SOS status:

```
PATCH /api/sos/:id
```

Fetch SOS history:

```
GET /api/sos
```

Delete SOS record:

```
DELETE /api/sos/:id
```

Using correct methods ensures clarity and prevents unintended behavior.

---

# 4️⃣ Idempotency (Important Interview Concept)

Idempotency means:

👉 Multiple identical requests result in the same final state on the server.

Example:

User clicks SOS button twice  
Network retries the same request

If backend creates 2 SOS events:

- Emergency contacts get duplicate alerts
- Multiple SMS notifications sent
- Multiple tracking sessions created
- Causes panic and resource misuse

So:

```
POST /api/sos
```

must be designed carefully to avoid duplication.

---

# 🔹 Achieving Idempotency

Client sends:

```
Idempotency-Key: abc123
```

Backend stores:

- Idempotency key
- Associated response

If same request arrives again:

- Do NOT create new SOS
- Return previously stored response

This prevents duplicate alerts during retries.

---

# 5️⃣ Idempotent HTTP Methods

Idempotent methods:

- PUT
- DELETE

Because:

Executing them multiple times results in the same final server state.

Example:

```
DELETE /api/users/123
```

First call deletes user.  
Second call does nothing.

Final state remains:

User does not exist.

So DELETE is idempotent.

---

# 6️⃣ API Versioning

Suppose current API expects:

```json
{
  "location": "12.23,45.67"
}
```

Later you update API to:

```json
{
  "latitude": 12.23,
  "longitude": 45.67
}
```

Older mobile apps calling old format may break.

Solution:

Introduce versioning:

```
/api/v1/sos
/api/v2/sos
```

Now:

- Old clients use v1
- New clients use v2

Backward compatibility is maintained.

---

# 7️⃣ Proper HTTP Status Codes

Avoid always returning:

```
200 OK
```

Use meaningful status codes:

| Code | Meaning |
|------|----------|
200 | Success |
201 | Resource created |
400 | Bad request |
401 | Unauthorized |
403 | Forbidden |
404 | Not found |
409 | Conflict |
500 | Server error |

Examples:

Invalid JWT:

```
401 Unauthorized
```

User banned from triggering SOS:

```
403 Forbidden
```

Duplicate SOS request detected:

```
409 Conflict
```

Proper status codes improve debugging and communication between systems.

---

# 8️⃣ Consistent Response Format

Avoid returning different response structures for different APIs.

Maintain consistency.

Success Response:

```json
{
  "data": {...},
  "message": "SOS triggered successfully"
}
```

Error Response:

```json
{
  "error": "User is inactive"
}
```

Consistency improves frontend integration and maintainability.

---

# 🧠 NeoAegis SOS API (Well Designed Example)

Endpoint:

```
POST /api/v1/sos
```

Request:

```json
{
  "latitude": 12.23,
  "longitude": 45.67
}
```

Response:

```json
{
  "data": {
    "sosId": "abc123"
  },
  "message": "SOS triggered successfully"
}
```

---
