# 📘 Backend Deep Prep – Day 7  
JWT Authentication Flow & Middleware Architecture (Conceptual + System Thinking)


# 1️⃣ Why JWT Is Needed?

Traditional authentication (session-based) works like this:

```
User logs in → Server creates session → Stores session in DB
```

For every future request:

```
Client sends session ID → Server checks DB → Allows request
```

This causes problems:

- DB lookup on every request
- Increased latency
- Hard to scale across multiple servers
- Requires sticky sessions
- Stateful architecture

For a system like **NeoAegis**, where SOS calls must be processed quickly:

If every API call needs DB lookup for authentication →  
latency increases → rescue response may get delayed.

---

# 2️⃣ How JWT Changes The Model

With JWT:

```
User logs in → Server generates JWT → Sends it to client
```

JWT contains user identity info like:

```json
{
  "userId": "123",
  "role": "user",
  "exp": "expiry time"
}
```

And is signed using server secret.

Now for every request:

```
Server verifies token signature
```

No DB lookup required for authentication.

This is called:

👉 Stateless Authentication

Server does not store session.

---

# 3️⃣ Important: JWT Is Just A Credential

JWT itself does NOT decide:

- Where it is stored
- How it is sent

JWT can be transported using:

- Authorization Header
- Cookie
- Request Body (not recommended)

Transport method is a design decision.

---

# 4️⃣ Two Common JWT Transport Models

---

## 🔹 Model 1 — Authorization Header (Bearer Token)

Login Response:

```json
{
   "accessToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

Frontend stores token in:

- localStorage
- memory
- application state

For every request, frontend manually attaches:

```
Authorization: Bearer <token>
```

Example:

```js
fetch("/api/profile", {
   headers: {
      Authorization: `Bearer ${token}`
   }
});
```

Backend extracts token using:

```js
req.headers.authorization
```

---

## 🔹 Model 2 — HTTP-Only Cookie (Secure Model)

Login Response:

Server sends:

```
Set-Cookie:
accessToken=eyJhbGciOiJIUzI1NiIs...
HttpOnly;
Secure;
SameSite=Strict
```

Browser automatically:

- Stores cookie
- Sends cookie in every request to same domain

Request becomes:

```
Cookie: accessToken=eyJhbGciOiJIUzI1NiIs...
```

Frontend JavaScript:

❌ Cannot read token  
❌ Cannot manually attach token  

Because:

👉 HttpOnly cookies are not accessible to JS

Backend extracts token using:

```js
req.cookies.accessToken
```

(using `cookie-parser`)

---

# 5️⃣ Header vs Cookie JWT Flow

| Storage Method | Token Sent How? | Backend Extracts From |
|----------------|-----------------|----------------------|
| localStorage   | Authorization header | req.headers.authorization |
| HttpOnly Cookie| Cookie header   | req.cookies.accessToken |

Bearer token is NOT used when JWT is stored in HttpOnly cookie.

Browser handles cookie transport automatically.

---

# 6️⃣ Why Use Middleware For Authentication?

Instead of verifying JWT inside every controller:

```js
jwt.verify(token)
```

Use:

```js
app.get("/profile", authMiddleware, handler);
```

Middleware will:

1. Extract token
2. Verify signature
3. Decode payload
4. Attach user info to `req.user`
5. Call `next()`

Controller can now focus only on business logic.

This ensures:

- Separation of concerns
- Reusability
- Maintainability
- Cleaner code

---

# 7️⃣ Should We Trust JWT Payload Completely?

JWT is signed → cannot be modified  
But JWT may become **stale**

Example:

- User banned after login
- Account disabled
- SOS permission revoked

Token may still be valid for 10 minutes.

Blindly trusting JWT means:

- Banned users can still access APIs

JWT should be trusted for:

👉 Identity (Who the user is)

But not always for:

👉 Current authorization state

---

# 8️⃣ Recommended Production Flow

For sensitive endpoints like:

```
POST /api/sos
```

Flow should be:

```
Verify JWT signature
↓
Extract userId
↓
Check lightweight DB / Redis:
   - isActive?
   - isBanned?
↓
Proceed if allowed
```

Use Redis instead of main DB for:

- Fast status checks (~1ms)
- Reduced latency
- Better scalability

---

# 9️⃣ Where Should JWT Verification Happen?

Best practice:

👉 At both API Gateway and Backend Service

Gateway verification:

- Reject invalid tokens early
- Reduce unnecessary load on services

Service-level verification:

- Prevent internal bypass
- Maintain zero-trust architecture

Never assume internal network is fully trusted.

---

# 🧠 Clean Mental Model

JWT:

- Used for authentication (identity)

Database / Cache:

- Used for authorization (permissions, status)

Cookie / Header:

- Used for transport

---

# 🔑 NeoAegis Recommendation (Web App)

Use:

👉 HttpOnly Cookie for Access Token

Because:

- Protects against XSS attacks
- Token not accessible to JS
- Browser handles sending automatically

---