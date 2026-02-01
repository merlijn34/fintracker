# Product Requirements Document: Personal Finance Platform
## Master PRD

**Product Name:** PersonalOS (or your choice of name)  
**Version:** 1.0.0  
**Last Updated:** February 2026  
**Status:** Product Definition Phase

---

## 1. EXECUTIVE SUMMARY

PersonalOS is an open-source, self-hosted personal finance platform designed to give users complete control and visibility over their financial life. Unlike existing solutions like Mint, YNAB, or Personal Capital, PersonalOS treats users' data as their own—stored on their servers, not third-party clouds.

The platform consolidates financial data from multiple sources (bank accounts, credit cards, investments, loans, real estate, vehicles) into a unified dashboard, providing net worth tracking, budgeting, expense categorization, investment portfolio management, and financial planning capabilities.

### Target Users
- Privacy-conscious individuals unwilling to share financial data with SaaS companies
- Power users who want customization and complete control
- Self-hosters comfortable with technical deployment
- Individuals seeking comprehensive wealth management without advisory fees

### Key Differentiators
1. **Data Ownership**: 100% user-hosted—no cloud company has access to financial data
2. **Open Source**: Transparent code, community-driven development, no vendor lock-in
3. **Comprehensive**: Beyond budgeting—includes investing, net worth, planning, and insights
4. **Smart Transfers**: Treats transfers as first-class citizens, excluding them from budgets automatically
5. **Multi-Currency**: Global support with proper currency conversion and multi-currency accounts
6. **Extensible**: Pluggable architecture for third-party integrations

---

## 2. PRODUCT VISION & GOALS

### Vision Statement
"The operating system for your personal finances—complete control, complete visibility, complete peace of mind."

### Strategic Goals
1. **MVP Launch**: Deliver core financial tracking functionality in 3 months
2. **Data Accuracy**: Achieve 100% calculation accuracy for net worth, budgets, and analytics
3. **Community**: Build an engaged open-source community with active contributors
4. **Scalability**: Support users with 10,000+ transactions efficiently
5. **UX Excellence**: Make complex financial data accessible and actionable for non-experts

### Non-Goals (Phase 1)
- Direct bank integration via Plaid/Teller (manual entry + CSV import only)
- AI-driven insights or machine learning recommendations
- Mobile app (PWA-responsive web is the MVP)
- Cryptocurrency portfolio tracking
- Tax planning or integration
- Financial advisor marketplace

---

## 3. PRODUCT SCOPE

### Phase 1: Core Platform (MVP - 3 months)
- User authentication and account management
- Multi-account support (create/edit/delete)
- Transaction management (create, edit, delete, categorize)
- Account types: Checking, Savings, Credit Card, Investment, Loan, Property, Vehicle
- Basic budgeting (set limits, track spending)
- Net worth dashboard and historical tracking
- Transfer detection and exclusion from budgets
- CSV import for transactions
- Basic charts and analytics
- Dark/light theme
- Responsive web design (desktop-first, mobile-friendly)

### Phase 2: Enhanced Tracking & Planning (Post-MVP)
- Investment portfolio management with buy/sell trades
- Recurring transactions and bill reminders
- Transaction rules engine for auto-categorization
- Multi-currency support with exchange rates
- Account reconciliation
- Financial goals tracking
- Retirement planning projections

### Phase 3: Advanced Features (Future)
- Bank integration (Plaid/Teller)
- AI-powered transaction categorization
- Collaborative family accounts
- Mobile app
- Advanced reporting and tax exports

---

## 4. CORE FEATURES

### 4.1 Authentication & Account Management

**User Registration & Login**
- Email-based authentication
- Password reset functionality
- Optional 2FA support
- One user per instance (single-tenant by default)

**Account Settings**
- Profile management (name, email, password)
- Preferred currency selection
- Theme preference (light/dark)
- Default account currency
- Export settings

### 4.2 Account Management

**Supported Account Types**
1. **Checking Account**: Regular spending account
2. **Savings Account**: Interest-bearing savings
3. **Credit Card**: Payment card with credit limit
4. **Investment Account**: Brokerage account for stocks/ETFs
5. **Loan Account**: Debt accounts (mortgage, auto loan, personal loan)
6. **Property**: Real estate ownership
7. **Vehicle**: Car/motorcycle ownership
8. **Other**: Catch-all for other assets/liabilities

**Account Operations**
- Create new account
- View account details (name, type, institution, account number, balance)
- Edit account information
- Archive/hide accounts
- Set opening balance
- Track account balance history over time
- Manual balance reconciliation (adjust to known balance without creating adjustment transaction)

**Account Balance History**
- Store daily/weekly snapshots of account balances
- Use for net worth calculations over time
- Enable historical trend analysis
- Handle multi-currency conversions

### 4.3 Transaction Management

**Transaction Attributes**
- Date of transaction
- Amount (stored with currency and precision for decimal places)
- Merchant/Description
- Category (optional, can be empty for transfers)
- Account source
- Related account (for transfers)
- Type: Income, Expense, Transfer, Investment Trade, Other
- Tags (optional, for custom grouping)
- Notes
- Reconciliation status (cleared/uncleared)

**Transaction Operations**
- Create transaction (manual entry)
- Edit transaction details
- Delete transaction
- Bulk import from CSV
- Search and filter transactions
- Sort by date, amount, merchant, category
- Pagination and infinite scroll support

**Transfer Detection**
- Auto-detect transfers between user's own accounts
- Exclude transfers from budget calculations
- Distinguish between:
  - Excluded transfers (credit card payment, transfer between savings)
  - Included transfers (loan payment, salary deposit to investment account)
- User can override auto-detection for special cases

**Transaction Categories**
- Predefined category hierarchy (Food > Groceries, Dining Out)
- User can create custom categories
- Categorize or leave uncategorized
- Tag transactions for additional grouping
- Support for "One-time Expense" flag
- Default categories:
  - Income (Salary, Bonus, Investment Returns, Gifts, Other Income)
  - Expenses (Food & Dining, Transportation, Utilities, Entertainment, Shopping, Healthcare, etc.)
  - Transfers (excluded from budgets)

### 4.4 Budgeting

**Budget Features**
- Set monthly spending limits by category
- Compare actual spending vs budget
- Visual progress indicators (budget vs spent percentage)
- Support for recurring budget templates
- Exclude transfers from budget calculations
- Mark transactions as "one-time" to exclude from budget
- Budget rollover (carry over unspent amount to next month)

**Budget Views**
- Monthly breakdown by category
- Year-to-date comparison
- Category-specific budget details
- Budget alerts when nearing limits

### 4.5 Net Worth Tracking

**Net Worth Dashboard**
- Display total net worth (all assets - all liabilities)
- Breakdown by account type
- Historical net worth chart (daily, weekly, monthly)
- Trend analysis

