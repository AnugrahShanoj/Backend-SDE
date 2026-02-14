
Async Model in Node.js (Promises, async/await, Internal Flow, Error Handling, Concurrency)


## 1️⃣ The Illusion of “Async”

When writing:

```js
const data = await fetchData();
```

It looks like:

- Execution is paused
- Code is blocked

Reality:

- JavaScript never blocks
- await pauses only the current async function
- The event loop continues running
- Other requests and callbacks are processed

await provides cooperative concurrency, not thread blocking.

## 2️⃣ What Is a Promise Internally?

A Promise is:

- A JavaScript object
- With internal states:
  - pending
  - fulfilled
  - rejected
- With internal callback queues

Example:

```js
new Promise((resolve, reject) => {
   console.log("inside");
});
console.log("outside");
```

Output:

```text
inside
outside
```

Important:

The Promise executor runs synchronously.
Creating a Promise is not asynchronous.

## 3️⃣ How Promise Resolution Works

When a Promise resolves:

1. Its state changes to fulfilled
2. .then() callbacks are queued
3. They go into the microtask queue
4. They execute after the current call stack clears

Example:

```js
Promise.resolve().then(() => console.log("A"));
console.log("B");
```

Output:

```text
B
A
```

Because .then runs as a microtask.

## 4️⃣ async Is Just Syntactic Sugar

This:

```js
async function test() {
   return 5;
}
```

Is equivalent to:

```js
function test() {
   return Promise.resolve(5);
}
```

An async function always returns a Promise.

## 5️⃣ What Happens During await

Example:

```js
async function run() {
   console.log("1");
   await Promise.resolve();
   console.log("2");
}

run();
console.log("3");
```

Execution:

- "1" prints
- await pauses function
- Function returns a Promise
- "3" prints
- Microtask resumes function
- "2" prints

Output:

```text
1
3
2
```

Important:

await splits the function into multiple parts.
Everything after await becomes a microtask continuation.

## 6️⃣ await Does NOT Block the Thread

When writing:

```js
await dbCall();
```

Node does NOT:

- Freeze execution
- Stop serving other requests

Instead:

- Current async function pauses
- Control returns to event loop
- Other callbacks execute
- When Promise resolves → function resumes

This is cooperative concurrency.

## 7️⃣ Sequential vs Parallel Awaits

Sequential version:

```js
await fetchA();
await fetchB();
```

Time taken = A + B

Parallel version:

```js
await Promise.all([fetchA(), fetchB()]);
```

Time taken = max(A, B)

Sequential awaits are a common backend performance mistake.

## 8️⃣ Hidden Performance Bug Example

Bad:

```js
for (let user of users) {
   await saveToDB(user);
}
```

Runs sequentially.

Better:

```js
await Promise.all(
   users.map(user => saveToDB(user))
);
```

Runs concurrently.

## 9️⃣ Understanding Promise.all

Given:

```js
const enriched = await Promise.all(
   users.map(user => fetchProfile(user.id))
);
```

Step-by-step:

1. users.map(...) returns an array of Promises
2. Promise.all(...) returns a single Promise
3. await waits for all to resolve
4. enriched becomes an array of resolved values

Example:

If:

```js
fetchProfile(1) → { name: "John" }
fetchProfile(2) → { name: "Alice" }
fetchProfile(3) → { name: "Bob" }
```

Then:

```js
enriched = [
  { name: "John" },
  { name: "Alice" },
  { name: "Bob" }
];
```

Important:

- Order is preserved
- enriched is a normal array
- Not a Promise (because of await)

Without await:

```js
const enriched = Promise.all(...);
```

enriched would be a Promise.

## 🔟 What Happens If One Promise Rejects in Promise.all

If any Promise rejects:

- Promise.all rejects immediately
- Remaining results are ignored
- No partial data is returned
- The whole await throws an error

Example:

```js
try {
   const data = await Promise.all([...]);
} catch (err) {
   console.log("One failed");
}
```

You must handle errors properly.

## 1️⃣1️⃣ Error Handling in Async Code

Using async/await:

```js
try {
   await something();
} catch (err) {
   console.log(err);
}
```

Equivalent to:

```js
something().catch(err => console.log(err));
```

If no catch is used:

Unhandled promise rejection occurs.

## 1️⃣2️⃣ Unhandled Promise Rejection

Example:

```js
async function test() {
   throw new Error("fail");
}

test();
```

If not caught:

- Node emits unhandledRejection
- In newer Node versions, process may terminate
- Dangerous in production

Always handle async errors.

## 1️⃣3️⃣ Concurrency Is Not Parallelism

Using async does NOT create threads.

It:

- Registers callbacks
- Lets event loop interleave execution

True parallelism requires:

- Worker threads
- Cluster
- Multiple processes

## 🧠 Mental Model Summary

- Promise executor runs synchronously
- .then() runs in microtask queue
- async returns a Promise
- await pauses only that function
- Code after await becomes a microtask
- Sequential awaits hurt performance
- Promise.all enables concurrency
- If one Promise rejects → Promise.all rejects
- Unhandled rejections are dangerous
- Async ≠ Parallel

## 📌 Key Interview Traps

❌ “async makes code parallel”  
❌ “await blocks the thread”  
❌ “Promise constructor is asynchronous”  
❌ “Promise.all returns partial results on failure”
