# 🚀 Notlify

> **Event-Driven Orchestration Platform executing reliable workflows with Node.js, Redis, and BullMQ.**

Notlify is a scalable, developer-centric backend framework demonstrating advanced asynchronous job orchestration, event-driven architecture, and deterministic workflow automation. It is engineered to cleanly decouple API layers from heavy side-effects (like email delivery or ticket generation) by leveraging distributed task queues and saga-like workflows.

## 🎯 The Problem it Solves
In modern distributed backends, orchestrating complex async work (especially those with inter-dependent steps or external API calls) often leads to brittle logic scattered across controllers. 

Notlify introduces a structured **Event-Driven Architecture** where controllers simply emit domain events. Centralized Event Routers handle these events, delegating side-effects to resilient **BullMQ Workflows** that guarantee execution ordering, retries on failure, state hydration from Postgres, and clear separation of concerns.

---

## 🏗️ System Architecture

### 1. Event-Driven Controllers
API endpoints validate requests and execute core domain logic (e.g., creating a booking in the database). Instead of triggering side effects immediately, they emit a durable Domain Event (`USER_SIGNED_UP`, `BOOKING_CONFIRMED`).

### 2. Event Router
A centralized dispatcher picks up these events from a generic `event_router` queue and routes them to specific workflow definitions based on the event type.

### 3. Queue & Flow Management (BullMQ)
For simple events, the router drops jobs into isolated Redis queues. For complex multi-step processes, it utilizes **BullMQ FlowProducers** to define dependencies (e.g., *Generate Ticket* must complete before *Send Confirmation Email*).

### 4. Idempotent Workers
Background workers consume queues, re-hydrate necessary state from the database, and execute isolated tasks. Workers support exponential backoff, retry mechanisms, and graceful failure handling.

```text
+---------------+       +------------------+       +-------------------------+
|               |       |                  |       |                         |
|  API Gateway  | ----> |  Event Router    | ----> | BullMQ (Redis) Queues   |
| (Express.js)  | EMIT  | (Queue Consumer) | ROUTE | (Welcome / Booking)     |
|               |       |                  |       |                         |
+-------+-------+       +------------------+       +----+--------------------+
        |                                               |
        v                                               v
+-------+-------+                                  +----+--------------------+
|               |                                  |                         |
|   PostgreSQL  |  <----------------------------   | Independent Workers     |
|   (Prisma)    |    STATE HYDRATION               | (Ticket Gen, Email)     |
|               |                                  |                         |
+---------------+                                  +-------------------------+
```

---

## ✨ Key Features
- **Deterministic Workflows:** Guaranteed ordered execution (Parent → Child jobs) via BullMQ Flows.
- **Domain-Driven Design (DDD) Patterns:** Logic resides in domains, separating API transport from business capabilities.
- **Resilient Event Routing:** Centralized dispatcher managing internal distributed messaging.
- **Robust Failure Policies:** Configurable exponential backoffs and retry semantics for external APIs (e.g., Resend Email API).
- **Stateless Workers:** Workers re-hydrate state via Prisma/Postgres, avoiding passing large payloads through Redis.
- **JWT Authentication:** Secure user sessions relying on the lightweight `jose` cryptography library.

---

## 💻 Tech Stack

- **Runtime:** Node.js (TypeScript)
- **Framework:** Express.js 5.x
- **Database:** PostgreSQL managed via Prisma ORM
- **Task Queues & State:** Redis + BullMQ (FlowProducers)
- **Authentication:** `bcrypt` + `jose` (JWT)
- **External Services:** Resend (Email Delivery)

---

## 📂 Project Structure

```bash
src/
├── api/             # HTTP layer (Express routes, Request/Response validation)
│   ├── controllers/ # Orchestrates domain services and emits events
│   └── routes/      # Endpoint definitions
├── domains/         # Core business logic separated by boundaries (Auth, Booking)
├── eventRoute/      # Core infrastructure for the internal Event Bus & Router
├── events/          # Domain Event type definitions and emitter wrappers
├── infra/           # Shared singletons (Database, Redis connections)
├── queues/          # BullMQ queue configurations and backoff policies
├── services/        # Integrations with 3rd-party providers (Email service)
└── workers/         # Background job processors and flow orchestrators
```

---

## ⚡ Engineering Decisions & Optimizations

1. **State Hydration vs. Fat Payloads:** 
   Event payloads only carry identifiers (e.g., `userId`, `bookingId`). Workers query PostgreSQL to hydrate the state immediately before execution. This prevents Redis memory bloat and avoids race conditions where data in the DB updates between queuing and execution times.
2. **`jose` over `jsonwebtoken`:** 
   Adopted standard web cryptography API library `jose` for generation and verification of JWTs, ensuring a modern, secure, and easily verifiable token lifecycle.
3. **Flow Producers for Saga Patterns:** 
   Instead of using simple queues for complex booking confirmations, BullMQ `FlowProducer` is utilized. It guarantees that an email is only dispatched *after* the heavy ticket generation process has finished successfully, abstracting the complex chaining logic out of the worker.

---

## 🛠️ Installation & Local Development

### Prerequisites
- Node.js >= 18.x
- PostgreSQL database
- Redis Server (local or managed)
- Resend API key

### 1. Environment Setup
Create a `.env` file at the root:
```env
DATABASE_URL="postgresql://user:password@localhost:5432/notlify?schema=public"
REDIS_HOST="127.0.0.1"
REDIS_PORT=6379
JWT_SECRET="super_secret_key"
RESEND_API_KEY="re_123456789"
```

### 2. Install & Migrate
```bash
npm install
npx prisma generate
npx prisma migrate dev --name init
```

### 3. Run the Services
Because Notlify is a distributed system, you need to run both the API server and the Worker processors:
```bash
# Terminal 1 - API Server
npm run dev

# Terminal 2 - Background Workers
npm run worker
```

---

## 🔌 Core API Endpoints

### Authentication
- `POST /auth/sign-up`: Register a new user (`name`, `email`, `password`). Emits `USER_SIGNED_UP`.
- `POST /auth/sign-in`: Authenticate user. Returns JWT. Emits `USER_SIGNED_IN`.

### Bookings (Event Trigger Example)
- `POST /booking/...`: Example endpoints that process payments/bookings and emit `BOOKING_CONFIRMED`.

*Once events are emitted, watch your Worker terminal to see jobs processed reliably in the background.*

---

## 🚀 Future Improvements
- **Idempotency Keys:** Implementing Redis-based distributed locks to ensure zero duplicate job execution.
- **Dead Letter Queues (DLQ):** Route persistently failing jobs to a DLQ for manual inspection.
- **Prometheus/Grafana Observability:** Exporting BullMQ metrics for real-time queuing dashboards.
