# Non-Functional Requirements

## Performance Expectations

### Response Time Targets

| Operation | Target | Acceptable |
|-----------|--------|------------|
| Page load (initial) | < 1s | < 2s |
| Page navigation (SPA) | < 200ms | < 500ms |
| API response (simple) | < 100ms | < 300ms |
| API response (complex query) | < 500ms | < 1s |
| Transaction list (1000 items) | < 500ms | < 1s |
| Balance calculation | < 2s | < 5s |
| Import (1000 rows) | < 30s | < 60s |

### Database Performance

1. **Query Optimization**
   - All queries must use indexes
   - No N+1 queries (use eager loading)
   - Large result sets paginated (max 100 per page)

2. **Index Strategy**
   - Foreign keys indexed
   - Frequently filtered columns indexed
   - Composite indexes for common query patterns

3. **Connection Pooling**
   - Pool size: 10 connections (configurable)
   - Connection timeout: 5 seconds

### Caching Strategy

1. **Redis Caching**
   - Session storage
   - Rate limit counters
   - Expensive calculation results (balance sheets)
   - Cache TTL: 5 minutes for financial data

2. **HTTP Caching**
   - ETag headers for conditional requests
   - Cache-Control for static assets

## Logging and Monitoring

### Log Levels

| Level | Use Case |
|-------|----------|
| ERROR | Exceptions, failed operations |
| WARN | Deprecated usage, recoverable errors |
| INFO | Request/response, significant events |
| DEBUG | Detailed debugging (disabled in production) |

### Log Format

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "INFO",
  "message": "Transaction created",
  "context": {
    "requestId": "abc-123",
    "userId": "user-uuid",
    "familyId": "family-uuid",
    "transactionId": "txn-uuid"
  }
}
```

### Metrics to Track

1. **Application Metrics**
   - Request count by endpoint
   - Response time percentiles (p50, p95, p99)
   - Error rate by endpoint
   - Active sessions count

2. **Business Metrics**
   - Accounts created per day
   - Transactions imported per day
   - Plaid connections per day
   - AI chat messages per day

3. **Infrastructure Metrics**
   - CPU/memory usage
   - Database connection count
   - Redis memory usage
   - Job queue depth

### Error Tracking

1. **Sentry Integration**
   - Automatic exception capture
   - User context attached
   - Source maps for frontend
   - Performance tracing

2. **Alert Conditions**
   - Error rate > 1% (warning)
   - Error rate > 5% (critical)
   - p95 response time > 2s (warning)
   - Job queue depth > 100 (warning)

## Accessibility Requirements

### WCAG 2.1 AA Compliance

1. **Perceivable**
   - Color contrast ratio >= 4.5:1 for text
   - Alt text for images
   - Captions for video (if any)

2. **Operable**
   - Full keyboard navigation
   - Focus indicators visible
   - Skip navigation links
   - No keyboard traps

3. **Understandable**
   - Clear error messages
   - Form labels associated with inputs
   - Consistent navigation

4. **Robust**
   - Valid HTML
   - ARIA attributes where needed
   - Works with screen readers

### Specific Requirements

1. **Form Accessibility**
   - Label associated with every input
   - Error messages linked to fields
   - Required fields indicated

2. **Table Accessibility**
   - Header cells marked with `<th>`
   - Scope attributes on headers
   - Caption for data tables

3. **Modal Accessibility**
   - Focus trapped in modal
   - ESC closes modal
   - Focus returns to trigger on close

4. **Chart Accessibility**
   - Text alternative describing data
   - Data table available as alternative
   - High contrast mode support

## SEO and SSR Considerations

### Nuxt SSR Configuration

1. **Server-Side Rendering**
   - All public pages rendered on server
   - Meta tags set dynamically
   - Structured data where appropriate

2. **Client-Side Only**
   - Dashboard (requires auth)
   - Interactive charts
   - Modal content

### Meta Tags

```html
<!-- Standard -->
<title>Maybe - Personal Finance</title>
<meta name="description" content="Track your finances, budget, and investments">

<!-- Open Graph -->
<meta property="og:title" content="Maybe - Personal Finance">
<meta property="og:description" content="Track your finances">
<meta property="og:image" content="/og-image.png">
<meta property="og:type" content="website">

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image">
```

### URL Structure

- Clean URLs without query strings where possible
- Consistent slug format (lowercase, hyphens)
- Canonical URLs set

### Sitemap

- Public pages only (marketing, docs)
- Updated on deploy
- Submitted to search engines

## Mobile Responsiveness

### Breakpoints

| Name | Width | Description |
|------|-------|-------------|
| sm | 640px | Small phones |
| md | 768px | Large phones, small tablets |
| lg | 1024px | Tablets, small laptops |
| xl | 1280px | Laptops, desktops |
| 2xl | 1536px | Large desktops |

### Mobile-Specific UX

1. **Navigation**
   - Bottom navigation on mobile
   - Hamburger menu for secondary items
   - Collapsible sidebar

2. **Tables**
   - Horizontal scroll on small screens
   - Card view alternative for transactions

3. **Forms**
   - Full-width inputs
   - Large touch targets (44x44px minimum)
   - Native date pickers on mobile

4. **Charts**
   - Simplified on mobile
   - Touch-friendly interactions
   - Reduced data points for performance
