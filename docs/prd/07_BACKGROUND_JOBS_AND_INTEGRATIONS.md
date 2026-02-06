# Background Jobs and Integrations

## Background Job System

### Queue Configuration (BullMQ)

| Queue | Concurrency | Purpose |
|-------|-------------|---------|
| `scheduled` | 10 | Cron-scheduled jobs |
| `high_priority` | 4 | Critical jobs (emails) |
| `medium_priority` | 2 | Sync and recalculation jobs |
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
// Triggers: Manual sync, auto-sync on login
// Queue: medium_priority

interface SyncJobPayload {
  syncableType: 'Account' | 'Family';
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
//    - Family: Sync all accounts
// 4. Update status to completed or failed
// 5. Broadcast completion event via WebSocket

// Retry: 3 attempts with exponential backoff
// Timeout: 5 minutes
```

#### RuleApplicationJob
```typescript
// Triggers: Rule created/updated, transactions created
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

**Purpose**: File uploads (profile images, exports)

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
- Exports: application/zip (generated)
