# 📘 Backend Deep Prep – Day 2
Event Loop Internal Mechanics (Phases, Execution Order, Microtasks)

## 🎯 Objective of Day 2

Understand:

- What the event loop actually is
- All Node.js event loop phases
- Execution order between setTimeout, setImmediate, Promises, and process.nextTick
- Microtask vs Macrotask queues
- Starvation scenarios
- Execution order inside and outside I/O callbacks

This is internal runtime behavior knowledge required for backend interviews.

## 1️⃣ What Is The Event Loop?

The event loop is a continuous mechanism that:

- Checks if the call stack is empty
- Picks callbacks from specific phase queues
- Pushes ready callbacks to the call stack
- Executes them in a strict order

Important points:

- Event loop does NOT run in parallel
- It only operates when call stack is empty
- It controls execution order of async callbacks

## 2️⃣ Event Loop Basic Structure

Conceptually:

```text
while (true) {
   1. If call stack empty
   2. Execute callbacks in current phase
   3. Drain microtask queues
   4. Move to next phase
}
```

Event loop runs in phases.

## 3️⃣ Node.js Event Loop Phases

Node has the following major phases:

1. timers
2. pending callbacks
3. idle / prepare (internal)
4. poll
5. check
6. close callbacks

Each phase has its own callback queue.

The event loop moves in this order.

## 4️⃣ Timers Phase

Handles:

- setTimeout()
- setInterval()

Example:

```js
setTimeout(() => {
   console.log("Hello");
}, 0);
```

Timer delay is a minimum threshold.
It does NOT guarantee exact timing.

## 5️⃣ Pending Callbacks Phase

Handles:

- Certain system-level callbacks
- Some TCP errors
- Deferred I/O callbacks

## 6️⃣ Poll Phase

Poll phase:

- Retrieves new I/O events
- Executes I/O callbacks (fs, network, etc.)
- Can block waiting for I/O if nothing else scheduled

Most async operations resolve here.

Example:

```js
fs.readFile("file.txt", () => {
   console.log("done");
});
```

The callback runs in poll phase.

## 7️⃣ Check Phase

Handles:

- setImmediate()

Example:

```js
setImmediate(() => {
   console.log("immediate");
});
```

Runs in check phase.

## 8️⃣ Close Callbacks Phase

Handles:

- socket.on("close")
- stream close events

## 9️⃣ Microtasks vs Macrotasks

Node has:

1. Macrotask queues (per phase)
2. Microtask queues

Microtasks include:

- Promise .then()
- queueMicrotask()
- process.nextTick (special queue)

Important rule:

After every macrotask execution,
Node drains microtasks completely before moving forward.

## 🔟 process.nextTick vs Promise

Priority order after call stack empties:

1. process.nextTick queue
2. Promise microtask queue
3. Continue event loop phases

Example:

```js
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));
```

Output:

```text
nextTick
promise
```

process.nextTick always runs first.

## 1️⃣1️⃣ Execution Order (Top-Level Context)

```js
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));
console.log("sync");
```

Correct order:

```text
sync
nextTick
promise
timeout
immediate
```

Reason:

- Sync runs first
- nextTick
- Promise
- timers phase
- check phase

## 1️⃣2️⃣ Execution Order Inside I/O Callback

```js
const fs = require("fs");

fs.readFile(__filename, () => {
   setTimeout(() => console.log("timeout"), 0);
   setImmediate(() => console.log("immediate"));
});
```

Correct order:

```text
immediate
timeout
```

Reason:

- I/O callback runs in poll phase
- After poll → check phase
- setImmediate runs before timers

## 1️⃣3️⃣ Starvation Scenario – process.nextTick

```js
function loop() {
   process.nextTick(loop);
}
loop();
```

Result:

- nextTick queue never empties
- Event loop never moves to timers or poll
- I/O starvation occurs

Reason:

Node drains nextTick queue completely before moving to next phase.

## 1️⃣4️⃣ Why setTimeout Recursion Is Less Dangerous

```js
function loop() {
   setTimeout(loop, 0);
}
loop();
```

Reason:

- setTimeout goes to timers phase
- Event loop continues cycling
- Other phases still get chance to execute
- No complete starvation

## 🧠 Mental Model Summary

Event loop:

- Moves phase by phase
- Executes macrotasks
- After each macrotask → drains microtasks
- process.nextTick has highest priority
- Promise microtasks run after nextTick
- Inside I/O callback:
  - check phase runs before timers
- Infinite nextTick recursion causes starvation

## 📌 Key Interview Traps

❌ “setImmediate always runs before setTimeout(0)”  
❌ “Promises run after timers”  
❌ “process.nextTick is same as Promise microtask”  
❌ “Event loop runs in parallel”
