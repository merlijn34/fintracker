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

### 4. Transaction List & Filter Flow

```
1. User navigates to /transactions
2. System loads transactions with default filters:
   - Date range: user.default_period (e.g., last_30_days)
   - All accounts, all categories
3. Transaction list displays with:
   - Date grouped sections
   - Amount (formatted with currency)
   - Description
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

### 5. Create/Edit Transaction Flow

```
1. User clicks "Add Transaction" or clicks existing row
2. Modal opens with form:
   - Account (select)
   - Date (date picker)
   - Description (text input)
   - Amount (money input)
   - Category (searchable select with hierarchy)
   - Tags (multi-select with create option)
   - Kind (dropdown: standard, one-time, etc.)
   - Notes (textarea)
3. User fills form
4. System validates:
   - Account required
   - Date required
   - Amount required (non-zero)
   - Description required
5. System creates/updates:
   - Entry record
   - Transaction record (entryable)
6. Modal closes
7. Account balance recalculated (background)
8. Transaction list refreshes

```

### 6. Rules Engine Flow

```
Create rule:
1. User navigates to /rules/new
2. User defines conditions:
   - Click "Add condition"
   - Select field (name, amount)
   - Select operator (contains, equals, greater than)
   - Enter value
   - Add more conditions with AND/OR
3. User defines actions:
   - Select action type (set_category, add_tags)
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
- When new transactions are created
- System evaluates active rules
- Matching transactions get rule actions applied
```
