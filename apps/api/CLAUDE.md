# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the NestJS backend API for fintracker. It's part of a pnpm + Turborepo monorepo.

## Commands

```bash
# Development
pnpm dev              # Start dev server with watch mode

# Build
pnpm build            # Build for production

# Testing
pnpm test             # Run all tests
pnpm test:watch       # Run tests in watch mode
pnpm test -- --testPathPattern=users  # Run specific test file

# Code Quality
pnpm lint             # Lint and auto-fix
pnpm typecheck        # TypeScript type checking
```

## Architecture

### Tech Stack

- **Framework**: NestJS with Fastify adapter (not Express)
- **Database**: PostgreSQL via TypeORM
- **Validation**: class-validator

### Code Structure

```
src/
  common/database/      # Shared database utilities (BaseEntity)
  modules/              # Feature modules (users, loan)
  health/               # Health check endpoint
```

### Module Pattern

Each feature module follows NestJS conventions:

- `*.module.ts` - Module definition
- `*.controller.ts` - HTTP endpoints
- `*.service.ts` - Business logic
- `entities/*.entity.ts` - TypeORM entities
- `dto/*.dto.ts` - Data transfer objects

### Base Entity

All entities extend `BaseEntity` from `src/common/database/base.entity.ts` which provides:

- `id: number` (auto-generated)
- `createdAt: Date`
- `updatedAt: Date`

### Domain Model

## Environment Variables

Required in `.env` at monorepo root:

- `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
- `API_PORT` (defaults to 3001)
- `NODE_ENV` (controls TypeORM synchronize mode)

## Database

Start PostgreSQL: `docker-compose up -d` from monorepo root.
TypeORM auto-syncs schema in non-production environments.
