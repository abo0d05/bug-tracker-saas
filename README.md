# Real-time Bug Tracking & Workspace Collaboration SaaS

> A prompt/specification for architecting and building the core foundation of a production-ready, secure, full-stack bug tracking SaaS (similar to Jira or Linear), organized as a Monorepo.

## Role

Act as a Senior Principal Software Engineer. Architect and build the core foundation of a production-ready, full-stack **Real-time Bug Tracking & Workspace Collaboration SaaS**. The project must use a **Monorepo** structure.

## Tech Stack Requirements

| Layer | Technology |
|---|---|
| Frontend | Next.js 14 (App Router), React, TypeScript, Tailwind CSS, Zustand |
| Backend | Node.js, Express, TypeScript, Clean Architecture |
| Database & ORM | PostgreSQL, Prisma ORM |
| Real-time & Caching | Socket.io, Redis |
| DevOps | Fully Dockerized environment |

## Components to Generate

### 1. Database Schema (Prisma)
Design a highly scalable schema including models for **Organizations, Users, Projects, Tickets (Status, Priority, Tags), and Comments**.
- Implement proper relational integrity (foreign keys, cascade/restrict rules).
- Add indexes for common query patterns (e.g. tickets by project + status, tickets by assignee + priority).
- Add membership models so roles are enforced at both organization and project level.
- Add an **AuditLog** model and a **RefreshToken** model (hashed token, token family ID, expiry, revoked flag).

### 2. Advanced Authentication & Security (Backend)
- Write a robust JWT authentication middleware handling both **Access** and **Refresh** tokens, stored in **HttpOnly cookies**.
- Implement **Role-Based Access Control (RBAC)** with three strict roles: `OrgAdmin`, `ProjectManager`, `Developer`.
- Hash passwords with **Argon2id**; enforce a password policy and account lockout after N failed attempts.
- Implement **Refresh Token Rotation**: store hashed refresh tokens, issue a new one on every refresh, and revoke the whole token family if a used token is replayed (reuse detection). Support logout via token revocation.
- Cookies: `HttpOnly`, `Secure`, `SameSite=Strict`. Add **CSRF protection** (double-submit token) for state-changing requests.

### 3. Smart Algorithm: Ticket Auto-Assignment Service
When a new **Critical** bug is created, the service must query the database to find the **Developer within that specific project** who currently has the **least number of active critical tickets**, and dynamically assign the ticket to them.
- "Active" means not `Resolved` or `Closed`.
- Handle edge cases: no developers in the project, ties (deterministic tie-break), and concurrent ticket creation (use a transaction with proper locking/isolation so two simultaneous tickets don't pick stale counts).
- Roll back the transaction on any failure.

### 4. Real-time Gateway (WebSockets)
Create a Socket.io service that broadcasts ticket updates (e.g., status changed to `Resolved`) **only to authenticated users currently viewing that specific project's dashboard**.
- Authenticate during the handshake; verify project membership before joining a project room.
- Re-validate on token expiry and rate-limit incoming events.
- Use the Redis adapter so broadcasting works across multiple API instances.

### 5. Complex Frontend Component (React / Next.js)
Build a **Kanban Board** component using Tailwind CSS that:
- Fetches tickets and groups them by status columns.
- Includes the logic structure for Drag-and-Drop state updates (optimistic update, rollback on server failure).
- Uses Zustand for state and subscribes to real-time updates.

### 6. Infrastructure (DevOps)
Provide a comprehensive `docker-compose.yml` orchestrating the Node.js API, the Next.js frontend, the PostgreSQL database, and the Redis cache instance, including environment variables and network bridging.
- Run containers as non-root.
- Do not expose Postgres/Redis ports publicly.
- Keep secrets in `.env` files excluded from git (or Docker secrets).
- Add healthchecks and `depends_on` conditions.

### 7. Security Hardening (applies across all components above)
- Add **Redis-based rate limiting** (strict on `/auth` routes), **Helmet** security headers, and a strict **CORS allowlist**.
- Validate every request body/params/query with **Zod**; sanitize comment content against XSS.
- Enforce **multi-tenant isolation**: every query must be scoped by `organizationId` / project membership to prevent IDOR/BOLA. Never trust IDs from the client alone.
- Record security-relevant actions (login, role change, ticket deletion) in the Audit Log.
- Include a short **threat-model table** (threat → mitigation) mapped to the OWASP Top 10.

## Output Requirements

1. Start with a brief overview of the directory structure.
2. Provide production-ready, **strictly-typed TypeScript** code for each of the 7 points above.
3. Ensure edge cases, error handling, and transaction rollbacks are considered.
