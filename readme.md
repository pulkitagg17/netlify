# 📦 Notlify

**Notlify — Event-Driven Orchestration Platform with BullMQ**

Notlify is a developer-centric backend framework demonstrating advanced asynchronous job orchestration, event-driven architecture, and workflow automation using Node.js, TypeScript, Redis, BullMQ, and Prisma.

It showcases a scalable pattern for handling background jobs, chained workflows, distributed event routing, and integration with external services such as email delivery — making it ideal for learning, interview portfolios, and real-world backend automation. :contentReference[oaicite:1]{index=1}

---

## 🎯 Overview

Notlify solves the common problem of orchestrating complex asynchronous work in modern backends:

✔ Event routing and dispatching  
✔ Workflow sequencing (flows) with dependencies  
✔ Robust retry/backoff strategies  
✔ Queue-based email delivery  
✔ Separation of concerns between API, events, workers, and business logic

Unlike a simple task queue example, Notlify implements **full workflows with ordered job execution**, **domain event abstraction**, and right-sized state hydration in workers.

---

## 🚀 Key Features

### ⚙️ Core Architecture

- **Event-Driven Controllers**  
  Controllers emit domain events instead of immediately executing side effects.

- **Event Router**  
  Central dispatcher that routes events into independent queues and workflows.

- **BullMQ Flows**  
  Defines parent → child job dependencies for orchestrated executions.

- **Worker Consumers**  
  Workers perform ordered actions like sending emails or generating tickets.

- **Prisma + PostgreSQL**  
  Reliable schema + database layer for stateful job hydration.

- **Redis + BullMQ**  
  Distributed job queueing with retries, backoff, and observability hooks.

---

## 🧪 Example Workflow

A booking confirmation workflow in Notlify:

1. User books an event
2. Event router emits `BOOKING_CONFIRMED`
3. A **flow** is created:
   - send booking confirmation email
   - then generate tickets
4. Workers consume steps and execute them reliably

Each step is isolated, retryable, and order-guaranteed.

---

## 📦 Tech Stack

| Component | Technology |
|-----------|------------|
| Language | TypeScript |
| Server | Node.js + Express |
| Database | PostgreSQL (Prisma ORM) |
| Queue | Redis + BullMQ |
| Workflow | BullMQ FlowProducer |
| Email | Resend API |
| Env Management | Dotenv |

---

## 🧰 Prerequisites

Make sure you have the following installed:

- Node.js >= 18.x
- npm or yarn
- Redis Server running locally or remote
- PostgreSQL database (Neon/Supabase/RDS)
- Resend API Key for email

---
## 🚀 Getting Started

### 1️⃣ Clone the Repo

```bash
git clone https://github.com/pulkitagg17/notlify.git
cd notlify

```
### 2️⃣ Install Dependencies
```bash

npm install
# or
yarn install
```
### 3️⃣ Environment Variables

Create a .env file in the root, and fill in required values:
- DATABASE_URL="postgresql://<user>:<pass>@<host>:<port>/<db>?schema=public"
- REDIS_HOST=127.0.0.1
- REDIS_PORT=6379
- RESEND_API_KEY=your_resend_api_key

### 4️⃣ Run Prisma Migrations
```bash
npx prisma migrate dev --name init
```
### 5️⃣ Start Dev Processes
In separate terminals:
```bash
npm run dev          # API server
npm run worker       # Worker processes
```
## 📌 How to Use

After starting:

- Hit API endpoints to emit domain events.

- Watch workers pick up jobs from Redis.

- Jobs execute workflows like sending emails and ticket creation.

- Add more workflows by extending the router and worker cases.

## 🧩 Architecture Diagram

[API Controllers] —> [Event Router] —> [BullMQ Queue / FlowProducer]
                             |                          |
                   [Multiple Redis Queues]        [Worker Consumers]
                             |                          |
                    Email / Ticket / Analytics    Business Logic


## 🧠 Detailed Concepts

### 📍 Event-Driven Design

Controllers never directly trigger side effects. They emit events that are durable and observable. This decouples API surface from workflow logic and allows safe retries or replay of events.

### 📍 Workflow Orchestration

Flows express job dependencies.
Each parent/child relationship models sequential or parallel steps in a workflow (e.g., confirmation → ticketing → analytics).

## ✔ Logging & Observability
Workers log job progress and capture errors:

- worker.on("completed", job => console.log("Completed:", job.id));
- worker.on("failed", (job, err) => console.error("Failed:", job.id, err));

### 🧠 Contributing
Notlify is open-source and ready for collaboration.

If you want to:

- Add metrics and observability dashboards

- Introduce a stress test CLI

- Improve resilience or monitoring

please fork the repo and submit a PR.

## 🧑‍💻 About the Author
Pulkit Aggarwal – Backend engineer passionate about distributed systems, background jobs, and real-world infrastructure challenges.
Connect:
- GitHub: https://github.com/pulkitagg17
- LinkedIn: https://www.linkedin.com/in/pulkit-aggarwal921/
- Email: pulkitaggarwal921@gmail.com







