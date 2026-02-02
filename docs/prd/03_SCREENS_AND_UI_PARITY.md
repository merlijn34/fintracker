# Screens and UI Parity

## Route/Screen List

### Authentication (Public)

| Route | Screen | Purpose |
|-------|--------|---------|
| `/login` | Login | Email/password login form |
| `/register` | Register | New user signup |
| `/password_resets/new` | Forgot Password | Request password reset email |
| `/password_resets/:token/edit` | Reset Password | Set new password |
| `/mfa/verify` | MFA Verification | Enter TOTP code |
| `/invitations/:token/accept` | Accept Invitation | Join family via invite |

### Main Application (Authenticated)

| Route | Screen | Purpose |
|-------|--------|---------|
| `/` | Dashboard | Net worth, balance sheet, cashflow |
| `/accounts` | Accounts List | All accounts grouped by type |
| `/accounts/:id` | Account Detail | Transactions, balance chart, settings |
| `/accounts/new` | Add Account | Account type selection |
| `/depositories/new` | New Depository | Bank account form |
| `/investments/new` | New Investment | Brokerage account form |
| `/credit_cards/new` | New Credit Card | Credit card form |
| `/loans/new` | New Loan | Loan form |
| `/properties/new` | New Property | Real estate form |
| `/vehicles/new` | New Vehicle | Vehicle form |
| `/cryptos/new` | New Crypto | Crypto account form |
| `/other_assets/new` | New Other Asset | Generic asset form |
| `/other_liabilities/new` | New Other Liability | Generic liability form |
| `/transactions` | Transaction List | Searchable/filterable list |
| `/transactions/new` | New Transaction | Transaction form modal |
| `/budgets` | Budgets (redirect) | Redirects to current month |
| `/budgets/:month_year` | Budget Detail | Monthly budget with donut chart |
| `/holdings` | Holdings List | Investment positions |
| `/trades/new` | New Trade | Buy/sell form |
| `/imports` | Import History | List of past imports |
| `/imports/new` | New Import | Upload CSV |
| `/imports/:id/*` | Import Wizard | Multi-step import flow |
| `/rules` | Rules List | Automation rules |
| `/rules/new` | New Rule | Rule builder |
| `/categories` | Categories | Category management |
| `/tags` | Tags | Tag management |
| `/family_merchants` | Merchants | Merchant management |
| `/chats` | Chat List | AI conversations |
| `/chats/:id` | Chat Detail | Conversation view |
| `/settings/profile` | Profile Settings | Name, email, image |
| `/settings/preferences` | Preferences | Theme, display options |
| `/settings/security` | Security | Password, MFA |
| `/settings/billing` | Billing | Subscription (optional) |
| `/settings/hosting` | Hosting | Self-hosted config |
| `/settings/api_key` | API Keys | API key management |
| `/onboarding/*` | Onboarding | Welcome flow |
| `/plaid_items/new` | Connect Bank | Plaid Link |

---

## Screen Details

### Dashboard (`/`)

**Layout**: Full width with sidebar, max-w-5xl content

**Sections**:
1. **Header**: "Dashboard" title with period selector dropdown
2. **Net Worth Card**:
   - Large number display
   - Trend indicator (up/down arrow with percentage)
   - Time series chart (D3.js line chart with gradient fill)
3. **Balance Sheet Cards** (2-column grid):
   - Assets card: Total, grouped by account type
   - Liabilities card: Total, grouped by account type
   - Each type expandable to show accounts
4. **Cashflow Sankey** (full width):
   - Income sources on left
   - Expense categories on right
   - Flow lines showing money movement

**States**:
- Loading: Skeleton cards with pulse animation
- Empty: "Add your first account to get started" with CTA button
- Error: Alert banner with retry option

**UI Parity Checklist**:
- [ ] Net worth number uses `text-4xl font-medium`
- [ ] Trend arrow colored (green positive, red negative)
- [ ] Chart uses gradient fill below line
- [ ] Cards use `bg-container rounded-xl shadow-border-xs`
- [ ] Period selector is dropdown with presets
- [ ] Account type icons from Lucide

---

### Accounts List (`/accounts`)

**Layout**: Sidebar visible, main content area

**Sections**:
1. **Header**: "Accounts" title, "Add Account" button
2. **Sync Status Banner** (conditional): Shows if any account syncing
3. **Account Groups** (collapsible):
   - Group header: Type name, total balance
   - Account rows: Icon, name, balance, sparkline, menu

**Components**:
- `AccountCard`: Icon, name, balance, sparkline mini-chart, overflow menu
- Menu items: View, Edit, Sync, Disable, Delete

**States**:
- Loading: Skeleton cards
- Empty: Illustration + "Connect your first account" CTA
- Sync in progress: Spinner on account row

**UI Parity Checklist**:
- [ ] Groups collapsible via `<details>` element
- [ ] Sparklines are 60x24 SVG
- [ ] Balances right-aligned, monospace font
- [ ] Negative balances in red
- [ ] "Add Account" button is primary style

---

### Account Detail (`/accounts/:id`)

**Layout**: Full width with breadcrumb navigation

**Sections**:
1. **Header**: Account name, balance, type badge, actions menu
2. **Tabs**: Activity | Holdings (investment only) | Overview (property/vehicle)
3. **Balance Chart**: Time series showing balance over time
4. **Activity Feed**: Transactions grouped by date

