# 📘 Backend Deep Prep – Day 4
Error Handling in Node.js (Sync vs Async, Promise Rejections, Async/Await Error Flow)


## 1️⃣ How Errors Work in Synchronous Code

Example:

```js
function test() {
   throw new Error("Something failed");
}

try {
   test();
} catch (err) {
   console.log("Caught:", err.message);
}
```

Execution Flow:

1. test() runs.
2. Error is thrown.
3. JavaScript looks up the current call stack.
4. It finds the nearest try/catch block.
5. The error is caught.

Key Concept:

Errors thrown synchronously travel up the same call stack until a matching catch block is found.

---

## 2️⃣ Why try/catch Fails With setTimeout

Example:

```js
try {
   setTimeout(() => {
      throw new Error("Boom");
   }, 0);
} catch (err) {
   console.log("Caught:", err.message);
}
```

Execution Flow:

1. The try block executes.
2. setTimeout registers the callback.
3. No error occurs yet.
4. The try block finishes execution.
5. Later, the event loop runs the callback.
6. The error is thrown inside a new call stack.
7. The original try/catch block no longer exists.

Key Concept:

try/catch only works within the same synchronous call stack.  
setTimeout executes in a future event loop phase, so the original try/catch cannot catch it.

---

## 3️⃣ How Promises Handle Errors

Example:

```js
Promise.reject("Fail")
   .catch(err => console.log("Caught:", err));
```

How It Works:

1. Promise state becomes rejected.
2. The rejection is stored internally.
3. .catch() registers a rejection handler.
4. That handler runs in the microtask queue.

Promises capture errors internally and propagate them through the Promise chain.

---

## 4️⃣ async Functions and Errors

Example:

```js
async function test() {
   throw new Error("Fail");
}

test();
```

Important:

An async function always returns a Promise.

This is equivalent to:

```js
function test() {
   return Promise.reject(new Error("Fail"));
}
```

Throwing inside async becomes a rejected Promise.

---

## 5️⃣ How await Handles Rejection

Example:

```js
try {
   await Promise.reject("Fail");
} catch (err) {
   console.log("Caught");
}
```

Execution Flow:

1. Promise.reject creates a rejected Promise.
2. await waits for it.
3. Since it rejects, await throws the rejection value.
4. The catch block handles it.

Key Concept:

await converts a rejected Promise into a thrown error inside the current execution context.

---

## 6️⃣ Control Flow After await Throws

Example:

```js
try {
   await Promise.reject("Fail");
   console.log("After");
} catch (err) {
   console.log("Caught");
}
```

Output:

```text
Caught
```

Explanation:

When await throws, execution jumps immediately to the catch block.  
The remaining lines inside the try block are skipped.

---

## 7️⃣ Execution Continues After catch

Example:

```js
try {
   await Promise.reject("Fail");
} catch (err) {
   console.log("Caught");
}

console.log("After");
```

Output:

```text
Caught
After
```

Explanation:

Once the catch block finishes, normal execution continues outside the try/catch block.

---

## 8️⃣ The Dangerous Mistake — Forgetting await

Example:

```js
async function test() {
   throw new Error("Fail");
}

try {
   test(); // no await
} catch (err) {
   console.log("Caught");
}
```

This does NOT catch the error.

Why?

- test() returns a Promise.
- The rejection happens asynchronously.
- try/catch only handles synchronous errors.
- No .catch() is attached.
- This becomes an unhandled rejection.

---

## 9️⃣ Synchronous vs Asynchronous Error Model

Synchronous Error Flow:

```text
throw → travels up call stack → caught by try/catch
```

Asynchronous Error Flow:

```text
Promise rejects → must be handled via .catch or await inside try/catch
```

If not handled:

- It becomes an unhandled promise rejection.

---

## 🔟 Rethrowing Inside catch

Example:

```js
try {
   await Promise.reject("Fail");
} catch (err) {
   console.log("Caught");
   throw new Error("Another Fail");
}

console.log("After");
```

What Happens:

- First rejection is caught.
- New error is thrown.
- If not caught elsewhere, it propagates further.
- "After" will NOT execute.

Key Concept:

If you rethrow inside catch, execution stops again unless handled by another try/catch.

---

## 🧠 Mental Model Summary

- try/catch works only in the same call stack.
- setTimeout errors are not caught by outer try/catch.
- Promise rejections must be handled via .catch or await.
- async functions always return Promises.
- await converts rejection into thrown error.
- Code after a failing await inside try does not execute.
- After catch finishes, execution continues normally.
- Forgetting await leads to unhandled rejection.
- Rethrowing inside catch stops execution again.

---

## 📌 Key Interview Traps

❌ “try/catch can catch setTimeout errors”  
❌ “async functions throw synchronously”  
❌ “await blocks the thread”  
❌ “Forgetting await is harmless”

---
