# 📘 Backend Deep Prep – Day 5  
Async/Await Deep Mechanics & Event Loop Integration (Foundation Reinforcement)

---

# 1️⃣ What `await` Actually Does

When JavaScript sees:

```js
await somePromise;
```

It does three things:

1. Pauses the current async function at that line.
2. Attaches a hidden `.then()` continuation to the Promise.
3. Returns control to the caller immediately.

Important:

- It does NOT block the thread.
- It does NOT block the event loop.
- It pauses only the async function in which it appears.

Conceptual transformation:

```js
await promise;
```

≈

```js
promise.then(() => {
   // resume rest of function
});
```

---

# 2️⃣ What Gets Paused?

Only the execution of the current async function.

Not:

- The entire program
- The Node.js process
- Other request handlers
- The event loop

Example:

```js
async function test() {
   console.log("A");
   await new Promise(resolve => setTimeout(resolve, 2000));
   console.log("B");
}

test();
console.log("C");
```

Output:

```text
A
C
B
```

Explanation:

- `test()` pauses at `await`
- Control returns immediately
- `"C"` prints
- After 2 seconds, timer fires
- Promise resolves
- Continuation is scheduled as microtask
- `"B"` prints

---

# 3️⃣ Critical Clarification: When Is Continuation Scheduled?

The continuation after `await` is NOT scheduled immediately.

It is scheduled only when:

- The awaited Promise settles (resolves or rejects)

Timeline for timer-based Promise:

1. Timer callback executes (`resolve()`).
2. Promise state changes to fulfilled.
3. Continuation is queued into microtask queue.
4. After timer callback finishes, microtasks are drained.
5. Async function resumes.

---

# 4️⃣ Already Resolved Promise Behavior

Example:

```js
async function test() {
   console.log("A");
   await Promise.resolve();
   console.log("B");
}

test();
console.log("C");
```

Output:

```text
A
C
B
```

Even though the Promise is already fulfilled:

- `await` still pauses.
- Continuation is scheduled as a microtask.
- It does NOT continue synchronously.

Rule:

`await` always yields once.

---

# 5️⃣ `await` With Non-Promise Values

Example:

```js
await 5;
```

Internally becomes:

```js
await Promise.resolve(5);
```

Output example:

```js
async function test() {
   console.log("A");
   await 5;
   console.log("B");
}

test();
console.log("C");
```

Output:

```text
A
C
B
```

---

# 6️⃣ Multiple Awaits in Same Function

```js
async function test() {
   console.log("A");
   await Promise.resolve();
   console.log("B");
   await Promise.resolve();
   console.log("C");
}

test();
console.log("D");
```

Output:

```text
A
D
B
C
```

Important:

- First `await` schedules continuation.
- That continuation executes and schedules another microtask.
- Microtasks added during draining are also executed before leaving microtask phase.
- Microtask queue drains completely before moving to next event loop phase.

---

# 7️⃣ Microtasks vs Timers

Example:

```js
async function test() {
   console.log("A");
   await Promise.resolve();
   console.log("B");
}

setTimeout(() => console.log("T"), 0);

test();
console.log("D");
```

Output:

```text
A
D
B
T
```

Why?

Execution order:

1. Finish synchronous code.
2. Drain microtasks.
3. Enter timers phase.

Microtasks always run before timers.

---

# 8️⃣ Microtasks Drain After Every Execution Turn

Important upgrade in event loop understanding:

Microtasks are drained after:

- Initial script execution
- Each timer callback
- Each I/O callback
- Each setImmediate callback

Rule:

After every execution turn → drain microtasks completely.

---

# 9️⃣ Microtasks Interleave Between Timers

Example:

```js
setTimeout(() => {
   console.log("T1");
   Promise.resolve().then(() => console.log("P1"));
}, 0);

setTimeout(() => {
   console.log("T2");
}, 0);

console.log("Start");
```

Output:

```text
Start
T1
P1
T2
```

Explanation:

- T1 executes.
- P1 scheduled as microtask.
- Microtasks drained before moving to T2.
- Then T2 executes.

Timers do NOT batch before microtasks.

---

# 🔟 Combined Example

```js
async function test() {
   console.log("A");
   await Promise.resolve();
   console.log("B");
}

Promise.resolve().then(() => console.log("P1"));

test();

Promise.resolve().then(() => console.log("P2"));

console.log("D");
```

Output:

```text
A
D
P1
B
P2
```

Microtask queue order (FIFO):

1. P1
2. Async continuation (B)
3. P2

---

# 🧠 Final Mental Model

Execution hierarchy:

1. Execute current synchronous code (one execution turn)
2. Drain microtasks completely
3. Enter next event loop phase (timers, poll, check, etc.)
4. After each callback, drain microtasks again
5. Repeat

---

# 📌 Clean Comparison

| Feature | Pauses Function | Schedules Microtask | Blocks Thread |
|----------|----------------|--------------------|---------------|
| console.log | No | No | No |
| Promise.then | No | Yes | No |
| await | Yes (current async function only) | Yes | No |
| while(true) | Yes (everything) | No | Yes |

---
