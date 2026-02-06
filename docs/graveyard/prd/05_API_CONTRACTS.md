# API Contracts

## API Design Principles

1. **RESTful Resources**: Standard CRUD endpoints with nested resources where logical
2. **JSON Responses**: All responses in JSON format
3. **JWT Authentication**: Bearer token in Authorization header
4. **Consistent Error Format**: Standardized error response structure
5. **Pagination**: Cursor-based or offset pagination with consistent format
6. **Filtering**: Query parameters for filtering and searching

## Authentication

### Register
```http
POST /api/v1/auth/signup
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "inviteCode": "optional-invite-code"
}

Response 201:
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "admin",
      "familyId": "550e8400-e29b-41d4-a716-446655440001"
    }
  }
}
```

### Login
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response 200:
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "user": { ... },
    "mfaRequired": false
  }
}

// If MFA required:
Response 200:
{
  "data": {
    "mfaRequired": true,
    "mfaToken": "temporary-mfa-token"
  }
}
```

### MFA Verify
```http
POST /api/v1/auth/mfa/verify
Content-Type: application/json

{
  "mfaToken": "temporary-mfa-token",
  "code": "123456"
}

Response 200:
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "user": { ... }
  }
}
```

### Refresh Token
```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}

Response 200:
{
  "data": {
    "accessToken": "new-access-token",
    "refreshToken": "new-refresh-token"
  }
}
```

## Standard Response Format

```typescript
// Success response
interface ApiResponse<T> {
  data: T;
  meta?: {
    pagination?: {
      page: number;
      perPage: number;
      totalPages: number;
      totalCount: number;
      hasNextPage: boolean;
      hasPrevPage: boolean;
    };
  };
}

// Error response
interface ApiError {
  error: {
    code: string;          // e.g., "VALIDATION_ERROR", "NOT_FOUND"
    message: string;       // Human-readable message
    details?: {
      field: string;
      message: string;
    }[];
  };
}
```

## Accounts

### List Accounts
```http
GET /api/v1/accounts
Authorization: Bearer {token}

Query params:
- classification: "asset" | "liability"
- accountableType: "Depository" | "Investment" | ...
- status: "active" | "disabled"

Response 200:
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Checking Account",
      "balance": "5000.00",
      "currency": "USD",
      "classification": "asset",
      "accountableType": "Depository",
      "subtype": "checking",
      "status": "active",
      "plaidConnected": false,
      "lastSyncedAt": null,
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ]
}
```

### Get Account
```http
GET /api/v1/accounts/:id
Authorization: Bearer {token}

Response 200:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Checking Account",
    "balance": "5000.00",
    "cashBalance": "5000.00",
    "currency": "USD",
    "classification": "asset",
    "accountableType": "Depository",
    "accountable": {
      "id": "550e8400-e29b-41d4-a716-446655440001"
    },
    "subtype": "checking",
    "status": "active",
    "plaidAccountId": null,
    "lockedAttributes": [],
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  }
}
```

### Create Account
```http
POST /api/v1/accounts
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Savings Account",
  "accountableType": "Depository",
  "subtype": "savings",
  "currency": "USD",
  "balance": "10000.00"
}

Response 201:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440002",
    "name": "Savings Account",
    ...
  }
}
```

### Update Account
```http
PATCH /api/v1/accounts/:id
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Primary Savings"
}

Response 200:
{
  "data": { ... }
}
```

### Delete Account
```http
DELETE /api/v1/accounts/:id
Authorization: Bearer {token}

Response 204 (No Content)
```

### Sync Account
```http
POST /api/v1/accounts/:id/sync
Authorization: Bearer {token}

Response 202:
{
  "data": {
    "syncId": "550e8400-e29b-41d4-a716-446655440003",
    "status": "pending"
  }
}
```

### Get Account Sparkline
```http
GET /api/v1/accounts/:id/sparkline
Authorization: Bearer {token}

Query params:
- period: "last_7_days" | "last_30_days" | "last_90_days"

Response 200:
{
  "data": {
    "points": [
      { "date": "2024-01-01", "value": "4500.00" },
      { "date": "2024-01-02", "value": "4600.00" },
      ...
    ]
  }
}
```

## Transactions

### List Transactions
```http
GET /api/v1/transactions
Authorization: Bearer {token}

