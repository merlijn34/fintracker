# Background Jobs and Integrations

## Background Job System

### Queue Configuration (BullMQ)

| Queue | Concurrency | Purpose |
|-------|-------------|---------|
| `scheduled` | 10 | Cron-scheduled jobs |
| `high_priority` | 4 | Critical jobs (emails) |
| `medium_priority` | 2 | Sync and import jobs |
| `low_priority` | 1 | Data cleanup |
| `default` | 1 | General background work |

### Scheduled Jobs (Cron)

| Job | Schedule | Purpose |
|-----|----------|---------|
| `ImportMarketDataJob` | `0 17 * * 1-5` (5 PM EST weekdays) | Fetch stock prices and exchange rates |
| `SyncCleanerJob` | `0 * * * *` (hourly) | Clean up stale sync records |
| `SecurityHealthCheckJob` | `0 2 * * 1-5` (2 AM EST weekdays) | Verify security price availability |

### Job Definitions

#### SyncJob
```typescript
// Triggers: Manual sync, Plaid webhook, auto-sync on login
// Queue: medium_priority

interface SyncJobPayload {
  syncableType: 'Account' | 'PlaidItem' | 'Family';
  syncableId: string;
  windowStartDate?: string;
  windowEndDate?: string;
  parentSyncId?: string;
}

// Process:
// 1. Create Sync record with status: pending
// 2. Update status to syncing
// 3. Execute sync based on type:
//    - Account: Calculate balances forward/reverse
//    - PlaidItem: Fetch transactions from Plaid API
//    - Family: Sync all accounts
// 4. Update status to completed or failed
// 5. Broadcast completion event via WebSocket

// Retry: 3 attempts with exponential backoff
// Timeout: 5 minutes
```

#### ImportJob
```typescript
// Triggers: User publishes import
// Queue: medium_priority

interface ImportJobPayload {
  importId: string;
}

// Process:
// 1. Load Import and ImportRows
// 2. For each row:
//    a. Parse according to configuration
//    b. Create Entry and Transaction/Trade
//    c. Apply category/tag mappings
// 3. Queue balance recalculation
// 4. Update Import status to complete

// Retry: 1 attempt (user can retry manually)
// Timeout: 10 minutes
```

#### RevertImportJob
```typescript
// Triggers: User reverts import
// Queue: medium_priority

interface RevertImportJobPayload {
  importId: string;
}

// Process:
// 1. Find all Entries with import_id
// 2. Delete entries in batches
// 3. Queue balance recalculation for affected accounts
// 4. Update Import status

// Retry: 1 attempt
// Timeout: 10 minutes
```

#### CreateChatResponseJob
```typescript
// Triggers: User sends chat message
// Queue: default

interface CreateChatResponseJobPayload {
  chatId: string;
  messageId: string;
}

// Process:
// 1. Load chat and message history
// 2. Build OpenAI request with function definitions
// 3. Send request with streaming
// 4. If function call requested:
//    a. Execute function (get_accounts, get_transactions, etc.)
//    b. Return result to OpenAI
//    c. Get final response
// 5. Save assistant message
// 6. Broadcast response via WebSocket

// Retry: 2 attempts
// Timeout: 2 minutes
```

#### RuleApplicationJob
```typescript
// Triggers: Rule created/updated, sync complete
// Queue: low_priority

interface RuleApplicationJobPayload {
  ruleId: string;
  transactionIds?: string[]; // If specific transactions, otherwise all matching
}

// Process:
// 1. Load rule with conditions and actions
// 2. Find matching transactions
// 3. Apply actions to each transaction
// 4. Log applied count

// Retry: 1 attempt
// Timeout: 5 minutes
```

#### FamilyDataExportJob
```typescript
// Triggers: User requests data export
// Queue: low_priority

interface FamilyDataExportJobPayload {
  exportId: string;
}

// Process:
// 1. Update FamilyExport status to processing
// 2. Export all family data to JSON files
// 3. Create ZIP archive
// 4. Upload to storage
// 5. Update status to completed with download URL

// Retry: 2 attempts
// Timeout: 15 minutes
```

## External Integrations

### Plaid Integration

**Purpose**: Bank account aggregation and transaction syncing

**Authentication**: Client ID + Secret (environment variables)

**API Calls**:

1. **Create Link Token**
   ```
   POST /link/token/create
   Headers: PLAID-CLIENT-ID, PLAID-SECRET
   Body: { user, client_name, products, country_codes, language }
   Response: { link_token, expiration }
   ```

2. **Exchange Public Token**
   ```
   POST /item/public_token/exchange
   Body: { public_token }
   Response: { access_token, item_id }
   ```

3. **Get Transactions**
   ```
   POST /transactions/sync
   Body: { access_token, cursor }
   Response: { added, modified, removed, next_cursor, has_more }
   ```

4. **Get Accounts**
   ```
   POST /accounts/get
   Body: { access_token }
   Response: { accounts, item }
   ```

