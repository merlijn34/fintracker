# Technical Architecture & Implementation Guide
## Personal Finance Platform

**Document Version**: 1.0  
**Last Updated**: February 2026

---

## Table of Contents
1. [Architecture Overview](#1-architecture-overview)
2. [Technology Stack](#2-technology-stack)
3. [API Design](#3-api-design)
4. [Database Schema](#4-database-schema)
5. [Frontend Architecture](#5-frontend-architecture)
6. [Backend Architecture](#7-backend-architecture)
7. [Security Considerations](#8-security-considerations)
8. [Performance Optimization](#9-performance-optimization)
9. [Deployment & DevOps](#10-deployment--devops)
10. [Testing Strategy](#11-testing-strategy)

---

## 1. Architecture Overview

### System Architecture Diagram
```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend (Web)                            │
│  React / Vue / Svelte + TypeScript + Tailwind CSS              │
│  - Authentication UI                                            │
│  - Account Management                                           │
│  - Transaction Management                                       │
│  - Analytics & Charts                                           │
│  - Settings                                                      │
└──────────────────────┬──────────────────────────────────────────┘
                       │ HTTPS / JSON
                       ↓
┌─────────────────────────────────────────────────────────────────┐
│                     API Gateway / Proxy                          │
│  - Rate limiting                                                │
│  - Authentication verification                                 │
│  - Request/response logging                                    │
│  - CORS handling                                                │
└──────────────────────┬──────────────────────────────────────────┘
                       │
    ┌──────────────────┼──────────────────┐
    ↓                  ↓                   ↓
┌────────────┐  ┌────────────┐  ┌────────────┐
│ REST API   │  │  WebSocket │  │  File      │
│ Endpoints  │  │  (Real-     │  │  Storage   │
│ (Node/     │  │   time      │  │  (Backups) │
│  Django/   │  │   updates)  │  │            │
│  Rails)    │  │             │  │            │
└──────┬─────┘  └──────┬──────┘  └────────────┘
       │                │
       └────────┬───────┘
                ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Business Logic Layer                          │
│  - Account Service                                              │
│  - Transaction Service                                          │
│  - Budget Service                                               │
│  - Analytics Service                                            │
│  - Net Worth Calculation                                        │
│  - Category Management                                          │
│  - Transfer Detection                                           │
│  - Currency Conversion                                          │
└──────────────────────┬──────────────────────────────────────────┘
                       │
    ┌──────────────────┼──────────────────┐
    ↓                  ↓                   ↓
┌────────────┐  ┌────────────┐  ┌────────────┐
│ Database   │  │ Cache      │  │ Queue      │
│ Layer      │  │ (Redis)    │  │ (Bull)     │
│ (PostgreSQL)│  │            │  │            │
│ - ORM      │  │ - Sessions │  │ - Email    │
│ - Queries  │  │ - Caching  │  │ - Reports  │
│ - Migrations│ │ - Locks    │  │ - Tasks    │
└────────────┘  └────────────┘  └────────────┘
```

### Key Architectural Decisions

1. **Monolithic with Modular Structure**
   - Single codebase for MVP
   - Service-oriented within same process
   - Can be split into microservices later

2. **Self-Hosted Only (Phase 1)**
   - No multi-tenancy required
   - Simpler architecture
   - Full data control for user

3. **Stateless Backend**
   - Sessions stored in database/Redis
   - Allows horizontal scaling
   - Stateless API endpoints

4. **Event-Driven Updates**
   - Financial calculations can be event-based
   - Eventual consistency for some features
   - Real-time net worth updates

5. **Privacy-First Data Handling**
   - No data tracking/analytics (local only)
   - Encrypted backups
   - No third-party calls with PII

---

## 2. Technology Stack

### Recommended Stack Options

#### Option A: Modern Node.js Stack (Most Flexible)
- **Frontend**: React 18+ or Vue 3 + TypeScript
- **Backend**: Node.js + Express or NestJS
- **Database**: PostgreSQL 13+
- **Cache**: Redis 6+
- **Job Queue**: Bull (Redis-backed)
- **ORM**: Prisma, TypeORM, or Sequelize
- **Styling**: Tailwind CSS
- **Charts**: Chart.js, Recharts, or D3.js

**Advantages**:
- Single language (JavaScript/TypeScript) across stack
- Large ecosystem
- Great developer experience
- Good performance for financial app
- Easy to deploy

#### Option B: Python Stack
- **Frontend**: React/Vue with TypeScript
- **Backend**: Django + Django REST Framework or FastAPI
- **Database**: PostgreSQL 13+
- **Cache**: Redis
- **Job Queue**: Celery
- **ORM**: Django ORM or SQLAlchemy
- **Styling**: Tailwind CSS

**Advantages**:
- Robust Python ecosystem
- Django's batteries-included approach
- Good for data processing
- Strong security defaults

#### Option C: Ruby on Rails (Original Maybe Stack)
- **Frontend**: React/Vue with Rails integration
- **Backend**: Ruby on Rails 7+
- **Database**: PostgreSQL
- **Cache**: Redis
- **Job Queue**: Sidekiq
- **ORM**: ActiveRecord
- **Styling**: Tailwind CSS

**Advantages**:
- Convention over configuration
- Mature ecosystem
- Excellent for rapid development
- Good for financial calculations

### Recommended Stack for This Project
**Node.js + React + PostgreSQL** (most flexibility, active community)

---

## 3. API Design

### REST API Structure

#### Base URL
```
https://yourdomain.com/api/v1
```

#### Authentication
```
Authorization: Bearer <JWT_TOKEN>
```

### Core Resource Endpoints

#### Authentication
```
POST   /auth/register
POST   /auth/login
POST   /auth/logout
POST   /auth/refresh-token
POST   /auth/forgot-password
POST   /auth/reset-password
POST   /auth/2fa/enable
POST   /auth/2fa/verify
POST   /auth/2fa/disable
```

#### User Profile
```
GET    /user/profile
PUT    /user/profile
PUT    /user/password
DELETE /user/account
GET    /user/preferences
PUT    /user/preferences
```

#### Accounts
```
GET    /accounts
POST   /accounts
GET    /accounts/:id
PUT    /accounts/:id
DELETE /accounts/:id
GET    /accounts/:id/balance-history
GET    /accounts/:id/transactions
GET    /accounts/:id/reconcile
POST   /accounts/:id/reconcile
```

#### Transactions
```
GET    /transactions
POST   /transactions
POST   /transactions/import
GET    /transactions/:id
PUT    /transactions/:id
DELETE /transactions/:id
POST   /transactions/:id/duplicate
GET    /transactions/search?q=<query>
```

**Query Parameters for Transactions**:
```
- account_id: filter by account
- category_id: filter by category
- from_date: filter by start date
- to_date: filter by end date
- type: filter by transaction type
- status: filter by reconciliation status
- sort: field to sort by
- order: asc|desc
- page: pagination page number
- limit: items per page (default 50, max 100)
```

#### Categories
```
GET    /categories
POST   /categories
GET    /categories/:id
PUT    /categories/:id
DELETE /categories/:id
GET    /categories/stats
```

#### Budgets
```
GET    /budgets?month=<YYYY-MM>
POST   /budgets
GET    /budgets/:id
PUT    /budgets/:id
DELETE /budgets/:id
GET    /budgets/:id/progress
```

#### Reports & Analytics
```
GET    /reports/net-worth
GET    /reports/net-worth/history
GET    /reports/spending
GET    /reports/spending/by-category
GET    /reports/income-vs-expenses
GET    /reports/account-summary
GET    /reports/budget-summary
```

#### Investments
```
GET    /accounts/:account_id/trades
POST   /accounts/:account_id/trades
GET    /accounts/:account_id/trades/:id
PUT    /accounts/:account_id/trades/:id
DELETE /accounts/:account_id/trades/:id
GET    /accounts/:account_id/portfolio
```

#### Import/Export
```
POST   /import/csv
POST   /import/preview
GET    /export/transactions
GET    /export/accounts
GET    /export/all
```

#### Settings
```
GET    /settings
PUT    /settings
POST   /settings/backup
GET    /settings/backup-history
POST   /settings/restore
```

### Request/Response Format

#### Request Example
```json
{
  "amount": 45.50,
  "merchant": "Whole Foods",
  "date": "2026-02-01",
  "category_id": "cat_123",
  "account_id": "acc_456",
  "notes": "Groceries"
}
```

#### Response Format
```json
{
  "success": true,
  "data": {
    "id": "txn_789",
    "amount": 45.50,
    "merchant": "Whole Foods",
    "date": "2026-02-01",
    "category_id": "cat_123",
    "account_id": "acc_456",
    "notes": "Groceries",
    "created_at": "2026-02-01T10:30:00Z",
    "updated_at": "2026-02-01T10:30:00Z"
  }
}
```

#### Error Response
```json
{
  "success": false,
  "error": {
    "code": "INVALID_AMOUNT",
    "message": "Amount must be a positive number",
    "details": {
      "field": "amount",
      "value": -45.50
    }
  }
}
```

### HTTP Status Codes
- **200**: Success (GET, PUT, POST)
- **201**: Created (POST resource creation)
- **204**: No content (DELETE)
- **400**: Bad request (validation error)
- **401**: Unauthorized (invalid/missing auth)
- **403**: Forbidden (insufficient permissions)
- **404**: Not found
- **409**: Conflict (duplicate, constraint violation)
- **422**: Unprocessable entity (invalid data)
- **429**: Too many requests (rate limit)
- **500**: Server error

### Rate Limiting
```
- Unauthenticated: 10 requests/minute per IP
- Authenticated: 100 requests/minute per user
- Auth endpoints: 5 attempts/minute per IP
- Import: 2 concurrent imports per user
```

---

## 4. Database Schema

### Core Tables (Simplified)

```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  preferred_currency CHAR(3) DEFAULT 'USD',
  theme VARCHAR(20) DEFAULT 'system',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_login_at TIMESTAMP,
  deleted_at TIMESTAMP -- soft delete
);

-- Accounts Table
CREATE TABLE accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  account_type VARCHAR(50) NOT NULL, -- checking, savings, etc.
  currency CHAR(3) NOT NULL DEFAULT 'USD',
  institution_name VARCHAR(255),
  account_number VARCHAR(50),
  opening_balance DECIMAL(19,2) DEFAULT 0,
  opening_date DATE,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP -- soft delete
);

CREATE INDEX idx_accounts_user_id ON accounts(user_id);
CREATE INDEX idx_accounts_active ON accounts(user_id, active);

-- Account Balance History
CREATE TABLE account_balances (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
  balance DECIMAL(19,2) NOT NULL,
  date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_account_balances_account_date ON account_balances(account_id, date DESC);

-- Transactions Table
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  account_id UUID NOT NULL REFERENCES accounts(id),
  date DATE NOT NULL,
  amount DECIMAL(19,2) NOT NULL,
  currency CHAR(3) NOT NULL,
  merchant_description VARCHAR(255) NOT NULL,
  category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
  transaction_type VARCHAR(50) NOT NULL, -- income, expense, transfer, trade
  related_account_id UUID REFERENCES accounts(id) ON DELETE SET NULL,
  is_transfer BOOLEAN DEFAULT false,
  is_excluded_from_budget BOOLEAN DEFAULT false,
  is_one_time BOOLEAN DEFAULT false,
  reconciliation_status VARCHAR(20) DEFAULT 'uncleared', -- uncleared, cleared, reconciled
  notes TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP -- soft delete
);

CREATE INDEX idx_transactions_user_date ON transactions(user_id, date DESC);
CREATE INDEX idx_transactions_account ON transactions(account_id);
CREATE INDEX idx_transactions_category ON transactions(category_id);
CREATE INDEX idx_transactions_merchant ON transactions USING GIN(to_tsvector('english', merchant_description));

-- Categories Table
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  parent_category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
  color VARCHAR(7), -- hex color
  icon VARCHAR(50),
  custom BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE INDEX idx_categories_user_parent ON categories(user_id, parent_category_id);

-- Budgets Table
CREATE TABLE budgets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id UUID NOT NULL REFERENCES categories(id),
  month_year DATE NOT NULL, -- first day of month
  limit_amount DECIMAL(19,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,
  UNIQUE(user_id, category_id, month_year)
);

CREATE INDEX idx_budgets_user_month ON budgets(user_id, month_year);

-- Tags Table
CREATE TABLE tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(50) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, name)
);

-- Transaction Tags Join Table
CREATE TABLE transaction_tags (
  transaction_id UUID REFERENCES transactions(id) ON DELETE CASCADE,
  tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY(transaction_id, tag_id)
);

-- Investment Trades Table
CREATE TABLE investment_trades (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id UUID NOT NULL REFERENCES accounts(id),
  transaction_id UUID REFERENCES transactions(id),
  ticker VARCHAR(10) NOT NULL,
  quantity DECIMAL(19,4) NOT NULL,
  price_per_share DECIMAL(19,4) NOT NULL,
  trade_date DATE NOT NULL,
  trade_type VARCHAR(10) NOT NULL, -- buy, sell
  fees DECIMAL(19,2) DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_investment_trades_account ON investment_trades(account_id);

-- Exchange Rates Table
CREATE TABLE exchange_rates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_currency CHAR(3) NOT NULL,
  to_currency CHAR(3) NOT NULL,
  rate DECIMAL(19,6) NOT NULL,
  date DATE NOT NULL,
  source VARCHAR(50), -- api, manual, etc.
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(from_currency, to_currency, date)
);

-- User Preferences Table
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  key VARCHAR(50) NOT NULL,
  value TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Data Integrity & Constraints

1. **Account Balance**
   - Always calculated from transactions (no stored balance)
   - Or stored separately and synced regularly

2. **Net Worth Calculation**
   - Real-time sum of all account balances
   - Cached for performance (update on transaction)
   - Multi-currency with exchange rates

3. **Transfer Detection**
   - Bidirectional relationship
   - Both sides linked via related_account_id
   - Consistent amounts

4. **Budget Calculation**
   - Sum transactions in month with category
   - Exclude transfers automatically
   - Exclude one-time expenses if flagged

5. **Account Reconciliation**
   - Lock period after reconciliation
   - Prevent editing cleared transactions
   - Optional: archive reconciled periods

---

## 5. Frontend Architecture

### Project Structure
```
frontend/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── manifest.json (PWA)
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   └── ...
│   │   ├── auth/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── ...
│   │   ├── accounts/
│   │   │   ├── AccountList.tsx
│   │   │   ├── AccountDetail.tsx
│   │   │   ├── AccountForm.tsx
│   │   │   └── ...
│   │   ├── transactions/
│   │   │   ├── TransactionList.tsx
│   │   │   ├── TransactionForm.tsx
│   │   │   ├── TransactionSearch.tsx
│   │   │   └── ...
│   │   ├── budgets/
│   │   ├── analytics/
│   │   └── settings/
│   ├── pages/
│   │   ├── Dashboard.tsx
│   │   ├── AccountsPage.tsx
│   │   ├── TransactionsPage.tsx
│   │   ├── BudgetsPage.tsx
│   │   ├── AnalyticsPage.tsx
│   │   ├── SettingsPage.tsx
│   │   └── ...
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useAccounts.ts
│   │   ├── useTransactions.ts
│   │   ├── useBudgets.ts
│   │   └── ...
│   ├── services/
│   │   ├── api.ts (axios/fetch client)
│   │   ├── authService.ts
│   │   ├── accountService.ts
│   │   ├── transactionService.ts
│   │   └── ...
│   ├── store/ (or context/)
│   │   ├── authStore.ts
│   │   ├── accountStore.ts
│   │   ├── transactionStore.ts
│   │   └── ...
│   ├── utils/
│   │   ├── formatting.ts (currency, numbers)
│   │   ├── validation.ts
│   │   ├── currency.ts
│   │   └── ...
│   ├── types/
│   │   ├── index.ts (TypeScript types/interfaces)
│   │   ├── api.ts
│   │   └── ...
│   ├── styles/
│   │   ├── globals.css
│   │   ├── tailwind.config.js
│   │   └── ...
│   ├── App.tsx
│   ├── App.css
│   └── index.tsx
├── package.json
├── tsconfig.json
└── .env (local, not committed)
```

### State Management
**Recommended**: Zustand (lightweight) or Redux (if complex)

```typescript
// Example Zustand store
import create from 'zustand';

interface AuthState {
  user: User | null;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  setUser: (user: User) => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isLoading: false,
  login: async (email, password) => {
    // API call
  },
  logout: () => set({ user: null }),
  setUser: (user) => set({ user }),
}));
```

### Component Patterns

**Presentational Component** (Pure, reusable):
```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ 
  label, 
  onClick, 
  variant = 'primary', 
  disabled = false 
}) => {
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {label}
    </button>
  );
};

export default Button;
```

**Container Component** (Logic, API calls):
```typescript
const TransactionList: React.FC = () => {
  const [transactions, setTransactions] = useState<Transaction[]>([]);
  const [loading, setLoading] = useState(true);
  const [filters, setFilters] = useState({...});

  useEffect(() => {
    fetchTransactions();
  }, [filters]);

  const fetchTransactions = async () => {
    setLoading(true);
    try {
      const data = await transactionService.list(filters);
      setTransactions(data);
    } catch (error) {
      // handle error
    } finally {
      setLoading(false);
    }
  };

  return <TransactionListView transactions={transactions} />;
};
```

### Forms & Validation
**Library**: React Hook Form + Zod

```typescript
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const transactionSchema = z.object({
  amount: z.number().positive('Amount must be positive'),
  merchant: z.string().min(1, 'Merchant required').max(100),
  date: z.date(),
  categoryId: z.string().optional(),
});

type TransactionFormData = z.infer<typeof transactionSchema>;

const TransactionForm: React.FC = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<TransactionFormData>({
    resolver: zodResolver(transactionSchema),
  });

  const onSubmit = async (data: TransactionFormData) => {
    await transactionService.create(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('amount')} type="number" />
      {errors.amount && <span>{errors.amount.message}</span>}
      {/* More fields */}
      <button type="submit">Save</button>
    </form>
  );
};
```

### Styling
**Framework**: Tailwind CSS

```typescript
export const Button: React.FC<ButtonProps> = ({ variant }) => {
  const baseStyles = 'px-4 py-2 rounded font-semibold transition';
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-600 text-white hover:bg-red-700',
  };

  return (
    <button className={`${baseStyles} ${variants[variant]}`}>
      Click me
    </button>
  );
};
```

### Dark Mode Implementation

Using Tailwind's built-in dark mode with CSS variables:

```typescript
// App.tsx
const [theme, setTheme] = useState('light');

useEffect(() => {
  const root = document.documentElement;
  if (theme === 'dark') {
    root.classList.add('dark');
  } else {
    root.classList.remove('dark');
  }
}, [theme]);

// Components
<div className="bg-white dark:bg-gray-900">
  <h1 className="text-gray-900 dark:text-white">Title</h1>
</div>
```

### Charts & Visualizations
**Library**: Recharts (React-friendly) or Chart.js

```typescript
import { LineChart, Line, XAxis, YAxis, Tooltip, Legend } from 'recharts';

const NetWorthChart: React.FC = () => {
  const data = [
    { date: '2026-01-01', netWorth: 50000 },
    { date: '2026-02-01', netWorth: 52500 },
    // ...
  ];

  return (
    <LineChart width={800} height={400} data={data}>
      <XAxis dataKey="date" />
      <YAxis />
      <Tooltip />
      <Legend />
      <Line type="monotone" dataKey="netWorth" stroke="#8884d8" />
    </LineChart>
  );
};
```

### API Client
```typescript
import axios, { AxiosInstance } from 'axios';

class ApiClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: process.env.REACT_APP_API_URL,
      withCredentials: true,
    });

    // Add JWT token to requests
    this.client.interceptors.request.use((config) => {
      const token = localStorage.getItem('token');
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    });

    // Handle responses
    this.client.interceptors.response.use(
      (response) => response.data,
      (error) => {
        if (error.response?.status === 401) {
          // Redirect to login
        }
        return Promise.reject(error);
      }
    );
  }

  get<T>(url: string, config?: any) {
    return this.client.get<T>(url, config);
  }

  post<T>(url: string, data?: any, config?: any) {
    return this.client.post<T>(url, data, config);
  }

  // ... other methods
}

