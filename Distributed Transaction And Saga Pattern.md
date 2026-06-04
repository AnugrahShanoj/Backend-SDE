# 📘 Backend Deep Prep – Day 19
# Distributed Transactions & Saga Pattern (Deep System-Level Understanding)

# 1️⃣ The Core Problem: Transactions Across Multiple Services

Inside a single database, we can use transactions:

```text
Start Transaction

Update User
Create SOS

Commit
```

If something fails:

```text
Rollback
```

Everything returns to its previous state.

This works because:

👉 One database controls all operations.

---

# 2️⃣ Why Modern Systems Make This Difficult

Modern systems are usually split into multiple services.

Example:

```text
User Service
SOS Service
Notification Service
Analytics Service
```

Each service often owns:

👉 Its own database

Architecture:

```text
SOS Service DB
Notification Service DB
Analytics Service DB
```

No single database controls all operations.

---

# 3️⃣ NeoAegis Example

SOS Workflow:

```text
1. Create SOS
2. Send SMS
3. Create Incident Record
4. Notify Admin Dashboard
```

Now imagine:

```text
Create SOS ✔
Send SMS ✔
Create Incident Record ❌
```

Question:

How do we rollback the SMS?

How do we rollback the SOS?

Traditional database transactions cannot help.

---

# 4️⃣ The Distributed Transaction Problem

Goal:

```text
Either all steps succeed
OR
System remains consistent
```

Challenge:

Each service is independent.

No single transaction manager controls everything.

---

# 5️⃣ Traditional Solution: Two-Phase Commit (2PC)

A coordinator asks every participant:

```text
Can you commit?
```

All services respond:

```text
YES
```

Then coordinator sends:

```text
Commit
```

---

## Problems With 2PC

- Slow
- Complex
- Coordinator becomes bottleneck
- Poor scalability
- Services remain locked while waiting

Because of these limitations:

👉 Modern microservices rarely use 2PC.

---

# 6️⃣ Modern Solution: Saga Pattern

Instead of one giant transaction:

```text
Transaction A
Transaction B
Transaction C
```

Each service performs:

👉 Its own local transaction

If something fails later:

👉 Execute compensating actions

---

# 7️⃣ Understanding Compensation

Traditional Transaction:

```text
Do Action
Rollback Action
```

Saga:

```text
Do Action
Compensating Action
```

Important:

Compensation is NOT rollback.

---

# 8️⃣ NeoAegis Saga Example

Workflow:

```text
Create SOS ✔
Send SMS ✔
Create Incident Record ❌
```

Instead of rollback:

Execute compensations:

```text
Delete SOS Record
Send Cancellation SMS
Mark Incident As Failed
```

The system reaches a consistent business state.

---

# 9️⃣ Rollback vs Compensation

## Rollback

Returns system to previous state.

Example:

```text
Insert Record
Rollback
Record Removed
```

The original operation disappears.

---

## Compensation

Creates a new action that offsets previous action.

Example:

```text
Send SMS
```

Compensation:

```text
Send "Previous Alert Cancelled" SMS
```

Original SMS still happened.

We cannot erase history.

---

# 🔥 Fundamental Insight

Database updates are:

```text
Reversible
```

Real-world side effects are:

```text
Irreversible
```

Examples:

- SMS sent
- Email sent
- Payment processed
- Push notification delivered

These cannot truly be rolled back.

---

# 1️⃣0️⃣ Why SMS Cannot Be Rolled Back

Suppose:

```text
SOS Created ✔
SMS Sent ✔
Tracking Failed ❌
```

Emergency contact already received:

"🚨 SOS Alert"

Maybe they already:

- Called the user
- Started travelling
- Informed others

Backend cannot:

```text
Unsend SMS
```

Therefore:

👉 Use compensation instead.

---

# 1️⃣1️⃣ Types of Saga

---

## Choreography Saga

Services communicate through events.

Example:

```text
SOS Created Event
        ↓
Notification Service
        ↓
SMS Sent Event
        ↓
Admin Service
```

No central coordinator.

---

### Advantages

- Highly scalable
- Decentralized

### Disadvantages

- Difficult debugging
- Complex event chains

---

# 1️⃣2️⃣ Orchestration Saga

A central coordinator controls workflow.

```text
Saga Coordinator
       ↓
Create SOS
       ↓
Send SMS
       ↓
Create Incident
```

If failure occurs:

Coordinator triggers compensations.

---

### Advantages

- Easier monitoring
- Easier debugging
- Clear workflow

### Disadvantages

- Coordinator becomes dependency

---

# 1️⃣3️⃣ Why Queues Are Important

Saga relies heavily on asynchronous communication.

Common tools:

- Kafka
- RabbitMQ
- BullMQ
- Event Bus

Flow:

```text
SOS Created Event
       ↓
Queue
       ↓
Notification Service
```

Services communicate without direct coupling.

---

# 1️⃣4️⃣ Why Idempotency Is Critical

Events may be retried.

Example:

```text
SOS_CREATED
```

Delivered twice.

Without idempotency:

```text
SMS Sent Twice ❌
```

Therefore:

Saga and Idempotency usually work together.

---

# 1️⃣5️⃣ Trade-Offs

## Traditional Database Transaction

### Pros

- Strong consistency
- Simple rollback

### Cons

- Doesn't scale across services

---

## Saga Pattern

### Pros

- Scalable
- Microservice-friendly
- Works across distributed systems

### Cons

- Eventual consistency
- Complex compensation logic
- Harder debugging

---

# 🧠 Deep System Insight

In distributed systems:

> Consistency is often achieved through compensation rather than rollback.

This is one of the most important principles in modern microservice architecture.

---

# 🎯 Interview-Ready Summary

Traditional database transactions work well within a single database, but they cannot reliably coordinate changes across multiple independent services. Modern distributed systems solve this using the Saga Pattern, where each service performs local transactions and, in case of failure, compensating actions are executed to restore business consistency rather than attempting a true rollback.

---
