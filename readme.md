# Notlify

Event-driven orchestration platform for reliable, ordered background job execution and saga workflows.

## Architecture

```text
HTTP Request
     │
     ▼
[ Express Router ] ───▶ (Emit Domain Event: USER_SIGNED_UP)
     │
     ▼
[ Event Router (Queue Consumer) ]
     │
     ▼ (Decoupled Dispatch)
[ BullMQ Redis Queues & Flows ]
     │
     ├─▶ [ Job Queue: Welcome ]
     │
     └─▶ [ Flow Producer: Booking Confirmation ] ──▶ [ Job: Send Email ] 
                                                           ▲ (Waits on)
                                                     [ Job: Gen Ticket ]
     │
     ▼
[ Stateless Workers ] ──▶ (Hydrate State) ──▶ [ PostgreSQL ]
     │
     ▼
[ External APIs ] (e.g. Resend)
```

## Tech Stack

* Node.js (TypeScript)
* Express.js 5.x
* PostgreSQL (Prisma ORM)
* Redis
* BullMQ
* bcrypt & jose (JWT)

## Key Features

* **Event-Driven Chaining:** Controllers emit events; Routers delegate execution.
* **Deterministic Workflows:** BullMQ `FlowProducers` enforce strict dependencies between long-running child and parent jobs.
* **Resilience:** Idempotent workers with exponential backoff for queue failures.
* **Thin Queues:** Events push minimal identifiers (e.g., `bookingId`); Workers hydrate full state from Postgres to prevent stale properties and reduce Redis memory footprint. 

## Run Locally

```bash
cp .env.example .env     # Configure DATABASE_URL, REDIS_HOST etc.
npm install
npx prisma migrate dev
npm run dev              # Terminal 1: API Server
npm run worker           # Terminal 2: Job Processors
```

## Project Structure

```text
src/
├── api/          # HTTP controllers, route definitions
├── domains/      # Business logic (Auth, Bookings)
├── eventRoute/   # Internal event bus dispatcher
├── events/       # Event schemas
├── infra/        # Redis & DB Singletons
├── queues/       # BullMQ configurations
├── services/     # Third-party integrations
└── workers/      # Stateless background consumers
```
