## **Distributed Workflow Orchestration Engine (Node.js)**

This project implements a simplified but production-grade **workflow orchestration engine**, similar in concept to Temporal, Cadence, or Netflix Conductor. The goal is to build a backend platform that can execute long-running, fault-tolerant workflows with strong guarantees such as durability, state persistence, retries, timers, and distributed task execution.

Unlike a typical monolithic job scheduler, this system maintains a complete execution history, supports replay-based workflow recovery, and coordinates workers across multiple nodes without duplicating executions.

---

## **What This System Does**

### **1. Workflow Definition & Registration**

Developers can define workflows using a JavaScript/TypeScript SDK:

```js
defineWorkflow("userSignupFlow", async (ctx) => {
  await ctx.step("sendWelcomeEmail");
  await ctx.wait(60 * 60 * 1000); // wait 1 hour
  await ctx.step("sendFollowupEmail");
});
```

The engine stores workflow definitions and ensures each step runs exactly once.

### **2. Durable, Crash-Resistant Scheduling**

All workflow state, steps, and timers are persisted.
If the server crashes or restarts, workflow execution resumes automatically from the last known state.

### **3. Worker Pool System**

Dedicated worker processes execute tasks.
Workers can be added or removed without interrupting active workflows.

### **4. Event Sourcing (Execution History)**

Every workflow event is written to an append-only event log:

* task started
* task completed
* task failed
* retry triggered
* timer created/fired
* workflow finished

This enables replay, debugging, and accurate state recovery.

### **5. Concurrency Control**

To prevent duplicate execution across multiple engine instances, the system uses:

* distributed locks
* leases
* idempotency keys

This ensures safe scaling across multiple machines.

### **6. Monitoring Dashboard**

A built-in dashboard visualizes:

* running workflows
* completed workflows
* failed tasks
* worker status
* pending timers

---

## **How It Works (High-Level Architecture)**

1. **API Server**
   Accepts workflow start requests, stores definitions, exposes monitoring endpoints.

2. **Scheduler Engine**
   Reads workflow state, resolves next step, manages durable timers, applies retry logic.

3. **Event Store**
   Stores all workflow actions using event-sourcing patterns.

4. **Queue System**
   Pushes tasks to distributed workers using Redis/BullMQ or a custom queue.

5. **Worker Nodes**
   Pull tasks, execute steps, report results back to the engine.

6. **State Store**
   Persists workflow metadata, execution state, and active timers.

7. **Dashboard**
   Real-time status via WebSockets or polling.

---

## **Technologies Used**

### **Core Backend**

* **Node.js** (execution engine + API server)
* **TypeScript** (type-safe workflow API & internal engine modules)
* **Express.js / Fastify** (API layer)
* **Node Cluster / PM2** (process management)

### **Data & Storage**

* **PostgreSQL**
  For workflow state, timers, metadata, and event logs.
* **Redis**
  For distributed locking, worker queues, and short-lived state.

### **Queues & Workers**

* **BullMQ** or **Custom Redis-based Queue**
  For task distribution.

### **Concurrency & Distributed Coordination**

* Redis Redlock (or PostgreSQL advisory locks)

### **Observability**

* Prometheus metrics
* Winston / Pino logging
* WebSocket-based dashboard

### **Developer Workflow**

* Docker & docker-compose
* Jest / Supertest for unit tests
* Prettier + ESLint for formatting and linting

---

## **How the Project Will Be Built (Step-By-Step Roadmap)**

### **Phase 1 — Core Foundations**

* Set up API server (TypeScript + Express/Fastify)
* Initialize PostgreSQL schema
* Build event-sourcing model for workflow execution history
* Implement workflow registration & metadata storage

### **Phase 2 — Scheduler Engine**

* Implement deterministic workflow executor
* Implement durable timers (DB-backed)
* Add retry logic (fixed, exponential backoff)

### **Phase 3 — Worker System**

* Build worker queue using Redis
* Implement distributed locks to guarantee exactly-once execution
* Add worker heartbeat & monitoring

### **Phase 4 — Distributed Architecture**

* Support multiple scheduler instances
* Ensure crash recovery through event replay
* Implement idempotency keys for tasks

### **Phase 5 — Dashboard & Monitoring**

* Build real-time dashboard (Next.js or plain React)
* Add workflow graphs, active timers, job execution logs
* Add Prometheus metrics exporter

### **Phase 6 — SDK for Developers**

* Build a simple workflow DSL
* Support steps, waits, retries, signals, external triggers

---

## **System Architecture Diagram (Explanation)**

The architecture diagram shown above (image group) represents the following flow:

```
Client → API Server → Event Store → Scheduler → Queue → Workers → Event Store
                                                        ↓
                                               Dashboard WebSocket Feed
```

Key properties:

* All state is durable.
* The scheduler and workers are horizontally scalable.
* No single point of failure (SPoF) in workflow execution.
* Crashes don’t corrupt state; workflows resume automatically.

---

## **Why This Project Matters**

This is infrastructure, not application code. Building it demonstrates mastery of:

* distributed systems
* queue-based architectures
* idempotent execution
* fault tolerance
* workflow engines
* state machines
* event sourcing
