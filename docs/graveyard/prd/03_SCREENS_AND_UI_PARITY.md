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

We use **shadcn-nuxt** components in the PWA, generated into `apps/pwa/components/ui` (style: **new-york**). The Nuxt module is configured with:
- `componentDir: "@/components/ui"`
- `prefix: ""` (so components are used as `<Button />`, `<Card />`, etc.)

### UI Primitives (shadcn-nuxt inventory in this repo)

These are the reusable primitives we should build screens with first (and only introduce custom wrappers when there’s a clear reuse/consistency win):

- **Button**: `Button` (+ variants via `buttonVariants`)
- **Card**: `Card`, `CardHeader`, `CardTitle`, `CardDescription`, `CardContent`, `CardFooter`
- **Input**: `Input`
- **Label**: `Label`
- **Checkbox**: `Checkbox`
- **Dropdown Menu**:
  `DropdownMenu`, `DropdownMenuTrigger`, `DropdownMenuContent`, `DropdownMenuItem`,
  `DropdownMenuLabel`, `DropdownMenuSeparator`, `DropdownMenuGroup`,
  `DropdownMenuCheckboxItem`, `DropdownMenuRadioGroup`, `DropdownMenuRadioItem`,
  `DropdownMenuSub`, `DropdownMenuSubTrigger`, `DropdownMenuSubContent`, `DropdownMenuShortcut`
- **Sheet (Drawer)**:
  `Sheet`, `SheetTrigger`, `SheetContent`, `SheetHeader`, `SheetTitle`, `SheetDescription`,
  `SheetFooter`, `SheetClose`
- **Tooltip**: `TooltipProvider`, `Tooltip`, `TooltipTrigger`, `TooltipContent`
- **Avatar**: `Avatar`, `AvatarImage`, `AvatarFallback`
- **Separator**: `Separator`
- **Skeleton**: `Skeleton`
- **Form (vee-validate helpers)**:
  `FormItem`, `FormLabel`, `FormControl`, `FormDescription`, `FormMessage`

Also present in `@/components/ui` as shared UI building blocks:
- **Sidebar** (app shell primitives): `SidebarProvider`, `SidebarTrigger`, plus the `Sidebar*` components under `@/components/ui/sidebar`
- **Chart helpers** (Unovis): `ChartTooltip`, `ChartCrosshair`, `ChartLegend`, etc. under `@/components/ui/chart`

### Migration mapping (Legacy DS names → shadcn-nuxt)

When translating PRD references like `Ds*` or “DS::Tabs”, use the following mapping:

| Legacy PRD name | Use in PWA (shadcn-nuxt) |
|---|---|
| `DsButton` | `Button` (variants: `default`, `secondary`, `destructive`, `outline`, `ghost`, `link`) |
| `DsLink` | `Button variant="link"` or `NuxtLink` styled with `buttonVariants({ variant: "link" })` |
| `DsMenu` / `DsMenuItem` | `DropdownMenu` + `DropdownMenuItem` / `DropdownMenuSeparator` / etc. |
| `DsDialog` | Prefer `Sheet` for drawers; add a `Dialog` primitive when true modal behavior is needed |
| `DsTabs` / `DS::Tabs` | Add/use a `Tabs` primitive when needed (not currently in `@/components/ui`) |
| `DsAlert` | Add/use an `Alert` primitive when needed (not currently in `@/components/ui`) |
| `DsDisclosure` | Prefer native `<details>` where appropriate; otherwise add `Accordion/Collapsible` when needed |
| `DsToggle` | Add/use a `Switch/Toggle` primitive when needed (not currently in `@/components/ui`) |
| `DsTooltip` | `Tooltip` (`TooltipProvider` + `TooltipTrigger` + `TooltipContent`) |
| `DsFilledIcon` | Custom pattern (icon + background), built with utility classes; promote to a component only if repeated |

### Domain Components (custom, built on shadcn primitives)

These remain app-specific, but should be composed from the primitives above:

1. `AccountCard` — Card layout + actions menu + sparkline
2. `TransactionRow` — Row layout + category pill + amount formatting + row actions
3. `CategoryPill` — Category label with color
4. `MoneyDisplay` — Currency formatting and alignment rules
5. `TrendIndicator` — Up/down arrow + percentage formatting
6. `DatePicker` — Date input UX (requires adding a calendar primitive when implemented)
7. `TimeSeriesChart` — Chart container + tooltip/crosshair helpers + loading/empty states
8. `DonutChart` — Budget ring chart + tooltip/legend
9. `SankeyChart` — Cashflow visualization + hover states
10. `Sparkline` — Mini chart for account rows/cards

### Form Components (custom, built on shadcn + vee-validate)

Prefer composing form UI using `@/components/ui/form` building blocks plus primitives like `Input`, `Label`, `Checkbox`, and `Button`:

1. `MoneyInput` — Currency-formatted numeric input (wraps `Input`)
2. `CategorySelect` — Hierarchical selection UI (will need a combobox/select primitive when implemented)
3. `MerchantSelect` — Searchable combobox with “create” option (will need a combobox primitive when implemented)
4. `TagSelect` — Multi-select tags UI (will need a multi-select primitive/pattern when implemented)
5. `AccountSelect` — Account dropdown/select (will need a select primitive when implemented)
6. `DateRangeSelect` — Start/end date picker (will need a calendar/date-range primitive when implemented)
