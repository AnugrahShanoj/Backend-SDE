# 📘 Backend Deep Prep – Day 6  
Express Request Lifecycle & Middleware Flow (Deep Backend View)

## 🎯 Objective of Day 6

Understand deeply:

- How a request travels from socket → Express → middleware → response
- How Express middleware chaining actually works internally
- What `next()` really does
- Difference between `throw` vs `next(err)`
- How async affects middleware flow
- Why some requests hang
- Why some async errors crash the process
- How Express error handling truly works (v4 behavior)

This connects Event Loop + Async knowledge with real backend architecture.

---

# 1️⃣ What Happens When A Request Hits Express?

When a client sends an HTTP request:

1. OS receives TCP packet.
2. Node’s HTTP server (built on `http` module) receives data.
3. libuv detects readable socket in **poll phase**.
4. Node parses HTTP headers.
5. Node creates:
   - `req` (IncomingMessage)
   - `res` (ServerResponse)
6. Node calls the request listener.

In Express:

```js
const app = express();
http.createServer(app);
```

This means:

👉 `app` itself is the request handler function.

---

# 2️⃣ Express Is Just A Middleware Dispatcher

Internally Express maintains a stack:

```text
[
  middleware1,
  middleware2,
  routeHandler,
  errorMiddleware
]
```

Each middleware is stored in registration order.

Order matters.

---

# 3️⃣ How Middleware Execution Works

Simplified internal idea:

```js
let index = 0;

function next(err) {
   const layer = stack[index++];
   layer(req, res, next);
}
```

Middleware execution is:

- Sequential
- Pointer-based
- Manual chain traversal

Nothing magical.

---

# 4️⃣ What `next()` Actually Means

`next()` means:

👉 Move to the next middleware in the stack.

It does NOT:

- Create a new thread
- Enter event loop
- Defer execution

It simply advances the middleware pointer.

---

# 5️⃣ Middleware Flow Example

```js
app.use((req, res, next) => {
   console.log("A");
   next();
});

app.use((req, res, next) => {
   console.log("B");
   next();
});

app.get("/test", (req, res) => {
   console.log("C");
   res.send("done");
});
```

Request to `/test`:

Output:

```text
A
B
C
```

Because each middleware calls `next()`.

---

# 6️⃣ Async Middleware Behavior

```js
app.use(async (req, res, next) => {
   console.log("A");
   await Promise.resolve();
   console.log("B");
   next();
});
```

Execution:

1. `"A"` prints.
2. `await` pauses this middleware.
3. Express does NOT move forward yet.
4. After Promise resolves, continuation runs.
5. `"B"` prints.
6. `next()` moves to next middleware.

Important:

👉 Express does NOT auto-continue after `await`.
👉 Only `next()` or `res.send()` advances the chain.

---

# 7️⃣ Hanging Request Scenario

```js
app.use((req, res, next) => {
   console.log("A");
});
```

No `next()`.  
No `res.send()`.

Result:

- Middleware chain stops.
- Response never sent.
- Client waits.
- Request eventually times out.

This is called a hanging request.

---

# 8️⃣ Express Error Handling – Core Rule

Express automatically catches:

👉 Synchronous errors thrown inside middleware

Example:

```js
app.use((req, res, next) => {
   throw new Error("Boom");
});
```

Express internally wraps middleware in try/catch:

```js
try {
   middleware(req, res, next);
} catch (err) {
   next(err);
}
```

So error middleware runs.

---

# 9️⃣ Error Middleware

Error middleware must have 4 parameters:

```js
app.use((err, req, res, next) => {
   res.status(500).send("Error");
});
```

Express identifies error middleware by function length = 4.

If `next(err)` is called:

- Express skips normal middleware
- Directly jumps to error middleware

---

# 🔟 Difference: `throw` vs `next(err)`

## Case A – Synchronous throw (Works)

```js
throw new Error("Boom");
```

✔ Express catches  
✔ Error middleware runs  

---

## Case B – Throw inside setTimeout (Fails)

```js
setTimeout(() => {
   throw new Error("Boom");
}, 0);
```

❌ Happens in new call stack  
❌ Express try/catch not active  
❌ Becomes uncaught exception  
❌ May crash process  

---

## Case C – `next(err)` inside setTimeout (Works)

```js
setTimeout(() => {
   next(new Error("Boom"));
}, 0);
```

✔ Explicitly signals Express  
✔ Error middleware runs  
✔ Safe pattern  

---

# 1️⃣1️⃣ Async Middleware (Express v4 Danger)

```js
app.use(async (req, res, next) => {
   throw new Error("Boom");
});
```

Inside async:

```js
throw → Promise.reject(...)
```

Express v4:

- Does NOT auto-handle rejected Promises
- Does NOT attach `.catch(next)`
- Results in unhandled rejection
- Request may hang
- Process may crash

This is a common production bug.

---

# 1️⃣2️⃣ Correct Async Error Handling Pattern

Wrap async middleware:

```js
const asyncHandler = fn =>
   (req, res, next) =>
      Promise.resolve(fn(req, res, next)).catch(next);
```

Usage:

```js
app.use(asyncHandler(async (req, res) => {
   throw new Error("Boom");
}));
```

Now error middleware runs safely.

---

# 1️⃣3️⃣ Express v4 vs v5

Express v4:
- Does NOT automatically catch async rejections.

Express v5 (beta):
- Automatically handles async rejections.

Most production systems still use v4.

---

# 1️⃣4️⃣ Dangerous Scenario

```js
app.use((req, res, next) => {
   setTimeout(() => {
      throw new Error("Boom");
   }, 0);
   next();
});
```

Execution:

1. Response may already be sent.
2. Later async throw happens.
3. Node sees uncaught exception.
4. Process may crash.

Even after response was successful.

Express lifecycle ≠ Node process lifecycle.

---

# 🧠 Core Distinction

`throw` → JavaScript-level error propagation (depends on call stack)

`next(err)` → Express-level error propagation (explicit framework signaling)

---

# 📌 Production Rules

- Always call `next()` or `res.send()`.
- Never leave middleware without advancing chain.
- Never throw inside async callbacks.
- Always wrap async middleware in `.catch(next)`.
- Use centralized error middleware.
- Never rely on Express to catch Promise rejections (v4).

---