Query params:
- page: number (default: 1)
- perPage: number (default: 25, max: 100)
- accountIds: string[] (comma-separated UUIDs)
- categoryIds: string[] (comma-separated UUIDs)
- merchantIds: string[] (comma-separated UUIDs)
- tagIds: string[] (comma-separated UUIDs)
- startDate: string (ISO date)
- endDate: string (ISO date)
- minAmount: number
- maxAmount: number
- search: string (searches name)
- kind: "standard" | "funds_movement" | "cc_payment" | "loan_payment" | "one_time"

Response 200:
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "entryId": "550e8400-e29b-41d4-a716-446655440001",
      "accountId": "550e8400-e29b-41d4-a716-446655440002",
      "date": "2024-01-15",
      "name": "Grocery Store",
      "amount": "-125.50",
      "currency": "USD",
      "category": {
        "id": "550e8400-e29b-41d4-a716-446655440003",
        "name": "Food & Drink",
        "color": "#22c55e",
        "icon": "utensils"
      },
      "merchant": {
        "id": "550e8400-e29b-41d4-a716-446655440004",
        "name": "Whole Foods",
        "logoUrl": null
      },
      "tags": [
        { "id": "...", "name": "groceries", "color": "#3b82f6" }
      ],
      "kind": "standard",
      "excluded": false,
      "notes": null,
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "perPage": 25,
      "totalPages": 4,
      "totalCount": 87,
      "hasNextPage": true,
      "hasPrevPage": false
    },
    "summary": {
      "totalIncome": "5000.00",
      "totalExpenses": "3250.75",
      "net": "1749.25"
    }
  }
}
```

### Get Transaction
```http
GET /api/v1/transactions/:id
Authorization: Bearer {token}

Response 200:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "entryId": "...",
    "accountId": "...",
    "account": {
      "id": "...",
      "name": "Checking Account"
    },
    "date": "2024-01-15",
    "name": "Grocery Store",
    "amount": "-125.50",
    "currency": "USD",
    "category": { ... },
    "merchant": { ... },
    "tags": [ ... ],
    "kind": "standard",
    "excluded": false,
    "notes": "Weekly groceries",
    "lockedAttributes": ["name", "amount"],
    "plaidId": "plaid_txn_123",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  }
}
```

### Create Transaction
```http
POST /api/v1/transactions
Authorization: Bearer {token}
Content-Type: application/json

{
  "accountId": "550e8400-e29b-41d4-a716-446655440000",
  "date": "2024-01-15",
  "name": "Coffee Shop",
  "amount": "-5.50",
  "categoryId": "550e8400-e29b-41d4-a716-446655440001",
  "merchantId": "550e8400-e29b-41d4-a716-446655440002",
  "tagIds": ["550e8400-e29b-41d4-a716-446655440003"],
  "kind": "standard",
  "notes": "Morning coffee"
}

Response 201:
{
  "data": { ... }
}
```

### Update Transaction
```http
PATCH /api/v1/transactions/:id
Authorization: Bearer {token}
Content-Type: application/json

{
  "categoryId": "new-category-id",
  "notes": "Updated notes"
}

Response 200:
{
  "data": { ... }
}
```

### Delete Transaction
```http
DELETE /api/v1/transactions/:id
Authorization: Bearer {token}

Response 204 (No Content)
```

### Bulk Update Transactions
```http
POST /api/v1/transactions/bulk-update
Authorization: Bearer {token}
Content-Type: application/json

{
  "transactionIds": ["id1", "id2", "id3"],
  "updates": {
    "categoryId": "new-category-id",
    "tagIds": ["tag1", "tag2"]
  }
}

Response 200:
{
  "data": {
    "updatedCount": 3
  }
}
```

### Bulk Delete Transactions
```http
POST /api/v1/transactions/bulk-delete
Authorization: Bearer {token}
Content-Type: application/json

{
  "transactionIds": ["id1", "id2", "id3"]
}

Response 200:
{
  "data": {
    "deletedCount": 3
  }
}
```

## Categories

### List Categories
```http
GET /api/v1/categories
Authorization: Bearer {token}

Response 200:
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Food & Drink",
      "color": "#22c55e",
      "icon": "utensils",
      "classification": "expense",
      "parentId": null,
      "subcategories": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440001",
          "name": "Restaurants",
          "color": "#22c55e",
          "icon": "utensils",
          "classification": "expense",
          "parentId": "550e8400-e29b-41d4-a716-446655440000"
        }
      ]
    }
  ]
}
```

### Create Category
```http
POST /api/v1/categories
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Entertainment",
  "color": "#8b5cf6",
  "icon": "film",
  "classification": "expense",
  "parentId": null
}

