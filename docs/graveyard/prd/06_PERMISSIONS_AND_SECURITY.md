# Permissions and Security

## Roles and Permissions Matrix

### User Roles

| Role | Description |
|------|-------------|
| `member` | Standard family member with full access to family data |
| `admin` | Can manage family settings and invite members |
| `super_admin` | Internal support role with impersonation capability |

### Permission Matrix

| Action | member | admin | super_admin |
|--------|--------|-------|-------------|
| View own profile | Yes | Yes | Yes |
| Update own profile | Yes | Yes | Yes |
| Delete own account | Yes | Yes | No |
| View family data | Yes | Yes | Yes |
| Create/edit accounts | Yes | Yes | Yes |
| Delete accounts | Yes | Yes | Yes |
| Create/edit transactions | Yes | Yes | Yes |
| Import data | Yes | Yes | Yes |
| Connect Plaid | Yes | Yes | Yes |
| View family settings | Yes | Yes | Yes |
| Update family settings | No | Yes | Yes |
| Invite family members | No | Yes | Yes |
| Remove family members | No | Yes | Yes |
| Manage invite codes | No | Yes | Yes |
| Impersonate users | No | No | Yes |
| Access admin tools | No | No | Yes |

### API Scopes

| Scope | Permissions |
|-------|-------------|
| `read` | Read-only access to accounts, transactions, budgets |
| `read_write` | Full CRUD access (includes `read`) |

## Sensitive Actions

### High-Risk Actions Requiring Confirmation

1. **Account Deletion**
   - Requires explicit confirmation
   - Soft delete with 7-day recovery period
   - Cascades to entries and balances

2. **User Account Deactivation**
   - Requires password confirmation
   - Scheduled purge after grace period

3. **Bulk Transaction Deletion**
   - Count confirmation dialog
   - Cannot be undone

4. **Import Revert**
   - Shows affected transaction count
   - Irreversible action

5. **Plaid Disconnection**
   - Warns about loss of sync capability
   - Historical data preserved

### Password-Protected Actions

1. Change email address
2. Enable/disable MFA
3. Generate API keys
4. Delete user account
5. Change password

## Security Considerations

### Authentication Security

1. **Password Requirements**
   - Minimum 8 characters
   - At least one uppercase letter
   - At least one lowercase letter
   - At least one number
   - At least one special character

2. **Session Management**
   - JWT access tokens (15 minute expiry)
   - Refresh tokens (7 day expiry, httpOnly cookie)
   - Session tracking with IP and user agent
   - Concurrent session support

3. **MFA Implementation**
   - TOTP-based (RFC 6238)
   - 30-second time step
   - 6 backup codes (one-time use)
   - Recovery via backup codes only

4. **Rate Limiting**
   - Login: 5 attempts per minute per IP
   - API: 100 requests per hour per key (configurable)
   - Password reset: 3 attempts per hour per email

### Data Security

1. **Encryption at Rest**
   - Plaid access tokens encrypted (ActiveRecord Encryption)
   - API key secrets hashed
   - OTP secrets encrypted

2. **Encryption in Transit**
   - HTTPS enforced in production
   - Secure WebSocket connections

3. **Data Isolation**
   - All queries scoped to family
   - Family ID validated on every request
   - No cross-family data access possible

### Input Validation

1. **SQL Injection Prevention**
   - Parameterized queries via ORM
   - No raw SQL with user input

2. **XSS Prevention**
   - Output encoding in templates
   - Content Security Policy headers
   - Sanitized user content

3. **CSRF Protection**
   - CSRF tokens for state-changing requests
   - SameSite cookie attribute

### Audit Trail

1. **Logged Events**
   - User login/logout
   - Failed login attempts
   - Password changes
   - MFA enable/disable
   - API key creation/revocation
   - Data exports
   - Impersonation sessions

2. **Impersonation Logging**
   - Every action logged with:
     - Controller/action
     - Path
     - HTTP method
     - IP address
     - User agent
     - Timestamp

### Security Headers

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' cdn.plaid.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' wss: https://api.openai.com https://production.plaid.com;
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```
