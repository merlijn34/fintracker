# User Flows

## Core User Flows

### 1. New User Registration Flow

```
1. User navigates to /register
2. User enters: email, password, first name, last name
3. System validates:
   - Email uniqueness
   - Password strength (8+ chars, upper, lower, number, special)
4. System creates:
   - Family record (with user's currency/locale defaults)
   - User record (role: admin for first user)
   - Session record
5. User redirected to /onboarding

<!-- Alternate: Registration with invitation
1. User clicks invitation link (/invitations/:token/accept)
2. System validates token not expired (3 days)
3. User enters password
4. System creates user in inviter's family
5. User redirected to onboarding -->
```

### 2. Login Flow

```
1. User navigates to /login
2. User enters email and password
3. System authenticates credentials
4. System creates Session record
5. Redirect to / (dashboard)

Error states:
- Invalid credentials: Show error, stay on form
- Account deactivated: Show specific error
```

### 3. Add Manual Account Flow

```
1. User clicks "Add Account" on accounts page
2. Modal opens with account type selector
3. User selects account type (e.g., "Checking Account")
4. Form loads with type-specific fields:
   - Depository: name, currency, balance
   - Credit Card: + APR, limit, minimum payment
   - Loan: + interest rate, term, initial balance
   - Investment: name, currency (balance from holdings)
   - Property: + address, year built, area
   - Vehicle: + make, model, year, mileage
5. User fills form and submits
6. System creates:
   - Accountable record (e.g., Depository)
   - Account record (linked via accountable_type/id)
   - Opening valuation Entry if balance provided
7. Modal closes, accounts list refreshes
8. User redirected to account detail page

Empty state: Show "No accounts yet" with prominent add button
```

### 4. Connect Bank Account (Plaid) Flow

```
1. User clicks "Connect Bank" button
2. System requests link_token from Plaid API
3. Plaid Link modal opens (external SDK)
4. User selects institution, enters credentials
5. User selects accounts to link
6. Plaid returns public_token
7. System exchanges for access_token (stored encrypted)
8. System creates:
   - PlaidItem record
   - PlaidAccount records for each selected account
   - Account records linked to PlaidAccounts
9. System queues initial SyncJob
10. User sees "Syncing..." status
11. Webhook or polling detects sync complete
12. Accounts appear with transactions

Error states:
- User cancels Plaid: Close modal, no changes
- Institution requires update: Store status="requires_update", show re-auth prompt
- Sync fails: Show error, allow retry
```

### 5. Transaction List & Filter Flow

```
1. User navigates to /transactions
2. System loads transactions with default filters:
   - Date range: user.default_period (e.g., last_30_days)
   - All accounts, all categories
3. Transaction list displays with:
   - Date grouped sections
   - Amount (formatted with currency)
   - Name/merchant
   - Category pill
   - Account name
4. User applies filters:
   a. Click date picker -> select custom range
   b. Click category filter -> select categories
   c. Click account filter -> select accounts
   d. Enter search text -> filters by name
5. List updates via Turbo (or Vue reactivity)
6. Summary bar shows: total income, total expenses, net

Bulk operations:
1. User clicks checkbox on row
2. Bulk action bar appears
3. User selects more rows (or "select all on page")
4. User clicks "Categorize" -> category picker
5. Confirm -> all selected transactions updated
```

### 6. Create/Edit Transaction Flow

```
1. User clicks "Add Transaction" or clicks existing row
2. Modal opens with form:
   - Account (select)
   - Date (date picker)
   - Name (text input)
   - Amount (money input)
   - Category (searchable select with hierarchy)
   - Merchant (combobox with create option)
   - Tags (multi-select with create option)
   - Kind (dropdown: standard, one-time, etc.)
   - Notes (textarea)
3. User fills form
4. System validates:
   - Account required
   - Date required
   - Amount required (non-zero)
   - Name required
5. System creates/updates:
   - Entry record
   - Transaction record (entryable)
6. Modal closes
7. Account balance recalculated (background)
8. Transaction list refreshes

Edit specific:
- If transaction has locked_attributes (from Plaid), those fields are disabled
```

### 7. Budget Management Flow

```
1. User navigates to /budgets (redirects to current month)
2. System creates Budget record if not exists
3. Page displays:
   - Header with month navigation
   - Income section: expected vs actual
   - Donut chart of spending by category
   - Category rows: budgeted, actual, available
4. User clicks category row
5. Inline edit or modal for budgeted amount
6. User saves
7. Donut chart and totals update
8. Overage categories highlighted in red

Empty state: "Set up your first budget" with category suggestions
```

### 8. Import CSV Flow

```
Step 1: Upload
1. User navigates to /imports/new
2. User selects import type (Transaction, Trade, Account, Mint)
3. User uploads CSV file
4. System parses CSV, creates Import and ImportRow records
5. Redirect to configuration step

Step 2: Configuration
1. User sees column mapping interface
2. System auto-maps common headers (Date, Amount, Description)
3. User adjusts mappings via dropdowns
4. User sets:
   - Date format (from presets)
   - Number format
   - Signage convention
5. User clicks "Continue"

Step 3: Clean/Review
1. System displays parsed rows in table
2. Invalid rows highlighted
3. User can edit individual rows
4. User can delete rows
5. User clicks "Continue"

Step 4: Confirm
1. System shows summary:
   - Total rows to import
   - New categories/tags to create
   - Potential duplicates flagged
2. User confirms
3. System queues ImportJob
4. User sees progress indicator
5. Redirect to transactions page on complete

Revert flow:
1. User navigates to /imports
2. User finds import, clicks "Revert"
3. Confirm dialog
4. System queues RevertImportJob
5. All entries with import_id deleted
```

### 9. AI Chat Flow

```
1. User opens AI sidebar (right sidebar)
2. User types question (e.g., "What did I spend on food last month?")
3. User presses send
4. System creates:
   - Chat record (if new conversation)
   - Message record (type: user_message)
5. System queues CreateChatResponseJob
6. Job sends to OpenAI with function definitions
7. If OpenAI requests function call:
   a. System executes function (e.g., get_transactions)
   b. Returns result to OpenAI
   c. OpenAI generates final response
8. Response streams to UI
9. Message record created (type: assistant_message)
10. UI updates with formatted response

Error handling:
- API error: Show retry button
- Rate limit: Show "Try again later"
- No API key configured: Show setup prompt
```

### 10. Rules Engine Flow

```
Create rule:
1. User navigates to /rules/new
2. User defines conditions:
   - Click "Add condition"
   - Select field (name, merchant, amount)
   - Select operator (contains, equals, greater than)
   - Enter value
   - Add more conditions with AND/OR
3. User defines actions:
   - Select action type (set_category, add_tags, set_merchant)
   - Select target value
4. User saves rule
5. System shows preview: "This rule will affect X transactions"

Apply rule:
1. User clicks "Apply" on rule
2. Confirm dialog shows affected count
3. System queues RuleJob
4. Matching transactions updated
5. Success notification

Auto-apply:
- When new transactions sync or import
- System evaluates active rules
- Matching transactions get rule actions applied
```
