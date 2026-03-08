# Notifly

Event-driven backend service for authentication, booking management, and asynchronous email notifications.

## Architecture

```mermaid
graph TD
    %% API & Storage
    C[Client Request] -->|REST Payload| API[Express API]
    API -->|Mutate State| DB[(PostgreSQL)]
    
    %% Event Brokerage
    API -->|Emit Domain Event| Router{Event Router}
    Router -->|Enqueue Job| Redis[(Redis Broker)]
    
    %% Async Workers
    subgraph BullMQ Worker Nodes
        direction TB
        Redis -.->|Poll| WW[Welcome Mailer]
        Redis -.->|Poll| SW[Sign-In Mailer]
        
        %% Complex Producer Flow
        subgraph FlowProducer [Booking Flow]
            direction BT
            GT[Generate Tickets] -->|Resolves| SC[Send Confirmation]
            SC -->|Resolves| CB[Complete Booking]
        end
        Redis -.->|Poll| FlowProducer
    end

    %% External
    WW & SW & SC -->|SMTP Trigger| Resend[Resend API]

    classDef storage fill:#f9f8f6,stroke:#333,stroke-width:1px;
    classDef worker fill:#f0f9ff,stroke:#0288d1,stroke-width:1px;
    classDef core fill:#f4fbf4,stroke:#2e7d32,stroke-width:1px;
    
    class DB,Redis storage;
    class WW,SW,GT,SC,CB worker;
    class API,Router core;
```

## Tech Stack
- **API Runtime:** Node.js, TypeScript, Express.js
- **Persistence:** PostgreSQL, Prisma ORM
- **Message Broker:** Redis
- **Queue Engine:** BullMQ (Job Queues & Flow Producers)
- **Email Gateway:** Resend
- **Auth Layer:** bcrypt, express-jwt, jose

## Key Features
- Stateless authentication leveraging JWT validation middleware.
- Extensively typed booking management module.
- Event-driven architecture mapped via explicit domain routers.
- Hierarchical processing via BullMQ FlowProducers (e.g., ticket generation gates confirmation emails).
- Highly decoupled queue structure horizontally scalable across Redis nodes.

## Run Locally
```bash
npm install
npm run dev      # Spin up API server
npm run worker   # Boot isolated workers
```

## Project Structure
```text
src/
├── api/        # REST routes & stateless controllers
├── eventRoute/ # Domain-specific event routing mapping
├── events/     # Typed application event schemas
├── infra/      # Redis cluster & Prisma pool initialization 
├── queues/     # BullMQ standard production queues
├── services/   # Isolated core logic implementations
└── workers/    # BullMQ consumers & FlowProducers
```