export default new ApiClient();
```

---

## 6. Backend Architecture

### Project Structure (Node.js/Express)
```
backend/
├── src/
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── accounts.routes.ts
│   │   ├── transactions.routes.ts
│   │   ├── budgets.routes.ts
│   │   ├── analytics.routes.ts
│   │   └── ...
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── accounts.controller.ts
│   │   ├── transactions.controller.ts
│   │   └── ...
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── account.service.ts
│   │   ├── transaction.service.ts
│   │   ├── budget.service.ts
│   │   ├── analytics.service.ts
│   │   ├── currency.service.ts
│   │   └── ...
│   ├── models/ (or database layer)
│   │   ├── user.model.ts
│   │   ├── account.model.ts
│   │   ├── transaction.model.ts
│   │   └── ...
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   ├── error.middleware.ts
│   │   ├── validation.middleware.ts
│   │   ├── rateLimit.middleware.ts
│   │   └── ...
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── errors.ts
│   │   ├── validators.ts
│   │   ├── currency.ts
│   │   └── ...
│   ├── jobs/ (background tasks)
│   │   ├── calculateNetWorth.job.ts
│   │   ├── generateReports.job.ts
│   │   ├── sendNotifications.job.ts
│   │   └── ...
│   ├── config/
│   │   ├── database.ts
│   │   ├── redis.ts
│   │   ├── env.ts
│   │   └── ...
│   ├── types/
│   │   ├── express.d.ts
│   │   ├── index.ts
│   │   └── ...
│   ├── app.ts
│   └── server.ts
├── migrations/ (database migrations)
├── seeds/ (database seeders)
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── package.json
├── tsconfig.json
├── docker-compose.yml
└── .env.example
```

### Core Services

#### Auth Service
```typescript
class AuthService {
  async register(email: string, password: string): Promise<User> {
    // Validate input
    // Check if email exists
    // Hash password
    // Create user
    // Return user
  }

