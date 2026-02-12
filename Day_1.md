# 📘 Backend Deep Prep – Day 1  
Node.js Concurrency & Architecture (Interview Revision Notes)

---

## 1️⃣ Is Node.js Single Threaded?

### ✅ Correct Statement

JavaScript execution in Node.js runs on a single main thread, but Node.js itself is not single-threaded.

### Why?

V8 executes JS in a single call stack.

Node uses libuv for asynchronous operations.

libuv provides:

Event loop

Thread pool

OS-level async I/O integration

---

## 2️⃣ High-Level Node Architecture

```text
Node Process
│
├── V8 Engine (Main JS Thread)
│ ├── Call Stack
│ └── Heap
│
├── Event Loop
│
├── libuv
│ ├── Thread Pool (default: 4)
│ └── OS I/O Integration
```

---

## 3️⃣ Call Stack vs Heap

Call Stack

Executes one function at a time

LIFO structure

Blocks if long-running code exists

Heap

Stores objects

Managed by garbage collector

Memory leaks occur if references are retained

---

## 4️⃣ Where Concurrency Comes From

Node achieves concurrency through:

Non-blocking I/O

Delegation of I/O to:

OS kernel (network I/O)

libuv thread pool (FS, crypto, DNS)

The main JS thread never waits for I/O completion.

---

## 5️⃣ What Happens During fs.readFile()

JS thread registers async operation.

libuv sends it to thread pool.

Worker thread performs file read.

On completion, callback is queued.

Event loop pushes callback to call stack.

Response is sent.

Main thread never blocks.

---

## 6️⃣ Network I/O vs Thread Pool

Network I/O

Handled by OS (epoll/kqueue/IOCP)

Does NOT use thread pool

Scales efficiently

Thread Pool Used For:

File system operations

Crypto (bcrypt async)

DNS

Compression

Default size = 4  
Can increase using:

UV_THREADPOOL_SIZE=8 node app.js

---

## 7️⃣ Why Node Handles 10,000 Concurrent Requests

Because:

It does NOT create one thread per request.

It registers async tasks.

OS handles waiting.

Event loop processes callbacks when ready.

Memory remains low because:

No per-request threads

Only request objects + closures in memory

---

## 8️⃣ CPU-Bound vs I/O-Bound

I/O Bound (Node is strong)

DB queries

HTTP calls

File reads

Redis operations

CPU Bound (Node struggles)

bcrypt hashing (sync)

Image processing

Video encoding

Large JSON parsing

Heavy loops

CPU-bound work blocks event loop.

---

## 9️⃣ bcrypt Scenario Under High Load

Using hashSync()

Blocks event loop

Requests processed sequentially

Catastrophic latency

Using hash() (Async)

Uses thread pool

Default 4 threads

Thread pool saturation possible

High latency under 10,000 requests

Bottleneck: CPU + thread pool

---

## 🔟 External API Call Scenario

When using:

await fetch("external-api")

Handled via OS sockets

No thread pool usage

Event loop not blocked

CPU low usage

Possible bottlenecks:

External API rate limits

Socket descriptor limits

Memory pressure

Slow downstream response

---

## 1️⃣1️⃣ What Actually Blocks Node

Infinite loops

Synchronous CPU-heavy tasks

Large synchronous JSON parsing

Thread pool saturation (indirect bottleneck)

---

## 1️⃣2️⃣ What Consumes Memory Under High Concurrency

Request objects

Response objects

Promises

Closures

Buffers

Large payloads

DB result objects¬

Not concurrency itself, but in-flight data size.

---

## 1️⃣3️⃣ Senior-Level Interview Answer

Why does Node handle high concurrency efficiently?

Node uses a single-threaded event loop combined with non-blocking I/O. Instead of creating one thread per request, it delegates I/O operations to the operating system or libuv’s thread pool. This allows it to handle thousands of concurrent connections with minimal memory overhead and without blocking the main thread.

---

## 1️⃣4️⃣ Real Production Bottlenecks

Under high load, Node fails due to:

CPU-bound blocking

Thread pool exhaustion

DB connection pool limits

External API latency

Garbage collection pressure

Memory leaks

---

## 🧠 Mental Model Summary

Node is:

Event-driven

Non-blocking

Concurrency-optimized

I/O efficient

CPU sensitive

---

## 📌 Key Interview Traps to Avoid

❌ “Node creates one thread per request.”  
❌ “Promises create parallel execution.”  
❌ “All async operations use thread pool.”