Response 201:
{
  "data": { ... }
}
```

### Bootstrap Default Categories
```http
POST /api/v1/categories/bootstrap
Authorization: Bearer {token}

Response 201:
{
  "data": {
    "createdCount": 14
  }
}
```

## Budgets

### Get Budget
```http
GET /api/v1/budgets/:monthYear
Authorization: Bearer {token}

// monthYear format: "jan-2024", "feb-2024", etc.

Response 200:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "startDate": "2024-01-01",
    "endDate": "2024-01-31",
    "currency": "USD",
    "budgetedSpending": "3000.00",
    "expectedIncome": "5000.00",
    "actualSpending": "2150.75",
    "actualIncome": "5000.00",
    "categories": [
      {
        "categoryId": "...",
        "categoryName": "Food & Drink",
        "categoryColor": "#22c55e",
        "budgeted": "500.00",
        "actual": "425.50",
        "available": "74.50",
        "percentUsed": 85.1,
        "isOverBudget": false
      }
    ]
  }
}
```

### Update Budget Category
```http
PATCH /api/v1/budgets/:monthYear/categories/:categoryId
Authorization: Bearer {token}
Content-Type: application/json

{
  "budgetedSpending": "600.00"
}

Response 200:
{
  "data": { ... }
}
```

## Balance Sheet

### Get Balance Sheet
```http
GET /api/v1/balance-sheet
Authorization: Bearer {token}

Query params:
- date: string (ISO date, default: today)

Response 200:
{
  "data": {
    "date": "2024-01-15",
    "currency": "USD",
    "totalAssets": "150000.00",
    "totalLiabilities": "25000.00",
    "netWorth": "125000.00",
    "assets": {
      "Depository": {
        "total": "25000.00",
        "accounts": [
          { "id": "...", "name": "Checking", "balance": "5000.00" },
          { "id": "...", "name": "Savings", "balance": "20000.00" }
        ]
      },
      "Investment": {
        "total": "100000.00",
        "accounts": [...]
      },
      "Property": {
        "total": "25000.00",
        "accounts": [...]
      }
    },
    "liabilities": {
      "CreditCard": {
        "total": "5000.00",
        "accounts": [...]
      },
      "Loan": {
        "total": "20000.00",
        "accounts": [...]
      }
    }
  }
}
```

### Get Net Worth Series
```http
GET /api/v1/balance-sheet/net-worth-series
Authorization: Bearer {token}

Query params:
- period: "last_30_days" | "last_90_days" | "last_12_months" | "all_time"

Response 200:
{
  "data": {
    "currency": "USD",
    "trend": {
      "direction": "up",
      "percentage": "5.2",
      "amount": "6250.00"
    },
    "series": [
      { "date": "2024-01-01", "netWorth": "120000.00" },
      { "date": "2024-01-02", "netWorth": "120500.00" },
      ...
    ]
  }
}
```

## Income Statement

### Get Income Statement
```http
GET /api/v1/income-statement
Authorization: Bearer {token}

Query params:
- startDate: string (ISO date)
- endDate: string (ISO date)

Response 200:
{
  "data": {
    "period": {
      "startDate": "2024-01-01",
      "endDate": "2024-01-31"
    },
    "currency": "USD",
    "totalIncome": "5000.00",
    "totalExpenses": "3500.00",
    "netIncome": "1500.00",
    "income": [
      {
        "categoryId": "...",
        "categoryName": "Income",
        "amount": "5000.00",
        "transactionCount": 2
      }
    ],
    "expenses": [
      {
        "categoryId": "...",
        "categoryName": "Food & Drink",
        "amount": "500.00",
        "transactionCount": 25,
        "subcategories": [
          {
            "categoryId": "...",
            "categoryName": "Restaurants",
            "amount": "200.00",
            "transactionCount": 10
          }
        ]
      }
    ]
  }
}
```

## AI Chat

### List Chats
```http
GET /api/v1/chats
Authorization: Bearer {token}

Response 200:
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "title": "Monthly spending analysis",
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-15T10:35:00Z"
    }
  ]
}
```

### Create Chat with Message
```http
POST /api/v1/chats
Authorization: Bearer {token}
Content-Type: application/json

{
  "message": "What did I spend on food last month?"
}