  async login(email: string, password: string): Promise<{ user: User; token: string }> {
    // Find user by email
    // Verify password
    // Generate JWT
    // Update last login
    // Return user and token
  }

  async resetPassword(email: string, token: string, newPassword: string): Promise<void> {
    // Verify reset token
    // Hash new password
    // Update user password
    // Invalidate reset token
  }

  verifyToken(token: string): JwtPayload {
    // Verify and decode JWT
  }
}
```

#### Account Service
```typescript
class AccountService {
  async createAccount(userId: string, data: CreateAccountDto): Promise<Account> {
    // Validate input
    // Create account
    // Set opening balance
    // Return account
  }

  async getAccounts(userId: string): Promise<Account[]> {
    // Fetch all active accounts
    // Include current balance
    // Sort by name
  }

  async getAccountBalance(accountId: string, date?: Date): Promise<number> {
    // Sum all transactions for account up to date
    // Or fetch from balance history
    // Handle multi-currency conversion
  }

  async getNetWorth(userId: string, date?: Date): Promise<{
    total: number;
    assets: number;
    liabilities: number;
    breakdown: { [key: string]: number };
  }> {
    // Sum all account balances (assets - liabilities)
    // Convert to preferred currency
    // Breakdown by account type
  }

  async reconcileAccount(accountId: string, statementBalance: number): Promise<void> {
    // Verify statement balance matches calculated balance
    // Create adjustment transaction if needed
    // Lock reconciliation period
  }
}
```

#### Transaction Service
```typescript
class TransactionService {
  async createTransaction(userId: string, data: CreateTransactionDto): Promise<Transaction> {
    // Validate input
    // Detect if transfer
    // Set category if auto-categorizable
    // Create transaction
    // Update account balance
    // Recalculate budgets
    // Return transaction
  }

