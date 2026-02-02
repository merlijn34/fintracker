# Milestones and Build Plan

## Implementation Phases

### Phase 1: Foundation (MVP Core)

**Objective**: Basic account management and transaction tracking

**Backend Tasks**:
- [ ] Project setup (NestJS, Prisma, PostgreSQL)
- [ ] Database schema migration from Rails
- [ ] Auth module (JWT, sessions, MFA)
- [ ] Users module with profile management
- [ ] Families module with settings
- [ ] Accounts module (all 10 types)
- [ ] Entries module (base)
- [ ] Transactions module with CRUD
- [ ] Categories module with hierarchy
- [ ] Tags module with polymorphic tagging
- [ ] Balance calculation service
- [ ] API rate limiting

**Frontend Tasks**:
- [ ] Project setup (Nuxt, Tailwind, Pinia)
- [ ] Design system components (port from Rails)
- [ ] Auth pages (login, register, password reset)
- [ ] Layout components (sidebar, header)
- [ ] Dashboard page with basic balance sheet
- [ ] Accounts list page
- [ ] Account detail page with activity feed
- [ ] Transaction list with filters
- [ ] Transaction CRUD forms
- [ ] Categories management
- [ ] Tags management
- [ ] Settings pages (profile, preferences, security)

**Infrastructure**:
- [ ] Docker Compose configuration
- [ ] Environment variable management
- [ ] Basic CI pipeline (lint, test)

**Deliverables**:
- Working login/registration
- Manual account creation for all types
- Transaction management with categories/tags
- Basic dashboard with net worth

---

### Phase 2: Advanced Financial Features

**Objective**: Investment tracking, budgets, and import system

**Backend Tasks**:
- [ ] Holdings module
- [ ] Securities module with price tracking
- [ ] Trades module
- [ ] Valuations module
- [ ] Transfers module with matching algorithm
- [ ] Budgets module with category allocations
- [ ] Import module (all types)
- [ ] Background jobs with BullMQ
- [ ] Sync module with state machine
- [ ] Exchange rates module
- [ ] Multi-currency conversion

**Frontend Tasks**:
- [ ] Holdings list and portfolio view
- [ ] Trade entry forms
- [ ] Investment account detail view
- [ ] Budget management pages
- [ ] Budget donut chart (D3.js)
- [ ] Import wizard (multi-step)
- [ ] Transfer management
- [ ] Time series charts (D3.js)
- [ ] Sparkline components
- [ ] Real-time sync status (WebSocket)

**Deliverables**:
- Full investment tracking
- Monthly budgeting
- CSV import capability
- Transfer detection and management
- Multi-currency support

---

### Phase 3: Integrations

**Objective**: External service integrations and automation

**Backend Tasks**:
- [ ] Plaid integration module
- [ ] Plaid webhook handling
- [ ] Rules engine module
- [ ] AI chat module with OpenAI
- [ ] Function calling implementation
- [ ] Synth API integration for market data
- [ ] Data export module

**Frontend Tasks**:
- [ ] Plaid Link integration
- [ ] Bank connection management
- [ ] Connection status indicators
- [ ] Rules builder UI
- [ ] AI chat sidebar
- [ ] Chat conversation view
- [ ] Data export UI

**Deliverables**:
- Automatic bank syncing via Plaid
- AI financial assistant
- Automated transaction rules
- Market data for investments
- Full data export

---

### Phase 4: Polish and Production

**Objective**: Performance, stability, and production readiness

**Backend Tasks**:
- [ ] Query optimization
- [ ] Redis caching layer
- [ ] API documentation (OpenAPI)
- [ ] Comprehensive error handling
- [ ] Security audit
- [ ] Load testing

**Frontend Tasks**:
- [ ] Performance optimization
- [ ] Code splitting and lazy loading
- [ ] Accessibility audit and fixes
- [ ] Mobile responsiveness polish
- [ ] Error boundaries
- [ ] Loading state polish
- [ ] Empty state designs

**Infrastructure**:
- [ ] Production deployment config
- [ ] Monitoring setup (Sentry)
- [ ] Backup configuration
- [ ] Documentation

**Deliverables**:
- Production-ready application
- Full documentation
- Monitoring and alerting
- Performance optimized

---

## Dependencies and Sequencing

```
Phase 1 (Foundation)
├── Auth (blocks everything)
├── Users & Families (blocks accounts)
├── Accounts (blocks transactions)
├── Transactions (blocks budgets, imports)
└── Categories & Tags (blocks transactions)

Phase 2 (Advanced)
├── Holdings & Securities (independent)
├── Budgets (depends on categories)
├── Imports (depends on transactions)
├── Transfers (depends on transactions)
└── Sync System (depends on accounts)

Phase 3 (Integrations)
├── Plaid (depends on accounts, sync)
├── Rules (depends on transactions)
├── AI Chat (depends on all data modules)
└── Market Data (depends on securities)

Phase 4 (Polish)
└── All previous phases complete
```

## Relative Effort Estimates

| Phase | Backend | Frontend | Total |
|-------|---------|----------|-------|
| Phase 1: Foundation | Large | Large | X-Large |
| Phase 2: Advanced | Large | Medium | Large |
| Phase 3: Integrations | Medium | Small | Medium |
| Phase 4: Polish | Small | Medium | Medium |

## Testing Strategy

### Unit Tests

**Backend**:
- Service methods
- Utility functions
- Validation logic
- Data transformations

**Frontend**:
- Composables
- Store actions/getters
- Utility functions
- Form validation

### Integration Tests

**Backend**:
- API endpoint tests
- Database operations
- External API mocks (Plaid, OpenAI)
- Job processors

**Frontend**:
- Component rendering
- User interactions
- API integration (mock)

### End-to-End Tests

**Critical Flows**:
- User registration and login
- Account creation
- Transaction CRUD
- Import wizard
- Plaid connection (sandbox)
- Budget management

**Tools**:
- Playwright or Cypress for E2E
- MSW for API mocking
- Plaid sandbox for integration

### Test Coverage Goals

| Type | Backend | Frontend |
|------|---------|----------|
| Unit | 80% | 70% |
| Integration | 60% | 50% |
| E2E | Critical flows only | Critical flows only |

---

## Summary

This PRD provides comprehensive documentation for rebuilding the Maybe personal finance application. Key points:

1. **59 database tables** mapped to Prisma schema
2. **250+ routes** translated to NestJS endpoints
3. **22 ViewComponents** to port to Vue
4. **4 external integrations** (Plaid, OpenAI, Synth, Stripe)
5. **4 implementation phases** from MVP to production

The rebuild maintains full feature parity while modernizing the stack to Vue 3/Nuxt + NestJS.
