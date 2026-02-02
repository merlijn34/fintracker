# Maybe Finance Rebuild - Product Requirements Document

This directory contains the comprehensive PRD for rebuilding the Maybe personal finance application using Vue 3/Nuxt + NestJS + PostgreSQL.

## Document Index

| Document | Description |
|----------|-------------|
| [00_OVERVIEW.md](./00_OVERVIEW.md) | Product summary, goals, personas, success criteria |
| [01_FEATURE_INVENTORY.md](./01_FEATURE_INVENTORY.md) | Complete feature list by module with priorities |
| [02_USER_FLOWS.md](./02_USER_FLOWS.md) | Core user flows and interaction sequences |
| [03_SCREENS_AND_UI_PARITY.md](./03_SCREENS_AND_UI_PARITY.md) | Route/screen list, UI components, design system |
| [04_DATA_MODEL.md](./04_DATA_MODEL.md) | Entity definitions, relationships, database schema |
| [05_API_CONTRACTS.md](./05_API_CONTRACTS.md) | REST API endpoints, request/response formats |
| [06_PERMISSIONS_AND_SECURITY.md](./06_PERMISSIONS_AND_SECURITY.md) | Roles, permissions, security considerations |
| [07_BACKGROUND_JOBS_AND_INTEGRATIONS.md](./07_BACKGROUND_JOBS_AND_INTEGRATIONS.md) | Job definitions, external service integrations |
| [08_NONFUNCTIONAL_REQUIREMENTS.md](./08_NONFUNCTIONAL_REQUIREMENTS.md) | Performance, accessibility, SEO, mobile |
| [09_MILESTONES_AND_BUILD_PLAN.md](./09_MILESTONES_AND_BUILD_PLAN.md) | Implementation phases, testing strategy |

## Quick Stats

- **59** database tables
- **250+** routes/endpoints
- **22** UI components to port
- **4** external integrations (Plaid, OpenAI, Synth, Stripe)
- **4** implementation phases

## Target Stack

| Layer | Technology |
|-------|------------|
| Frontend | Vue 3, Nuxt 3, Tailwind CSS, Pinia, D3.js |
| Backend | NestJS, Prisma, PostgreSQL |
| Background Jobs | BullMQ, Redis |
| Authentication | JWT with refresh tokens |
| Deployment | Docker Compose |

## Getting Started

1. Read [00_OVERVIEW.md](./00_OVERVIEW.md) for project context
2. Review [01_FEATURE_INVENTORY.md](./01_FEATURE_INVENTORY.md) for scope
3. Study [04_DATA_MODEL.md](./04_DATA_MODEL.md) before database setup
4. Follow [09_MILESTONES_AND_BUILD_PLAN.md](./09_MILESTONES_AND_BUILD_PLAN.md) for implementation order