  async listTransactions(
    userId: string,
    filters: TransactionFilters,
    pagination: Pagination
  ): Promise<{ transactions: Transaction[]; total: number }> {
    // Build query with filters
    // Apply search
    // Apply pagination
    // Return transactions and count
  }

  async detectTransfers(userId: string): Promise<Transfer[]> {
    // Find transactions that appear to be transfers
    // Match by amount, date, accounts
    // Return candidates
  }

  async categorizeTransaction(transaction: Transaction): Promise<string> {
    // Rule-based or AI categorization
    // Return category ID
  }

  async importTransactions(userId: string, file: File, mapping: ColumnMapping): Promise<{
    imported: number;
    skipped: number;
    errors: string[];
  }> {
    // Parse CSV
    // Validate each row
    // Detect duplicates
    // Batch insert
    // Return summary
  }
}
```

#### Budget Service
```typescript
class BudgetService {
  async createBudget(userId: string, data: CreateBudgetDto): Promise<Budget> {
    // Validate inputs
    // Create budget
    // Return budget
  }

  async getBudgetProgress(budgetId: string): Promise<{
    budget: Budget;
    spent: number;
    remaining: number;
    percentage: number;
  }> {
    // Calculate sum of transactions in category for month
    // Exclude transfers and one-time expenses
    // Calculate percentage used
  }

