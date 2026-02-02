# Data Model

## Entity Relationship Overview

```
Family (1) ──────┬──── (N) User
                 │
                 ├──── (N) Account ──── (N) Entry ──┬── Transaction
                 │           │                      ├── Trade
                 │           ├── (N) Balance        └── Valuation
                 │           └── (N) Holding
                 │
                 ├──── (N) Category (hierarchical)
                 ├──── (N) Tag ──── (N) Tagging (polymorphic)
                 ├──── (N) Merchant
                 ├──── (N) Budget ──── (N) BudgetCategory
                 ├──── (N) Rule ──┬── (N) RuleCondition
                 │                └── (N) RuleAction
                 ├──── (N) Import ──── (N) ImportRow
                 │                └── (N) ImportMapping
                 ├──── (N) PlaidItem ──── (N) PlaidAccount
                 └──── (N) Invitation

User (1) ──┬── (N) Session
           ├── (N) Chat ──── (N) Message ──── (N) ToolCall
           └── (N) ApiKey

Security (global) ──── (N) SecurityPrice
                  └── (N) Holding
                  └── (N) Trade

ExchangeRate (global) - currency conversion rates
```

## Entity Definitions

### Family
```typescript
interface Family {
  id: UUID;                    // Primary key
  name: string | null;         // Family name
  currency: string;            // Base currency (default: "USD")
  locale: string;              // Locale code (default: "en")
  country: string;             // Country code (default: "US")
  timezone: string | null;     // Timezone identifier
  dateFormat: string;          // Date format pattern (default: "%m-%d-%Y")
  stripeCustomerId: string | null;  // For billing (optional)
  dataEnrichmentEnabled: boolean;   // Enable third-party enrichment
  earlyAccess: boolean;        // Beta features flag
  autoSyncOnLogin: boolean;    // Auto-sync accounts on login
  latestSyncActivityAt: Date;  // Last sync activity timestamp
  latestSyncCompletedAt: Date; // Last successful sync
  createdAt: Date;
  updatedAt: Date;
}
```

### User
```typescript
interface User {
  id: UUID;
  familyId: UUID;              // Foreign key to Family
  email: string;               // Unique email
  passwordDigest: string;      // bcrypt hash
  firstName: string | null;
  lastName: string | null;
  role: 'member' | 'admin' | 'super_admin';
  active: boolean;             // Soft delete flag
  onboardedAt: Date | null;    // Completed onboarding
  theme: 'light' | 'dark' | 'system';
  showSidebar: boolean;
  showAiSidebar: boolean;
  aiEnabled: boolean;
  defaultPeriod: string;       // Default date range
  rulePromptsDisabled: boolean;
  goals: string[];             // User goals array
  // MFA fields
  otpSecret: string | null;    // TOTP secret (encrypted)
  otpRequired: boolean;
  otpBackupCodes: string[];    // Single-use backup codes
  // Email change
  unconfirmedEmail: string | null;
  // Tracking
  lastViewedChatId: UUID | null;
  setOnboardingPreferencesAt: Date | null;
  setOnboardingGoalsAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### Account
```typescript
interface Account {
  id: UUID;
  familyId: UUID;
  name: string;
  subtype: string | null;      // Account subtype (checking, savings, etc.)
  balance: Decimal;            // Current balance (19,4 precision)
  cashBalance: Decimal;        // Cash portion of balance
  currency: string;
  classification: 'asset' | 'liability';  // Computed from accountableType
  status: 'active' | 'draft' | 'disabled' | 'pending_deletion';
  accountableType: string;     // Polymorphic type (Depository, Investment, etc.)
  accountableId: UUID;         // Polymorphic ID
  importId: UUID | null;       // Link to import that created this
  plaidAccountId: UUID | null; // Link to Plaid account
  lockedAttributes: JsonB;     // Fields locked from editing
  createdAt: Date;
  updatedAt: Date;
}