Response 201:
{
  "data": {
    "chatId": "550e8400-e29b-41d4-a716-446655440000",
    "messageId": "550e8400-e29b-41d4-a716-446655440001",
    "status": "processing"
  }
}
```

### Get Chat with Messages
```http
GET /api/v1/chats/:id
Authorization: Bearer {token}

Response 200:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "title": "Monthly spending analysis",
    "messages": [
      {
        "id": "...",
        "type": "user_message",
        "content": "What did I spend on food last month?",
        "createdAt": "2024-01-15T10:30:00Z"
      },
      {
        "id": "...",
        "type": "assistant_message",
        "content": "Based on your transactions, you spent $425.50 on food last month...",
        "status": "complete",
        "createdAt": "2024-01-15T10:30:05Z"
      }
    ]
  }
}
```

### Send Message
```http
POST /api/v1/chats/:id/messages
Authorization: Bearer {token}
Content-Type: application/json

{
  "content": "How does that compare to the previous month?"
}

Response 201:
{
  "data": {
    "messageId": "550e8400-e29b-41d4-a716-446655440001",
    "status": "processing"
  }
}
```

## Plaid Integration

### Create Link Token
```http
POST /api/v1/plaid/link-token
Authorization: Bearer {token}
Content-Type: application/json

{
  "region": "us"
}

Response 200:
{
  "data": {
    "linkToken": "link-sandbox-abc123...",
    "expiration": "2024-01-15T11:30:00Z"
  }
}
```

### Exchange Public Token
```http
POST /api/v1/plaid/items
Authorization: Bearer {token}
Content-Type: application/json

{
  "publicToken": "public-sandbox-abc123...",
  "institutionId": "ins_123",
  "accountIds": ["account_1", "account_2"]
}

Response 201:
{
  "data": {
    "plaidItemId": "550e8400-e29b-41d4-a716-446655440000",
    "accounts": [
      {
        "id": "...",
        "name": "Checking ****1234",
        "type": "depository"
      }
    ],
    "syncStatus": "pending"
  }
}
```

### Sync Plaid Item
```http
POST /api/v1/plaid/items/:id/sync
Authorization: Bearer {token}

Response 202:
{
  "data": {
    "syncId": "550e8400-e29b-41d4-a716-446655440000",
    "status": "pending"
  }
}
```

## Imports

### Create Import
```http
POST /api/v1/imports
Authorization: Bearer {token}
Content-Type: multipart/form-data

FormData:
- file: (CSV file)
- type: "TransactionImport" | "TradeImport" | "AccountImport"
- accountId: (optional) target account UUID

Response 201:
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "status": "pending",
    "detectedColumns": ["Date", "Description", "Amount"],
    "rowCount": 150
  }
}
```

### Update Import Configuration
```http
PATCH /api/v1/imports/:id/configuration
Authorization: Bearer {token}
Content-Type: application/json

{
  "dateColLabel": "Date",
  "amountColLabel": "Amount",
  "nameColLabel": "Description",
  "dateFormat": "%m/%d/%Y",
  "signageConvention": "inflows_positive"
}

Response 200:
{
  "data": { ... }
}
```

### Get Import Rows
```http
GET /api/v1/imports/:id/rows
Authorization: Bearer {token}

Query params:
- page: number
- perPage: number

Response 200:
{
  "data": [
    {
      "id": "...",
      "date": "01/15/2024",
      "name": "Coffee Shop",
      "amount": "-5.50",
      "category": null,
      "isValid": true,
      "errors": []
    }
  ],
  "meta": {
    "pagination": { ... }
  }
}
```

### Publish Import
```http
POST /api/v1/imports/:id/publish
Authorization: Bearer {token}

Response 202:
{
  "data": {
    "status": "importing",
    "jobId": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Revert Import
```http
POST /api/v1/imports/:id/revert
Authorization: Bearer {token}

Response 202:
{
  "data": {
    "status": "reverting",
    "jobId": "550e8400-e29b-41d4-a716-446655440001"
  }
}
```

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `INVALID_CREDENTIALS` | 401 | Email or password incorrect |
| `MFA_REQUIRED` | 401 | MFA verification needed |
| `MFA_INVALID` | 401 | Invalid MFA code |
| `UNAUTHORIZED` | 401 | Not authenticated |
| `FORBIDDEN` | 403 | Not authorized for this resource |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource already exists |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |
