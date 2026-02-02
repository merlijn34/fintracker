# Maybe Finance Rebuild - PRD Documentation

## Project Overview

This document set provides a comprehensive Product Requirements Document for rebuilding the Maybe personal finance application. The original app is written in Ruby on Rails; this PRD targets a full rebuild using **Vue 3/Nuxt + NestJS + PostgreSQL**.

---

## Product Summary

Maybe is a comprehensive open-source personal finance management application designed to help individuals and families track their financial health through account aggregation, transaction management, budgeting, investment tracking, and AI-powered financial insights. It supports both manual data entry and automatic bank synchronization via Plaid, with multi-currency support and detailed financial reporting.

## Goals

1. **Full Feature Parity**: Recreate 100% of existing functionality with identical user flows
2. **UI/UX Preservation**: Match the existing design system, layouts, and interactive behaviors
3. **Modern Stack**: Leverage Vue 3/Nuxt for reactive frontend, NestJS for type-safe backend
4. **Self-Hostable**: Maintain Docker-based deployment for personal hobby use
5. **Data Migration**: Ensure existing PostgreSQL schema can be migrated or imported

## Non-Goals

1. Managed/SaaS mode features (Stripe billing, subscription management) - optional
2. Mobile app development (focus on responsive web)
3. Performance optimization beyond reasonable levels
4. Support for legacy browsers (target modern browsers only)

## Personas

1. **Personal Finance Enthusiast**: Tracks all accounts manually, wants detailed insights
2. **Hands-Off User**: Links bank accounts via Plaid, relies on automation
3. **Investor**: Focuses on investment tracking, holdings, and portfolio performance
4. **Family Admin**: Manages household finances with multiple family members
5. **Privacy-Focused Self-Hoster**: Deploys on personal infrastructure

## Success Criteria

- All 59 database tables recreated with equivalent relationships
- All 250+ routes functional with matching behavior
- Design system tokens fully ported (colors, typography, spacing)
- Plaid integration working for US region
- Import system handling CSV files identically
- AI chat functional with OpenAI

## Assumptions

1. PostgreSQL 16+ will be used (same as original)
2. Redis will be used for caching and job queues
3. User has access to Plaid API credentials (optional)
4. User has access to OpenAI API credentials (optional)
5. Single-family usage is sufficient (multi-tenant not required for hobby use)

## Open Questions

1. **Q**: Should we maintain OAuth2/Doorkeeper for API access or simplify to JWT-only?
   **Proposed**: JWT-only with refresh tokens for simplicity

2. **Q**: Is Stripe billing integration needed for personal use?
   **Proposed**: Omit for MVP, add as optional feature