// Account subtypes (separate tables)
interface Depository { id: UUID; lockedAttributes: JsonB; }
interface CreditCard {
  id: UUID;
  availableCredit: Decimal | null;
  minimumPayment: Decimal | null;
  apr: Decimal | null;
  expirationDate: Date | null;
  annualFee: Decimal | null;
  lockedAttributes: JsonB;
}
interface Investment { id: UUID; lockedAttributes: JsonB; }
interface Loan {
  id: UUID;
  rateType: string | null;     // fixed, variable
  interestRate: Decimal | null;
  termMonths: number | null;
  initialBalance: Decimal | null;
  lockedAttributes: JsonB;
}
interface Property {
  id: UUID;
  yearBuilt: number | null;
  areaValue: number | null;
  areaUnit: string | null;
  lockedAttributes: JsonB;
}
interface Vehicle {
  id: UUID;
  make: string | null;
  model: string | null;
  year: number | null;
  mileageValue: number | null;
  mileageUnit: string | null;
  lockedAttributes: JsonB;
}
interface Crypto { id: UUID; lockedAttributes: JsonB; }
interface OtherAsset { id: UUID; lockedAttributes: JsonB; }
interface OtherLiability { id: UUID; lockedAttributes: JsonB; }
```

### Entry (Base for all financial entries)
```typescript
interface Entry {
  id: UUID;
  accountId: UUID;
  entryableType: 'Transaction' | 'Trade' | 'Valuation';
  entryableId: UUID;
  amount: Decimal;             // Signed amount (negative = outflow)
  currency: string;
  date: Date;
  name: string;
  notes: string | null;
  excluded: boolean;           // Exclude from calculations
  plaidId: string | null;      // External Plaid ID
  importId: UUID | null;       // Link to import
  lockedAttributes: JsonB;
  createdAt: Date;
  updatedAt: Date;
}
```

### Transaction
```typescript
interface Transaction {
  id: UUID;
  categoryId: UUID | null;
  merchantId: UUID | null;
  kind: 'standard' | 'funds_movement' | 'cc_payment' | 'loan_payment' | 'one_time';
  lockedAttributes: JsonB;
  createdAt: Date;
  updatedAt: Date;
}
```

### Trade
```typescript
interface Trade {
  id: UUID;
  securityId: UUID;
  qty: Decimal;                // Quantity (positive=buy, negative=sell)
  price: Decimal;              // Price per unit
  currency: string;
  lockedAttributes: JsonB;
  createdAt: Date;
  updatedAt: Date;
}
```

### Valuation
```typescript
interface Valuation {
  id: UUID;
  kind: 'reconciliation' | 'opening_anchor' | 'current_anchor';
  lockedAttributes: JsonB;
  createdAt: Date;
  updatedAt: Date;
}
```

### Balance
```typescript
interface Balance {
  id: UUID;
  accountId: UUID;
  date: Date;
  currency: string;
  // Starting balances
  startCashBalance: Decimal;
  startNonCashBalance: Decimal;
  startBalance: Decimal;       // Computed: cash + non-cash
  // Flows
  cashInflows: Decimal;
  cashOutflows: Decimal;
  nonCashInflows: Decimal;
  nonCashOutflows: Decimal;
  netMarketFlows: Decimal;
  // Adjustments
  cashAdjustments: Decimal;
  nonCashAdjustments: Decimal;
  // Factor
  flowsFactor: number;         // 1 or -1 for direction
  // Ending balances (computed)
  endCashBalance: Decimal;
  endNonCashBalance: Decimal;
  endBalance: Decimal;
  balance: Decimal;            // Legacy field
  cashBalance: Decimal;        // Legacy field
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (accountId, date, currency)
```

### Category
```typescript
interface Category {
  id: UUID;
  familyId: UUID;
  parentId: UUID | null;       // For subcategories
  name: string;
  color: string;               // Hex color
  lucideIcon: string;          // Icon name
  classification: 'income' | 'expense';
  createdAt: Date;
  updatedAt: Date;
}
```

### Tag
```typescript
interface Tag {
  id: UUID;
  familyId: UUID;
  name: string;
  color: string;               // Hex color
  createdAt: Date;
  updatedAt: Date;
}

interface Tagging {
  id: UUID;
  tagId: UUID;
  taggableType: string;        // 'Transaction'
  taggableId: UUID;
  createdAt: Date;
  updatedAt: Date;
}
```

### Merchant
```typescript
interface Merchant {
  id: UUID;
  familyId: UUID | null;       // null for provider merchants
  name: string;
  type: 'FamilyMerchant' | 'ProviderMerchant';
  color: string | null;
  logoUrl: string | null;
  websiteUrl: string | null;
  source: string | null;       // For provider merchants
  providerMerchantId: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### Budget
```typescript
interface Budget {
  id: UUID;
  familyId: UUID;
  startDate: Date;
  endDate: Date;
  budgetedSpending: Decimal | null;
  expectedIncome: Decimal | null;
  currency: string;
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (familyId, startDate, endDate)

interface BudgetCategory {
  id: UUID;
  budgetId: UUID;
  categoryId: UUID;
  budgetedSpending: Decimal;
  currency: string;
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (budgetId, categoryId)
```

### Holding
```typescript
interface Holding {
  id: UUID;
  accountId: UUID;
  securityId: UUID;
  date: Date;
  qty: Decimal;
  price: Decimal;
  amount: Decimal;             // qty * price
  currency: string;
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (accountId, securityId, date, currency)
```

### Security
```typescript
interface Security {
  id: UUID;
  ticker: string;
  name: string | null;
  countryCode: string | null;
  exchangeMic: string | null;
  exchangeAcronym: string | null;
  exchangeOperatingMic: string | null;
  logoUrl: string | null;
  offline: boolean;            // Temporarily unavailable
  failedFetchAt: Date | null;
  failedFetchCount: number;
  lastHealthCheckAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (upper(ticker), coalesce(upper(exchangeOperatingMic), ''))

interface SecurityPrice {
  id: UUID;
  securityId: UUID;
  date: Date;
  price: Decimal;
  currency: string;
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (securityId, date, currency)
```

### Transfer
```typescript
interface Transfer {
  id: UUID;
  inflowTransactionId: UUID;
  outflowTransactionId: UUID;
  status: 'pending' | 'confirmed';
  notes: string | null;
  createdAt: Date;
  updatedAt: Date;
}

interface RejectedTransfer {
  id: UUID;
  inflowTransactionId: UUID;
  outflowTransactionId: UUID;
  createdAt: Date;
  updatedAt: Date;
}
```

### Rule
```typescript
interface Rule {
  id: UUID;
  familyId: UUID;
  name: string | null;
  resourceType: string;        // 'Transaction'
  effectiveDate: Date | null;
  active: boolean;
  createdAt: Date;
  updatedAt: Date;
}

interface RuleCondition {
  id: UUID;
  ruleId: UUID | null;
  parentId: UUID | null;       // For nested conditions
  conditionType: string;       // 'compound', 'name', 'merchant', 'amount'
  operator: string;            // 'and', 'or', 'contains', 'equals', 'gt', 'lt'
  value: string | null;
  createdAt: Date;
  updatedAt: Date;
}

interface RuleAction {
  id: UUID;
  ruleId: UUID;
  actionType: string;          // 'set_category', 'set_merchant', 'add_tag'
  value: string | null;        // UUID or value
  createdAt: Date;
  updatedAt: Date;
}
```

### Import
```typescript
interface Import {
  id: UUID;
  familyId: UUID;
  accountId: UUID | null;      // Target account (optional)
  type: 'TransactionImport' | 'TradeImport' | 'AccountImport' | 'MintImport';
  status: 'pending' | 'complete' | 'importing' | 'reverting' | 'failed';
  colSep: string;              // Column separator
  // Column mappings
  columnMappings: JsonB | null;
  dateColLabel: string | null;
  amountColLabel: string | null;
  nameColLabel: string | null;
  categoryColLabel: string | null;
  tagsColLabel: string | null;
  accountColLabel: string | null;
  qtyColLabel: string | null;
  tickerColLabel: string | null;
  priceColLabel: string | null;
  entityTypeColLabel: string | null;
  notesColLabel: string | null;
  currencyColLabel: string | null;
  exchangeOperatingMicColLabel: string | null;
  // Format settings
  dateFormat: string;
  numberFormat: string | null;
  signageConvention: 'inflows_positive' | 'inflows_negative';
  amountTypeStrategy: 'signed_amount' | 'type_column';
  amountTypeInflowValue: string | null;
  // Raw data
  rawFileStr: string | null;
  normalizedCsvStr: string | null;
  error: string | null;
  createdAt: Date;
  updatedAt: Date;
}

interface ImportRow {
  id: UUID;
  importId: UUID;
  account: string | null;
  date: string | null;
  qty: string | null;
  ticker: string | null;
  price: string | null;
  amount: string | null;
  currency: string | null;
  name: string | null;
  category: string | null;
  tags: string | null;
  entityType: string | null;
  notes: string | null;
  exchangeOperatingMic: string | null;
  createdAt: Date;
  updatedAt: Date;
}

interface ImportMapping {
  id: UUID;
  importId: UUID;
  type: string;                // 'CategoryMapping', 'TagMapping', 'AccountMapping'
  key: string | null;
  value: string | null;
  createWhenEmpty: boolean;
  mappableType: string | null;
  mappableId: UUID | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### Plaid Integration
```typescript
interface PlaidItem {
  id: UUID;
  familyId: UUID;
  accessToken: string;         // Encrypted
  plaidId: string;             // Plaid item ID
  name: string | null;         // Institution name
  institutionId: string | null;
  institutionUrl: string | null;
  institutionColor: string | null;
  plaidRegion: 'us' | 'eu';
  status: 'good' | 'requires_update';
  nextCursor: string | null;   // For incremental sync
  availableProducts: string[];
  billedProducts: string[];
  scheduledForDeletion: boolean;
  rawPayload: JsonB;
  rawInstitutionPayload: JsonB;
  createdAt: Date;
  updatedAt: Date;
}

interface PlaidAccount {
  id: UUID;
  plaidItemId: UUID;
  plaidId: string;             // Plaid account ID
  plaidType: string;           // depository, credit, loan, investment
  plaidSubtype: string | null;
  currentBalance: Decimal | null;
  availableBalance: Decimal | null;
  currency: string;
  name: string;
  mask: string | null;         // Last 4 digits
  rawPayload: JsonB;
  rawTransactionsPayload: JsonB;
  rawInvestmentsPayload: JsonB;
  rawLiabilitiesPayload: JsonB;
  createdAt: Date;
  updatedAt: Date;
}
```

### Sync
```typescript
interface Sync {
  id: UUID;
  syncableType: string;        // 'Account', 'PlaidItem', 'Family'
  syncableId: UUID;
  parentId: UUID | null;       // For child syncs
  status: 'pending' | 'syncing' | 'completed' | 'failed' | 'stale';
  error: string | null;
  data: JsonB | null;
  windowStartDate: Date | null;
  windowEndDate: Date | null;
  pendingAt: Date | null;
  syncingAt: Date | null;
  completedAt: Date | null;
  failedAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### Chat & AI
```typescript
interface Chat {
  id: UUID;
  userId: UUID;
  title: string;
  instructions: string | null;
  error: JsonB | null;
  latestAssistantResponseId: string | null;
  createdAt: Date;
  updatedAt: Date;
}

interface Message {
  id: UUID;
  chatId: UUID;
  type: 'user_message' | 'assistant_message' | 'developer_message';
  status: 'complete' | 'streaming';
  content: string | null;
  aiModel: string | null;
  providerId: string | null;
  debug: boolean;
  reasoning: boolean;
  createdAt: Date;
  updatedAt: Date;
}

interface ToolCall {
  id: UUID;
  messageId: UUID;
  providerId: string;
  providerCallId: string | null;
  type: string;                // 'function'
  functionName: string | null;
  functionArguments: JsonB | null;
  functionResult: JsonB | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### Session & Auth
```typescript
interface Session {
  id: UUID;
  userId: UUID;
  userAgent: string | null;
  ipAddress: string | null;
  activeImpersonatorSessionId: UUID | null;
  subscribedAt: Date | null;
  prevTransactionPageParams: JsonB;
  data: JsonB;
  createdAt: Date;
  updatedAt: Date;
}

interface ApiKey {
  id: UUID;
  userId: UUID;
  name: string | null;
  displayKey: string;          // Shown to user (partial)
  scopes: string[];            // ['read', 'read_write']
  source: 'web' | 'mobile';
  lastUsedAt: Date | null;
  expiresAt: Date | null;
  revokedAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### Supporting Entities
```typescript
interface ExchangeRate {
  id: UUID;
  fromCurrency: string;
  toCurrency: string;
  rate: Decimal;
  date: Date;
  createdAt: Date;
  updatedAt: Date;
}
// Unique constraint: (fromCurrency, toCurrency, date)

interface Address {
  id: UUID;
  addressableType: string;
  addressableId: UUID;
  line1: string | null;
  line2: string | null;
  county: string | null;
  locality: string | null;
  region: string | null;
  country: string | null;
  postalCode: number | null;
  createdAt: Date;
  updatedAt: Date;
}

interface Invitation {
  id: UUID;
  familyId: UUID;
  inviterId: UUID;
  email: string;
  role: 'member' | 'admin';
  token: string;
  expiresAt: Date;
  acceptedAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}

interface Subscription {
  id: UUID;
  familyId: UUID;
  stripeId: string | null;
  status: string;
  amount: Decimal | null;
  currency: string | null;
  interval: string | null;
  currentPeriodEndsAt: Date | null;
  trialEndsAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}

interface FamilyExport {
  id: UUID;
  familyId: UUID;
  status: 'pending' | 'processing' | 'completed' | 'failed';
  createdAt: Date;
  updatedAt: Date;
}

interface DataEnrichment {
  id: UUID;
  enrichableType: string;
  enrichableId: UUID;
  source: string | null;
  attributeName: string | null;
  value: JsonB | null;
  metadata: JsonB | null;
  createdAt: Date;
  updatedAt: Date;
}

interface InviteCode {
  id: UUID;
  token: string;
  createdAt: Date;
  updatedAt: Date;
}

interface MobileDevice {
  id: UUID;
  userId: UUID;
  deviceId: string | null;
  deviceName: string | null;
  deviceType: string | null;
  osVersion: string | null;
  appVersion: string | null;
  lastSeenAt: Date | null;
  oauthApplicationId: number | null;
  createdAt: Date;
  updatedAt: Date;
}

interface Setting {
  id: number;
  var: string;
  value: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

## Key Indexes

```sql
-- Performance indexes (most critical)
CREATE INDEX idx_accounts_family_type ON accounts(family_id, accountable_type);
CREATE INDEX idx_entries_account_date ON entries(account_id, date);
CREATE INDEX idx_balances_account_date ON balances(account_id, date DESC);
CREATE INDEX idx_transactions_category ON transactions(category_id);
CREATE INDEX idx_holdings_account_security ON holdings(account_id, security_id);

-- Unique constraints
CREATE UNIQUE INDEX idx_balances_account_date_currency ON balances(account_id, date, currency);
CREATE UNIQUE INDEX idx_exchange_rates_unique ON exchange_rates(from_currency, to_currency, date);
CREATE UNIQUE INDEX idx_holdings_unique ON holdings(account_id, security_id, date, currency);
CREATE UNIQUE INDEX idx_users_email ON users(email);
```