  async getMonthlyBudgets(userId: string, month: Date): Promise<Budget[]> {
    // Get all budgets for month
    // Include progress for each
  }
}
```

#### Analytics Service
```typescript
class AnalyticsService {
  async getSpendingByCategory(userId: string, startDate: Date, endDate: Date) {
    // Sum transactions by category
    // Exclude transfers
    // Return breakdown
  }

  async getIncomeVsExpenses(userId: string, startDate: Date, endDate: Date) {
    // Sum income and expense transactions
    // Calculate net
    // Return by month
  }

  async getNetWorthHistory(userId: string, months: number = 12) {
    // Fetch daily snapshots for last N months
    // Return time series
  }

  async getCashFlow(userId: string, month: Date) {
    // Calculate inflows and outflows
    // Return summary
  }
}
```

### Middleware

#### Authentication Middleware
```typescript
export const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  try {
    const payload = authService.verifyToken(token);
    req.user = payload;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

#### Error Handling Middleware
```typescript
export const errorHandler = (
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (error instanceof ValidationError) {
    return res.status(422).json({ errors: error.details });
  }

  if (error instanceof NotFoundError) {
    return res.status(404).json({ error: error.message });
  }

  logger.error(error);
  res.status(500).json({ error: 'Internal server error' });
};
```

### Background Jobs (Bull Queue)

```typescript
import Queue from 'bull';

// Create job queues
const netWorthQueue = new Queue('net-worth', process.env.REDIS_URL);
const reportQueue = new Queue('reports', process.env.REDIS_URL);

// Define job processors
netWorthQueue.process(async (job) => {
  const { userId } = job.data;
  await analyticsService.updateNetWorthSnapshot(userId);
});

reportQueue.process(async (job) => {
  const { userId, month } = job.data;
  const report = await analyticsService.generateMonthlyReport(userId, month);
  await emailService.sendReport(userId, report);
});

// Schedule jobs
async function scheduleJobs() {
  // Every night at 2 AM, update net worth for all users
  netWorthQueue.add({}, { repeat: { cron: '0 2 * * *' } });
}
```

---

## 7. Security Considerations

### Authentication & Authorization

1. **Password Security**
   - Hash with bcrypt or Argon2 (min 12 rounds for bcrypt)
   - Never log passwords
   - Require 8+ characters minimum
   - Suggest complexity but don't require

2. **JWT Token Management**
   - Access token: 15 minutes expiry
   - Refresh token: 7 days expiry
   - Store refresh token in httpOnly cookie
   - Revoke tokens on logout

3. **Two-Factor Authentication**
   - Support TOTP (Google Authenticator, Authy)
   - Generate and display backup codes
   - Store as hashed values only

### Data Protection

1. **Encryption**
   - HTTPS/TLS 1.2+ for all communication
   - Encrypt sensitive data at rest (optional):
     - User emails
     - Account numbers (partial)
     - Sensitive preferences

2. **Input Validation**
   - Validate all user inputs on server
   - Use strong typing (TypeScript)
   - Sanitize inputs to prevent injection attacks

3. **SQL Injection Prevention**
   - Use parameterized queries / ORM
   - Never concatenate SQL strings
   - Validate data types

4. **XSS Prevention**
   - Sanitize HTML output
   - Use Content Security Policy headers
   - Never eval() user input

### API Security

1. **Rate Limiting**
   - Global: 100 requests/minute per user
   - Auth endpoints: 5 attempts/minute per IP
   - Implement using Redis

2. **CSRF Protection**
   - Use SameSite cookies
   - CSRF tokens for state-changing requests
   - Verify origin headers

3. **CORS**
   - Allow only your domain (no *)
   - Specify allowed methods and headers
   - Allow credentials only for same origin

4. **API Versioning**
   - Version all endpoints (/api/v1)
   - Support multiple versions during transition
   - Sunset old versions

### Environment Security

1. **Configuration**
   - Never commit .env files
   - Use environment variables for secrets
   - Different configs for dev/staging/production
   - Rotate secrets regularly

2. **Logging**
   - Log all auth attempts
   - Never log passwords or tokens
   - Log API errors for debugging
   - Store logs securely

3. **Dependencies**
   - Keep npm packages updated
   - Use `npm audit` regularly
   - Vet new dependencies
   - Use lock files (package-lock.json)

---

## 8. Performance Optimization

### Database Performance

1. **Indexing Strategy**
   - Index on user_id for all user tables
   - Index on date for transaction queries
   - Index on account_id for account queries
   - Composite indexes for common filters
   - Avoid over-indexing

2. **Query Optimization**
   - Use EXPLAIN to analyze slow queries
   - Lazy load related data
   - Batch queries when possible
   - Limit result sets (pagination)

3. **Connection Pooling**
   - Use connection pools (min 10, max 100 connections)
   - Reuse connections
   - Close idle connections

### Caching Strategy

1. **Redis Caching**
   - Cache user preferences (5 minute TTL)
   - Cache account balances (1 minute TTL)
   - Cache budget progress (5 minute TTL)
   - Cache exchange rates (1 hour TTL)
   - Invalidate on transaction/account update

2. **HTTP Caching**
   - Set Cache-Control headers
   - Use ETags for conditional requests
   - Cache static assets (1 year)
   - Cache API responses (1-5 minutes)

### API Response Optimization

1. **Pagination**
   - Default: 50 items per page
   - Max: 100 items per page
   - Use offset/limit or cursor-based
   - Include total count for UI

2. **Lazy Loading**
   - Return account list without full transaction history
   - Load related data on detail pages
   - Load charts on demand

3. **Compression**
   - Enable gzip compression
   - Minify JSON responses
   - Use HTTP/2 for multiplexing

### Frontend Performance

1. **Code Splitting**
   - Lazy load pages and components
   - Split dashboard and settings into separate chunks
   - Load charts on demand

2. **Bundle Optimization**
   - Minify JavaScript and CSS
   - Tree-shake unused code
   - Analyze bundle size regularly

3. **Image Optimization**
   - Use WebP format with fallbacks
   - Lazy load images
   - Serve appropriately sized images

---

## 9. Deployment & DevOps

### Docker Setup

```dockerfile
# Dockerfile (Node.js example)
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/personalos
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=your_secret_here
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: personalos
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data

volumes:
  pgdata:
  redisdata:
```

### Deployment Options

1. **Self-Hosted (Recommended)**
   - Docker + Docker Compose
   - VPS (DigitalOcean, Linode, Vultr)
   - Reverse proxy (Nginx)
   - SSL certificate (Let's Encrypt)

2. **Containerized Platforms**
   - Heroku (simple, more expensive)
   - Render (similar to Heroku)
   - Railway (modern alternative)
   - Cloud Run (Google Cloud)

3. **Kubernetes (Advanced)**
   - Helm charts for deployment
   - Service mesh (optional)
   - Auto-scaling
   - Production-ready setup

### CI/CD Pipeline

**GitHub Actions Example**:
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: password
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to VPS
        run: |
          ssh user@server "cd ~/app && git pull && docker-compose down && docker-compose up -d"
```

---

## 10. Testing Strategy

### Unit Tests
**Framework**: Jest

```typescript
describe('AuthService', () => {
  describe('register', () => {
    it('should create a new user', async () => {
      const user = await authService.register('test@example.com', 'password123');
      expect(user.email).toBe('test@example.com');
    });

    it('should hash password', async () => {
      const user = await authService.register('test@example.com', 'password123');
      expect(user.password_hash).not.toBe('password123');
    });

    it('should reject duplicate email', async () => {
      await authService.register('test@example.com', 'password123');
      expect(async () => {
        await authService.register('test@example.com', 'password456');
      }).rejects.toThrow('Email already exists');
    });
  });
});
```

### Integration Tests
```typescript
describe('Transaction API', () => {
  it('POST /transactions should create transaction', async () => {
    const response = await request(app)
      .post('/api/v1/transactions')
      .set('Authorization', `Bearer ${token}`)
      .send({
        amount: 45.50,
        merchant: 'Whole Foods',
        date: '2026-02-01',
        account_id: accountId,
      });

    expect(response.status).toBe(201);
    expect(response.body.data.id).toBeDefined();
  });

  it('GET /transactions should list transactions', async () => {
    const response = await request(app)
      .get('/api/v1/transactions')
      .set('Authorization', `Bearer ${token}`);

    expect(response.status).toBe(200);
    expect(Array.isArray(response.body.data)).toBe(true);
  });
});
```

### E2E Tests
**Framework**: Cypress or Playwright

```typescript
describe('Transaction Creation', () => {
  it('should create transaction from UI', () => {
    cy.login('test@example.com', 'password');
    cy.visit('/transactions');
    cy.get('[data-testid="add-transaction"]').click();
    cy.get('[name="amount"]').type('45.50');
    cy.get('[name="merchant"]').type('Whole Foods');
    cy.get('[data-testid="save"]').click();
    cy.contains('Transaction created successfully').should('be.visible');
  });
});
```

### Test Coverage Targets
- Unit tests: 80%+ coverage
- Integration tests: 60%+ coverage
- E2E tests: Happy paths only
- Overall: 75%+ code coverage

---

## Performance Benchmarks

### Target Metrics

| Metric | Target | Notes |
|--------|--------|-------|
| Dashboard load | < 2s | Cached, initial load |
| Transaction list (100 items) | < 1s | Includes search |
| Net worth calculation | < 500ms | Cached |
| Transaction search | < 500ms | Real-time |
| API endpoint response | < 200ms | p95 |
| Database query | < 100ms | p95 |
| CSS/JS bundle size | < 500KB | Gzipped |
| Database size | < 1GB | 50k transactions |

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Status**: Complete Technical Specification
