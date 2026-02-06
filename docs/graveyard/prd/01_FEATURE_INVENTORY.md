# Feature Inventory

## Complete Feature List by Module

### 1. Authentication & User Management

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Email/password registration | P0 | New user signup with email verification | Optional email confirmation in self-hosted |
| Login with session | P0 | Cookie-based session authentication | 15-min access token + 7-day refresh |
| Password reset | P0 | Token-based password reset (15-min expiry) | Email required |
| Email change confirmation | P1 | Confirm new email before changing | 1-day token expiration |
| User profile update | P0 | Name, profile image | Image variants: 40x40, 80x80, 200x200 |
| Theme preference | P0 | Light/dark/system theme toggle | Persists to user record |

<!-- ### 2. Family & Household Management

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Family settings | P0 | Currency, locale, date format, timezone | 9 date format options |
| Family member roles | P1 | member, admin, super_admin | Admin can invite, super_admin for impersonation |
| Family invitations | P1 | Email-based invites with 3-day expiry | Unique token, role assignment |
| Invite codes | P2 | Require invite code for signup | Site-level setting |
| Data isolation | P0 | All data scoped to family | Critical security requirement | -->

### 3. Account Management

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| **Account Types** | | | |
| Depository (checking/savings) | P0 | Bank accounts with subtypes | HSA, CD, Money Market |
| Credit Card | P0 | Credit cards with APR, limits | available_credit, minimum_payment, annual_fee |
| Investment | P0 | Brokerage accounts | Holdings tracking, 401k, IRA, Roth |
| Loan | P0 | Loans with amortization | rate_type, interest_rate, term_months, initial_balance |
| Property | P1 | Real estate tracking | Address, year_built, area, subtypes |
| Vehicle | P1 | Car/vehicle tracking | make, model, year, mileage |
| Crypto | P1 | Cryptocurrency accounts | Holdings-based like investments |
| Other Asset | P1 | Generic asset | Flexible tracking |
| Other Liability | P1 | Generic liability | Flexible tracking |
| **Account Operations** | | | |
| Manual account creation | P0 | Create accounts with opening balance | Status: draft -> active |
| Account editing | P0 | Update name, settings | Type-specific fields |
| Account deletion | P0 | Soft delete with cascade | scheduled_deletion status |
| Account sync | P0 | Manual sync trigger | Calls balance calculator |
| Bulk sync all | P1 | Sync all accounts at once | Background job |
| Account toggle active/disable | P1 | Enable/disable without deleting | Excluded from calculations |
| Balance reconciliation | P1 | Adjust balance with valuation | Creates Valuation entry |
| Sparkline chart | P0 | Mini balance chart for list | D3.js, 30-day default |

### 4. Transaction Management

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Transaction CRUD | P0 | Create, read, update, delete | Via Entry model |
| Transaction kinds | P0 | standard, funds_movement, cc_payment, loan_payment, one_time | Affects budget inclusion |
| Category assignment | P0 | Single category per transaction | Hierarchical categories |
| Merchant assignment | P0 | Single merchant per transaction | Family or provider merchant |
| Tag assignment | P0 | Multiple tags per transaction | Polymorphic taggings |
| Transaction search | P0 | Full-text search on name | Lowercase index |
| Date range filter | P0 | Filter by date range | Start/end date params |
| Category filter | P0 | Filter by category | Include subcategories |
| Account filter | P0 | Filter by account(s) | Multi-select |
| Amount filter | P1 | Min/max amount range | Decimal comparison |
| Transaction notes | P1 | Free-text notes field | Text column |
| Exclude from reports | P1 | excluded flag | Removes from totals |
| Bulk update | P0 | Update multiple transactions | Category, tags, merchant |
| Bulk delete | P0 | Delete multiple transactions | Confirmation required |
| Locked attributes | P1 | Prevent editing synced fields | JSONB locked_attributes |

### 5. Categories & Tags

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Category CRUD | P0 | Create, edit, delete categories | Family-scoped |
| Subcategories | P0 | Parent-child hierarchy | Max 2 levels |
| Category classification | P0 | income or expense | Affects reporting |
| Category icons | P1 | Lucide icon selection | lucide_icon field |
| Category colors | P0 | Color picker | Hex color |
| Bootstrap defaults | P0 | Initialize default categories | 14 built-in categories |
| Category replacement | P1 | Reassign before delete | Prevent orphan transactions |
| Tag CRUD | P0 | Create, edit, delete tags | Family-scoped |
| Tag colors | P0 | 10 predefined colors | Color picker |
| Tag replacement | P1 | Reassign before delete | Prevent orphan taggings |

### 6. Budgeting

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Monthly budgets | P0 | Budget per calendar month | Auto-created on access |
| Budget categories | P0 | Allocation per category | Synced with expense categories |
| Budgeted spending | P0 | Set total budget amount | Per category allocation |
| Expected income | P1 | Set expected income | Optional tracking |
| Actual vs budgeted | P0 | Compare spending to budget | Real-time calculation |
| Overage detection | P0 | Flag over-budget categories | Percentage calculation |
| Budget navigation | P0 | Navigate months | Limit to 2-year lookback |
| Donut chart visualization | P0 | Visual budget breakdown | D3.js donut chart |
| Median/average calculations | P1 | Historical spending analysis | 3-month rolling |

### 7. Investment Tracking

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Holdings list | P0 | Current positions by security | Aggregated per security |
| Holdings history | P1 | Historical position snapshots | Daily snapshots |
| Security database | P0 | Ticker, name, exchange | Global securities table |
| Security prices | P0 | Historical price data | Daily prices by currency |
| Trades | P0 | Buy/sell transactions | qty, price, currency |
| Cost basis calculation | P1 | Approximate cost basis | FIFO or average |
| Unrealized gains/losses | P1 | Current value vs cost | Per holding |
| Holdings sync | P1 | Gap-fill missing dates | Background job |
| Security health check | P2 | Verify price availability | Scheduled job |

### 8. Transfers

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Create transfer | P0 | Link inflow + outflow transactions | Same amount, opposite signs |
| Auto-match transfers | P1 | Detect matching transactions | Within 4-day window |
| Transfer confirmation | P1 | Confirm suggested matches | pending -> confirmed |
| Reject transfer match | P1 | Prevent future auto-match | RejectedTransfer record |
| Transfer kinds | P0 | funds_movement, cc_payment, loan_payment | Auto-set on match |

<!-- ### 9. Import System

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| CSV upload | P0 | Upload transaction CSV | Max row limit |
| Column mapping | P0 | Map CSV columns to fields | Drag-drop or select |
| Import types | P0 | Transaction, Trade, Account, Mint | Type-specific processors |
| Date format selection | P0 | 10+ date format options | Auto-detect attempt |
| Number format | P0 | Decimal/thousands separator | 4 format options |
| Signage convention | P0 | Inflows positive/negative | Toggle setting |
| Amount type strategy | P1 | Signed amount or separate column | For credit/debit columns |
| Category mapping | P1 | Map CSV categories to system | Fuzzy matching |
| Tag mapping | P1 | Map CSV tags to system | Create if missing option |
| Account mapping | P1 | Map CSV accounts to system | For multi-account imports |
| Import preview | P0 | Preview parsed rows | Edit before confirm |
| Row editing | P0 | Edit individual rows | Before publishing |
| Import publish | P0 | Finalize and create entries | Background job |
| Import revert | P1 | Undo entire import | Deletes created entries |
| Import templates | P2 | Save/reuse import config | Template application |
| Duplicate detection | P1 | Warn on potential duplicates | Date + amount + name | -->

<!-- ### 10. Plaid Integration

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Plaid Link | P0 | Connect bank account | Plaid Link SDK |
| Link token creation | P0 | Generate link token | Short-lived token |
| Access token exchange | P0 | Exchange public token | Store encrypted |
| Account sync | P0 | Pull transactions from Plaid | Incremental with cursor |
| Investment sync | P1 | Pull holdings/trades | Investment accounts |
| Liability sync | P1 | Pull credit card/loan data | APR, minimum payment |
| Webhook handling | P0 | Process Plaid notifications | Sync completion events |
| Connection status | P0 | Good, requires_update | Update mode for re-auth |
| Multiple institutions | P0 | Connect multiple banks | PlaidItem per connection |
| Scheduled sync | P1 | Auto-sync on login | family.auto_sync_on_login |
| Manual sync | P0 | Trigger sync on demand | Per account or all | -->

<!-- ### 11. AI Chat

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Chat creation | P1 | Start new conversation | Auto-title generation |
| Message send | P1 | Send user message | Async response |
| AI response | P1 | OpenAI completion | Streaming support |
| Function calling | P1 | Tool use for data queries | 4 functions defined |
| Get accounts function | P1 | List family accounts | Includes balances |
| Get balance sheet function | P1 | Assets, liabilities, net worth | By account type |
| Get income statement function | P1 | Income/expenses by category | Period-based |
| Get transactions function | P1 | Filtered transaction list | With pagination |
| Chat history | P1 | Load previous chats | Per-user |
| Message retry | P1 | Retry failed response | Error recovery |
| Debug mode | P2 | Show function call details | Development aid | -->

<!-- ### 12. Rules Engine

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Rule CRUD | P1 | Create, edit, delete rules | Family-scoped |
| Rule conditions | P1 | If conditions (compound) | AND/OR logic |
| Condition types | P1 | name, merchant, amount | Operators: contains, equals, gt, lt |
| Rule actions | P1 | Then actions | set_category, set_merchant, set_tags |
| Rule preview | P1 | Count affected transactions | Before applying |
| Rule application | P1 | Apply to matching transactions | Bulk update |
| Auto-apply | P1 | Apply to new transactions | On sync/import |
| Rule activation | P1 | Toggle active/inactive | Skip inactive rules |
| Effective date | P2 | Rule start date | Future-dated rules | -->

### 13. Reporting & Analytics

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Balance sheet | P0 | Assets vs liabilities summary | By account type |
| Net worth calculation | P0 | Total assets - liabilities | Multi-currency aware |
| Net worth time series | P0 | Historical net worth chart | Daily data points |
| Income statement | P0 | Income/expense by category | Period-based |
| Spending by category | P0 | Category breakdown | Donut chart |
| Cash flow sankey | P1 | Money flow visualization | D3.js sankey |
| Account balance series | P0 | Historical balance chart | Line chart with gradient |

<!-- ### 14. Data Export

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Family data export | P1 | Full data backup | ZIP file |
| Export status tracking | P1 | pending, processing, completed, failed | Async job |
| Export download | P1 | Download generated file | Signed URL | -->

### 15. Settings & Preferences

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Profile settings | P0 | Name, email, image | Image upload |
| Security settings | P0 | Password, MFA | Security-sensitive |
| Preferences | P0 | Theme, default period, sidebar | User-level |
| Family settings | P1 | Currency, locale, date format | Admin-only |
| Hosting settings | P1 | API keys, cache clear | Self-hosted only |
| API key management | P1 | Generate/revoke API keys | Scoped access |

<!-- ### 16. Multi-Currency Support

| Feature | Priority | Description | Notes/Edge Cases |
|---------|----------|-------------|------------------|
| Per-account currency | P0 | Currency per account | 3-letter code |
| Family base currency | P0 | Reporting currency | Conversion target |
| Exchange rates | P0 | Historical exchange rates | Daily rates |
| Rate provider (Synth) | P1 | Fetch rates from API | Scheduled job |
| Automatic conversion | P0 | Convert for reporting | Real-time calculation | -->