**Tab: Activity**:
- Date headers (Today, Yesterday, specific dates)
- Transaction rows: Time, name, category pill, amount
- Click row to edit in modal

**Tab: Holdings** (Investment accounts):
- Holdings table: Security, quantity, price, value, gain/loss
- Add holding button

**Tab: Overview** (Property/Vehicle):
- Property: Address, year built, area
- Vehicle: Make, model, year, mileage
- Valuation history table

**States**:
- Loading: Skeleton chart and list
- Empty activity: "No transactions yet"
- Sync in progress: Banner with progress

**UI Parity Checklist**:
- [ ] Tabs use `DS::Tabs` component pattern
- [ ] Chart matches time series pattern (gradient fill, trend coloring)
- [ ] Activity feed has sticky date headers
- [ ] Balance reconciliation accessible via menu

---

### Transaction List (`/transactions`)

**Layout**: Full width with filter sidebar option

**Sections**:
1. **Header**: Title, date range selector, "Add Transaction" button
2. **Filters Bar**: Categories, accounts, search, clear filters
3. **Summary Bar**: Income total, expense total, net
4. **Transaction Table**:
   - Checkbox column
   - Date
   - Description/name
   - Category (pill)
   - Account
   - Amount
5. **Bulk Action Bar** (when items selected): Categorize, Delete
6. **Pagination**: Page numbers, prev/next

**Components**:
- `TransactionRow`: Selectable row with hover state
- `CategoryPill`: Colored pill with icon
- `FilterDropdown`: Multi-select with search

**States**:
- Loading: Skeleton table rows
- Empty: "No transactions match your filters"
- Filtered: "Showing X of Y transactions", clear filters link

**UI Parity Checklist**:
- [ ] Table has zebra striping on hover
- [ ] Amount column right-aligned
- [ ] Negative amounts (expenses) no special color
- [ ] Positive amounts (income) green
- [ ] Bulk action bar slides up from bottom
- [ ] Pagination shows current range "1-25 of 150"

---

### Budget Detail (`/budgets/:month_year`)

**Layout**: Full width content

**Sections**:
1. **Header**: Month/year with prev/next arrows, period picker
2. **Income Card**: Expected vs actual income
3. **Spending Card**:
   - Donut chart visualization
   - Legend with category colors
4. **Category List**:
   - Category row: Name, budgeted, actual, available
   - Progress bar showing usage
   - Overage highlighted in red

**Components**:
- `DonutChart`: D3.js ring chart with segments
- `BudgetCategoryRow`: Inline editable budget amount

**States**:
- Loading: Skeleton chart and rows
- Empty: "Set up your budget categories"
- Over budget: Red highlight on affected categories

**UI Parity Checklist**:
- [ ] Donut chart has hover interaction (shows category detail)
- [ ] Category rows have progress bar
- [ ] "Over budget" badge on exceeded categories
- [ ] Month picker is modal

---

### Import Wizard (`/imports/:id/*`)

**Multi-step flow**:

**Step 1: Upload** (`/imports/:id/upload`)
- File drop zone
- CSV preview (first 5 rows)
- Column separator select

**Step 2: Configuration** (`/imports/:id/configuration`)
- Column mapping grid
- Date format select
- Number format select
- Signage convention toggle

**Step 3: Clean** (`/imports/:id/clean`)
- Data table with all rows
- Invalid rows highlighted
- Edit cells inline
- Delete row button

**Step 4: Confirm** (`/imports/:id/confirm`)
- Summary statistics
- Duplicate warnings
- Confirm button

**States**:
- Step indicator showing progress
- Back button to previous step
- Cancel to abandon import

**UI Parity Checklist**:
- [ ] Step indicator at top
- [ ] File drop zone has drag state
- [ ] Preview table scrolls horizontally
- [ ] Invalid cells have red border
- [ ] Summary shows counts by type

---

## Reusable UI Components

Based on existing ViewComponents and patterns:

### Design System (DS) Components
1. `DsButton` - Primary, secondary, destructive, outline, ghost, icon variants
2. `DsLink` - Button-styled links
3. `DsDialog` - Modal and drawer variants
4. `DsMenu` - Dropdown menu with items
5. `DsMenuItem` - Menu item (link, button, divider)
6. `DsTabs` - Tab navigation with panels
7. `DsAlert` - Info, success, warning, error banners
8. `DsDisclosure` - Collapsible content
9. `DsToggle` - Switch input
10. `DsTooltip` - Hover tooltip
11. `DsFilledIcon` - Icon with background

### Domain Components
1. `AccountCard` - Account summary with sparkline
2. `TransactionRow` - Transaction list item
3. `CategoryPill` - Category label with color
4. `MoneyDisplay` - Formatted currency amount
5. `TrendIndicator` - Up/down arrow with percentage
6. `DatePicker` - Date input with calendar
7. `TimeSeriesChart` - Line chart with gradient (D3.js)
8. `DonutChart` - Ring chart for budgets (D3.js)
9. `SankeyChart` - Flow diagram for cashflow (D3.js)
10. `Sparkline` - Mini line chart for account cards

### Form Components
1. `FormField` - Input wrapper with label/error
2. `MoneyInput` - Currency-formatted number input
3. `CategorySelect` - Hierarchical category picker
4. `MerchantSelect` - Combobox with create option
5. `TagSelect` - Multi-select tags
6. `AccountSelect` - Account dropdown
7. `DateRangeSelect` - Start/end date picker
