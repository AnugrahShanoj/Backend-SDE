# 📘 Backend Deep Prep – Day 8  
CSRF Attacks & Secure JWT Cookie Architecture (Conceptual Understanding)

## 🎯 Objective of Day 8

Understand clearly:

- Why storing JWT in HttpOnly cookies introduces CSRF risk
- What CSRF (Cross Site Request Forgery) attack actually is
- Why browser automatically sends authentication cookies
- How SameSite cookie attribute works
- When SameSite=None is required
- Why CSRF tokens are needed in cookie-based authentication
- How Same-Origin Policy protects CSRF tokens
- Why CSRF tokens fail when XSS exists

This topic is important whenever JWT is stored in cookies instead of Authorization header.

---

# 1️⃣ Problem Introduced By HttpOnly JWT Cookie

When JWT is stored in:

```
Authorization Header
```

Frontend manually attaches the token in every request.

But when JWT is stored in:

```
HttpOnly Cookie
```

Browser automatically sends the cookie with every request to matching domain.

Frontend JavaScript does NOT manually attach token.

This automatic cookie sending creates a new security risk:

👉 CSRF Attack

---

# 2️⃣ What Is CSRF?

CSRF stands for:

Cross Site Request Forgery

It occurs when an attacker tricks a logged-in user’s browser into sending a request to your backend without the user's knowledge.

Since cookies are automatically sent by the browser, backend assumes the request is legitimate.

---

# 🔬 Example Scenario (NeoAegis)

User logs into NeoAegis:

Browser stores:

```
Cookie: accessToken=validJWT
```

User then visits a malicious site:

```
evil.com
```

That site contains:

```html
<form action="https://api.neo.com/api/sos" method="POST">
</form>

<script>
document.forms[0].submit();
</script>
```

Now browser sends:

```
POST /api/sos
Cookie: accessToken=validJWT
```

Backend verifies JWT → valid  
SOS request gets accepted.

Even though user never intended to trigger SOS.

This is CSRF.

---

# 3️⃣ Why Browser Allows This?

Browser automatically sends cookies when:

- Request domain matches cookie domain

Browser does NOT verify:

- Who initiated the request
- Whether it came from your frontend
- Whether user intended it

So authentication cookie gets sent even from malicious site.

---

# 4️⃣ SameSite Cookie Attribute

SameSite attribute tells browser:

"When should this cookie be sent?"

---

## SameSite=Strict

Cookie sent only when:

- Request originates from same site

Cross-site requests from:

```
evil.com → api.neo.com
```

Will NOT include cookie.

CSRF attack fails.

---

## SameSite=Lax

Cookie sent for:

- Top-level navigation (GET requests)

But not sent for:

- POST requests from external site

So CSRF POST attack still fails.

---

## SameSite=None

Cookie sent for:

- All cross-site requests

Required when:

Frontend and backend are deployed on different domains.

Must be used with:

```
Secure
```

Otherwise browser blocks cookie.

---

# 5️⃣ NeoAegis Deployment Scenario

If:

```
Frontend → app.neo.com
Backend → api.neo.com
```

This becomes cross-site communication.

To allow cookie transmission:

```
SameSite=None;
Secure;
```

must be set.

But now CSRF risk comes back.

So additional protection is required.

---

# 6️⃣ CSRF Token Protection

Backend generates:

```
csrfToken = abc123
```

Backend sends:

- JWT → HttpOnly cookie
- CSRF Token → readable by frontend

Frontend stores CSRF token in memory.

Now every protected request includes:

```
Cookie: accessToken=validJWT
X-CSRF-Token: abc123
```

Backend verifies:

- JWT from cookie
- CSRF token from header

Only if both are valid → request allowed.

---

# 7️⃣ Why Attacker Cannot Use CSRF Token?

Attacker’s JavaScript runs on:

```
evil.com
```

CSRF token belongs to:

```
app.neo.com
```

Due to Same-Origin Policy:

JavaScript from one origin cannot:

- Read response from another origin
- Access cookies of another origin
- Access localStorage of another origin

So attacker cannot:

```
fetch("https://api.neo.com/getCsrfToken")
```

and read the response.

Request may be sent,
but response cannot be accessed by attacker’s JS.

Hence attacker cannot include:

```
X-CSRF-Token
```

in forged request.

Attack fails.

---

# 8️⃣ Important Limitation

CSRF protection assumes:

👉 No XSS vulnerability exists

If attacker executes JavaScript inside:

```
app.neo.com
```

Then:

- CSRF token can be read
- Authenticated request can be made
- Backend will accept request

So:

CSRF token protects against CSRF  
But NOT against XSS

---

# 🧠 Clean Mental Model

| Threat | Protection |
|---------|-------------|
CSRF attack | CSRF Token |
JWT theft via JS | HttpOnly Cookie |
Cross-site token read | Same-Origin Policy |
XSS attack | CSP, sanitization |

---