5. **Get Investment Holdings**
   ```
   POST /investments/holdings/get
   Body: { access_token }
   Response: { accounts, holdings, securities }
   ```

6. **Get Liabilities**
   ```
   POST /liabilities/get
   Body: { access_token }
   Response: { accounts, credit, student, mortgage }
   ```

**Webhooks**:

| Event | Action |
|-------|--------|
| `TRANSACTIONS.SYNC_UPDATES_AVAILABLE` | Queue SyncJob for PlaidItem |
| `TRANSACTIONS.HISTORICAL_UPDATE` | Full historical sync |
| `ITEM.ERROR` | Update PlaidItem status to requires_update |
| `ITEM.PENDING_EXPIRATION` | Notify user to re-authenticate |

**Retry/Failure**:
- Rate limit: 429 response, exponential backoff
- Auth error: Mark item as requires_update
- Transient error: Retry up to 3 times

### OpenAI Integration

**Purpose**: AI chat assistant with financial analysis

**Authentication**: API Key (environment variable)

**API Calls**:

1. **Chat Completion with Functions**
   ```
   POST /v1/chat/completions
   Headers: Authorization: Bearer {api_key}
   Body: {
     model: "gpt-4",
     messages: [...],
     tools: [...],
     stream: true
   }
   ```

**Function Definitions**:

```typescript
const functions = [
  {
    name: "get_accounts",
    description: "Get list of user's financial accounts with balances",
    parameters: {
      type: "object",
      properties: {
        classification: { type: "string", enum: ["asset", "liability"] }
      }
    }
  },
  {
    name: "get_balance_sheet",
    description: "Get current balance sheet with assets, liabilities, and net worth",
    parameters: {
      type: "object",
      properties: {
        date: { type: "string", format: "date" }
      }
    }
  },
  {
    name: "get_income_statement",
    description: "Get income and expenses for a time period",
    parameters: {
      type: "object",
      properties: {
        startDate: { type: "string", format: "date" },
        endDate: { type: "string", format: "date" }
      },
      required: ["startDate", "endDate"]
    }
  },
  {
    name: "get_transactions",
    description: "Search and filter transactions",
    parameters: {
      type: "object",
      properties: {
        startDate: { type: "string", format: "date" },
        endDate: { type: "string", format: "date" },
        categoryId: { type: "string" },
        minAmount: { type: "number" },
        maxAmount: { type: "number" },
        search: { type: "string" },
        limit: { type: "integer", maximum: 100 }
      }
    }
  }
];
```

**Retry/Failure**:
- Rate limit: Exponential backoff with jitter
- Token limit exceeded: Truncate history, retry
- API error: Retry up to 2 times, then show error to user

### Synth Finance Integration

**Purpose**: Market data (exchange rates, security prices)

**Authentication**: API Key (environment variable or Settings table)

**API Calls**:

1. **Get Exchange Rates**
   ```
   GET /rates?base={from}&symbols={to}&date={date}
   Headers: Authorization: Bearer {api_key}
   Response: { rates: { [currency]: rate } }
   ```

2. **Get Security Price**
   ```
   GET /prices?ticker={ticker}&exchange={mic}&date={date}
   Headers: Authorization: Bearer {api_key}
   Response: { price, currency }
   ```

**Scheduled Import**:
- Run daily at 5 PM EST (after market close)
- Fetch rates for all unique currency pairs in accounts
- Fetch prices for all securities with holdings
- Store in exchange_rates and security_prices tables

**Retry/Failure**:
- Missing data: Mark security as offline, retry next day
- API error: Retry 3 times, log failure

### Stripe Integration (Optional)

**Purpose**: Subscription billing (managed mode)

**Authentication**: API Key + Webhook Secret

**API Calls**:

1. **Create Checkout Session**
   ```
   POST /v1/checkout/sessions
   Body: { customer, line_items, mode: "subscription", success_url, cancel_url }
   Response: { id, url }
   ```

2. **Create Customer Portal Session**
   ```
   POST /v1/billing_portal/sessions
   Body: { customer, return_url }
   Response: { url }
   ```

**Webhooks**:

| Event | Action |
|-------|--------|
| `customer.subscription.created` | Create Subscription record |
| `customer.subscription.updated` | Update Subscription record |
| `customer.subscription.deleted` | Mark Subscription as canceled |
| `invoice.payment_failed` | Notify user |

## File Storage

### S3-Compatible Storage

**Purpose**: File uploads (profile images, exports, import files)

**Configuration**:
```typescript
interface StorageConfig {
  service: 'disk' | 'amazon' | 'cloudflare';
  bucket?: string;
  region?: string;
  endpoint?: string;
  accessKeyId?: string;
  secretAccessKey?: string;
}
```

**Operations**:
- Upload file with content type
- Generate signed download URL (1 hour expiry)
- Delete file

**File Types**:
- Profile images: JPEG, PNG (max 5MB)
- CSV imports: text/csv (max 10MB)
- Exports: application/zip (generated)