**Net Worth Calculation**
- Sum all asset accounts (with conversion to user's preferred currency)
- Subtract all liability accounts
- Update in real-time as transactions are added
- Handle multi-currency accounts with exchange rates

**Asset & Liability Tracking**
- Assets: Bank accounts, investments, property, vehicles, cryptocurrency (future)
- Liabilities: Credit cards, loans, mortgages
- Custom asset/liability categories

### 4.6 Investment Portfolio

**Portfolio Tracking**
- Track investment trades (buy/sell)
- Store investment details: ticker, quantity, price, date, fees
- Calculate cost basis
- Calculate unrealized gains/losses
- Calculate returns percentage

**Investment Operations**
- Add buy/sell trades
- Adjust cost basis
- View transaction history
- Portfolio composition view (pie chart of asset allocation)
- Benchmark against market indices (future)

### 4.7 Data Import & Export

**Import Features**
- CSV import for transactions
- Bulk account setup
- Support for common formats (CSV with headers)
- Import preview and validation
- Duplicate detection

**Export Features**
- Export transactions (CSV, JSON)
- Export account data
- Export budget reports

### 4.8 Analytics & Reporting

**Dashboards**
- Summary dashboard with key metrics
- Net worth over time
- Spending by category (pie, bar, line charts)
- Income vs expenses
- Account breakdown

**Reports**
- Monthly spending report by category
- Annual summary
- Budget performance report
- Cash flow analysis

**Charts & Visualizations**
- Line charts (net worth, account balance trends)
- Pie charts (asset allocation, spending breakdown)
- Bar charts (category spending comparison)
- Sparklines (quick trends)

### 4.9 Multi-Currency Support

**Currency Features**
- Support for all ISO 4217 currencies
- Set account currency independently
- Set user's preferred currency for display
- Exchange rate handling:
  - Store exchange rates with date
  - Use historical rates for accuracy
  - Support manual rate entry or API integration (future)
- Convert amounts when displaying consolidated net worth
- Track gains/losses from currency fluctuations

### 4.10 User Interface & Experience

**Design Principles**
- Clean, minimal interface
- Dark and light mode support
- Responsive design (mobile-first approach)
- Accessibility (WCAG 2.1 AA minimum)
- Fast load times
- Intuitive navigation

**Key Pages**
- Dashboard/Overview
- Accounts (list and detail views)
- Transactions (table view with filters, search)
- Budgets (setup and monitoring)
- Analytics (charts and reports)
- Settings
- Import/Export

**Mobile Responsiveness**
- Touch-friendly buttons and inputs
- Readable charts on small screens
- Responsive tables that convert to cards on mobile
- Bottom navigation or hamburger menu
- Full-screen PWA capability

---

## 5. DATA MODEL

### Core Entities

```
User
  - id (UUID)
  - email (unique)
  - password_hash
  - name
  - preferred_currency
  - theme_preference (light|dark|system)
  - created_at
  - updated_at

Account
  - id (UUID)
  - user_id (FK)
  - name
  - account_type (enum: checking, savings, credit_card, investment, loan, property, vehicle, other)
  - currency (ISO 4217 code)
  - institution_name
  - account_number
  - opening_balance
  - opening_date
  - active (boolean)
  - created_at
  - updated_at

AccountBalance (for historical tracking)
  - id (UUID)
  - account_id (FK)
  - balance (decimal with precision)
  - date
  - created_at

Transaction
  - id (UUID)
  - user_id (FK)
  - account_id (FK)
  - date
  - amount (decimal, always positive)
  - currency (ISO 4217 code)
  - merchant_description
  - category_id (FK, nullable)
  - transaction_type (enum: income, expense, transfer, investment_trade, other)
  - related_account_id (FK, nullable - for transfers)
  - is_transfer (boolean, auto-detected)
  - is_excluded_from_budget (boolean)
  - is_one_time_expense (boolean)
  - reconciliation_status (enum: uncleared, cleared, reconciled)
  - notes
  - created_at
  - updated_at

Category
  - id (UUID)
  - user_id (FK)
  - name
  - parent_category_id (FK, nullable - for hierarchy)
  - color (hex)
  - icon
  - custom (boolean - user-created or system default)
  - created_at
  - updated_at

Tag
  - id (UUID)
  - user_id (FK)
  - name
  - created_at

TransactionTag (join table)
  - transaction_id (FK)
  - tag_id (FK)

Budget
  - id (UUID)
  - user_id (FK)
  - category_id (FK)
  - month_year (YYYY-MM)
  - limit_amount (decimal)
  - created_at
  - updated_at

InvestmentTrade
  - id (UUID)
  - account_id (FK)
  - transaction_id (FK, optional)
  - ticker
  - quantity
  - price_per_share
  - trade_date
  - trade_type (buy|sell)
  - fees
  - created_at
  - updated_at

ExchangeRate
  - id (UUID)
  - from_currency
  - to_currency
  - rate
  - date
  - source (api|manual)
  - created_at

Preference
  - user_id (FK)
  - key
  - value
  - created_at
  - updated_at
```

---

## 6. USER WORKFLOWS

### Workflow 1: Initial Setup
1. User signs up with email and password
2. Sets preferred currency and theme
3. Creates first account(s) or imports from CSV
4. Reviews transactions that were imported/entered
5. Sets up initial budget categories and limits
6. Views dashboard with net worth summary

### Workflow 2: Daily Transaction Entry
1. User logs in
2. Reviews dashboard
3. Clicks "Add Transaction"
4. Enters amount, merchant, date, category
5. Transaction saved and categorized
6. Returns to dashboard

### Workflow 3: Bulk Import
1. User exports data from bank (CSV format)
2. Navigates to Import page
3. Selects CSV file and maps columns
4. Reviews preview of transactions to be imported
5. Handles duplicates (if any)
6. Confirms import
7. Transactions now visible in app

### Workflow 4: Monthly Budget Review
1. User navigates to Budget section
2. Views monthly spending by category
3. Sees actual spending vs budget limits
4. Identifies overspend categories
5. Can adjust categories marked as one-time
6. Can adjust next month's budget

### Workflow 5: Net Worth Check
1. User navigates to Dashboard/Net Worth page
2. Sees current total net worth
3. Views breakdown by account type
4. Sees historical trend chart
5. Identifies changes and patterns

### Workflow 6: Transfer Handling
1. User creates transaction between two accounts (e.g., checking → savings)
2. System auto-detects as transfer
3. Transaction appears in both accounts
4. Transfer is excluded from budget calculations
5. Net worth unchanged by transfer

---

## 7. TECHNICAL REQUIREMENTS

### Technology Stack (Agnostic - Adapt to Your Stack)

**Frontend Options**
- React / Vue / Angular / Svelte
- TypeScript recommended
- Tailwind CSS for styling
- Responsive design framework
- State management (Redux, Zustand, Pinia, etc.)

**Backend Options**
- Node.js (Express, NestJS, etc.)
- Python (Django, FastAPI, etc.)
- Ruby on Rails (as in original Maybe)
- Go (for high performance)
- Java (Spring Boot)

**Database**
- PostgreSQL (relational, excellent for financial data)
- MySQL (alternative relational)
- SQLite (for single-user self-hosted only)

**Hosting & Deployment**
- Docker containerization
- Docker Compose for local deployment
- Kubernetes support (optional, for larger deployments)
- CI/CD pipeline (GitHub Actions, GitLab CI, etc.)

### API Design

**REST API** (alternative: GraphQL)
- RESTful endpoints for all resources
- Proper HTTP status codes
- JWT or session-based authentication
- Rate limiting
- Input validation
- Error responses with descriptive messages

**Key Endpoints**
```
/api/v1/auth/*
/api/v1/accounts
/api/v1/accounts/:id
/api/v1/accounts/:id/balance-history
/api/v1/transactions
/api/v1/transactions/import
/api/v1/transactions/bulk
/api/v1/transactions/:id
/api/v1/categories
/api/v1/budgets
/api/v1/budgets/:id
/api/v1/reports/net-worth
/api/v1/reports/spending
/api/v1/dashboard
/api/v1/settings
```

### Performance Requirements
- Dashboard load time: < 2 seconds
- Transaction list load (100 items): < 1 second
- Search/filter: < 500ms
- Support 10,000+ transactions without significant degradation
- Database queries optimized with proper indexing
- Caching strategy for frequently accessed data

### Security Requirements
- HTTPS only (TLS 1.2+)
- Password hashing (bcrypt, Argon2)
- SQL injection prevention (parameterized queries)
- CSRF protection
- XSS prevention
- Secure session management
- Rate limiting on auth endpoints
- Input validation and sanitization
- No sensitive data in logs
- Optional 2FA support

### Data Integrity
- Transaction amount precision: 2+ decimal places
- Currency conversion using standard rates
- Double-entry bookkeeping principles (optional advanced feature)
- Account reconciliation support
- Audit trails for data changes (optional)

---

## 8. SUCCESS METRICS

### Key Performance Indicators (KPIs)
1. **User Adoption**: 100+ self-hosted instances in first 6 months
2. **Data Completeness**: 95%+ of imported transactions categorized (auto or manual)
3. **Calculation Accuracy**: 100% accuracy for net worth and budget calculations
4. **Performance**: Page load < 2 seconds, transaction search < 500ms
5. **Community**: 50+ GitHub contributors, 5+ active maintainers
6. **Reliability**: 99.5% uptime for typical deployments
7. **User Retention**: 80%+ monthly active users (for self-hosted)

### Measurement Methods
- Usage analytics (open-source friendly)
- Community engagement (GitHub issues, discussions)
- Performance monitoring (optional telemetry)
- User feedback (surveys, issue tracker)
- Bug/error tracking

---

## 9. CONSTRAINTS & DEPENDENCIES

### Constraints
- Self-hosted only (no official SaaS in Phase 1)
- Manual transaction entry or CSV import (no bank APIs in Phase 1)
- Single-user per instance (no multi-user in Phase 1)
- No mobile app (PWA only)
- US English language (Phase 1)

### Dependencies
- Open-source libraries and frameworks (must be compatible with AGPLv3 or similar)
- PostgreSQL or compatible database
- Docker and Docker Compose (for deployment)
- Third-party optional services (exchange rates API, etc.)

---

## 10. RELEASE PLAN

### Phase 1 (MVP) - Months 1-3
- User authentication
- Account management (CRUD)
- Transaction management (create, edit, delete, categorize)
- CSV import
- Basic budgeting
- Net worth dashboard
- Charts and analytics

**Release Target**: v0.1.0 (Beta)

### Phase 2 - Months 4-6
- Investment tracking
- Transaction rules/automation
- Multi-currency support
- Advanced reporting
- Account reconciliation
- Recurring transactions

**Release Target**: v0.2.0 (RC)

### Phase 3 - Months 7+
- Bank integration (Plaid/Teller)
- AI-powered categorization
- Mobile app
- Family sharing
- Tax features

**Release Target**: v1.0.0 (Stable)

---

## 11. APPENDIX: DEFINITION OF DONE

A feature is considered complete when:
- ✅ Code is written and reviewed
- ✅ Unit tests pass (80%+ coverage)
- ✅ Integration tests pass
- ✅ Manual testing completed
- ✅ Accessibility requirements met
- ✅ Documentation updated
- ✅ No critical security issues
- ✅ Performance benchmarks met
- ✅ UI/UX follows design system
- ✅ Database migrations tested
- ✅ Backwards compatibility verified

---

## 12. REFERENCES & RESOURCES

- Original Maybe Finance: https://github.com/maybe-finance/maybe
- Sure (community fork): https://github.com/we-promise/sure
- YNAB: https://www.youneedabudget.com/
- Actual Budget: https://actualbudget.org/
- Firefly III: https://www.firefly-iii.org/
- GnuCash: https://www.gnucash.org/

---

**Document Version**: 1.0  
**Last Updated**: February 1, 2026  
**Created By**: AI Research Team  
**Approval Status**: Draft - Ready for Stakeholder Review
