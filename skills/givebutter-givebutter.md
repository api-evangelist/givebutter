---
name: Givebutter
description: Use when building donation widgets for websites, managing fundraising campaigns via API, creating and updating donor contacts, recording transactions, setting up webhooks for real-time events, or integrating Givebutter with external systems. Agents should reach for this skill when working with fundraising infrastructure, donor management, campaign automation, or payment processing.
metadata:
    mintlify-proj: givebutter
    version: "1.0"
---

# Givebutter Skill

## Product Summary

Givebutter is a fundraising and donor management platform with two primary integration paths: **Widgets** for embedding donation forms directly on websites, and a **REST API** for programmatic access to campaigns, contacts, transactions, and webhooks. Use the Widgets library to add donation buttons, forms, goal bars, and signup forms to any website with minimal code. Use the API to automate donor workflows, manage campaigns, record transactions, and listen for real-time events. Key files and endpoints: API base URL is `https://api.givebutter.com/v1/`, authentication uses Bearer tokens, and widgets load from `https://widgets.givebutter.com/latest.umd.cjs`. See [https://docs.givebutter.com](https://docs.givebutter.com) for complete documentation.

## When to Use

- **Embedding donation widgets**: Add donation buttons, forms, goal bars, or email signup forms to websites (WordPress, Wix, Squarespace, custom HTML)
- **Managing campaigns programmatically**: Create, update, list, or delete fundraising campaigns via API
- **Donor/contact management**: Create, update, list, tag, or restore donor contacts; manage contact activities and histories
- **Recording transactions**: Log donations, payments, or offline contributions; update transaction details
- **Real-time event handling**: Set up webhooks to listen for campaign creation, transaction success, contact updates, plan changes, or refunds
- **Discount code management**: Create and manage campaign-specific discount codes
- **Reporting and analytics**: Query transactions, contacts, campaigns, and payouts with filtering and pagination
- **Integration workflows**: Connect Givebutter data to external CRMs, accounting systems, or analytics platforms

## Quick Reference

### API Authentication
```bash
# All API requests require Bearer token in Authorization header
curl https://api.givebutter.com/v1/campaigns \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Get API Key**: Settings → Integrations → API Keys in Givebutter Dashboard. Keys are shown only once—store securely and never commit to version control.

### Widget Installation
```html
<!-- Add to <head> of every page -->
<script async src="https://widgets.givebutter.com/latest.umd.cjs?acct=YOUR_ACCOUNT_ID"></script>

<!-- Then embed widgets anywhere on page -->
<givebutter-button campaign="YOUR_CAMPAIGN_CODE"></givebutter-button>
<givebutter-giving-form campaign="YOUR_CAMPAIGN_CODE"></givebutter-giving-form>
<givebutter-goal-bar campaign="YOUR_CAMPAIGN_CODE"></givebutter-goal-bar>
<givebutter-signup-form account="YOUR_ACCOUNT_ID"></givebutter-signup-form>
```

### Core API Endpoints

| Resource | Methods | Key Endpoints |
|----------|---------|---------------|
| **Campaigns** | GET, POST, PUT, DELETE | `/v1/campaigns`, `/v1/campaigns/{id}` |
| **Contacts** | GET, POST, PUT, DELETE, PATCH | `/v1/contacts`, `/v1/contacts/{id}`, `/v1/contacts/{id}/restore` |
| **Transactions** | GET, POST, PUT | `/v1/transactions`, `/v1/transactions/{id}` |
| **Contact Activities** | GET, POST, PUT, DELETE | `/v1/contacts/{id}/activities` |
| **Contact Tags** | POST | `/v1/contacts/{id}/tags/add`, `/v1/contacts/{id}/tags/remove` |
| **Webhooks** | GET, POST, PUT, DELETE | `/v1/webhooks`, `/v1/webhooks/{id}` |
| **Funds** | GET, POST, PUT, DELETE | `/v1/funds`, `/v1/funds/{id}` |
| **Households** | GET, POST, PUT, DELETE | `/v1/households`, `/v1/households/{id}` |
| **Discount Codes** | GET, POST, PUT, DELETE | `/v1/campaigns/{id}/discount-codes` |

### Pagination
All list endpoints return paginated responses with `data`, `links`, and `meta` objects. Default page size is 20, max is 100.

```bash
# Request specific page and size
curl "https://api.givebutter.com/v1/campaigns?page=2&per_page=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Use `links.next` to iterate through all pages until it becomes `null`.

### Rate Limits
- **500 requests per minute** across all API endpoints
- Exceeding limit returns `429 Too Many Requests` with `Retry-After` header
- Implement exponential backoff when rate limited

### Webhook Events
Available events for webhook subscriptions:
- `campaign.created`, `campaign.updated`, `campaign.published`
- `transaction.succeeded`
- `contact.created`, `contact.updated`
- `plan.created`, `plan.updated`, `plan.canceled`, `plan.failed`, `plan.paused`, `plan.resumed`
- `ticket.created`
- `refund.created`

## Decision Guidance

| Scenario | Use Widgets | Use API |
|----------|------------|---------|
| Add donation button to website | ✓ | - |
| Embed full donation form on page | ✓ | - |
| Show fundraising progress bar | ✓ | - |
| Automate donor workflows | - | ✓ |
| Sync contacts to external CRM | - | ✓ |
| Record offline donations | - | ✓ |
| Listen for real-time events | - | ✓ |
| Manage campaigns programmatically | - | ✓ |
| Build custom donation UI | - | ✓ |

| Contact Type | Required Fields | Notes |
|--------------|-----------------|-------|
| Individual | `first_name`, `last_name`, `type: "individual"` | Can include emails, phones, addresses |
| Company | `company_name`, `type: "company"` | Represents organization, not person |

| Transaction Method | Use Case |
|-------------------|----------|
| `card` | Credit/debit card payments |
| `ach` | Bank transfer (ACH) |
| `check` | Physical check (offline) |
| `cash` | Cash donation (offline) |
| `paypal`, `venmo`, `cashapp` | Digital wallet payments |
| `stock`, `property`, `daf` | Non-cash gifts |

## Workflow

### 1. Embed a Donation Widget on Website
1. Find your Account ID: Dashboard → Settings → Integrations
2. Add library script to website `<head>`: `<script async src="https://widgets.givebutter.com/latest.umd.cjs?acct=YOUR_ACCOUNT_ID"></script>`
3. Find your Campaign Code: Campaign page → top section (6-character code)
4. Embed widget tag: `<givebutter-button campaign="YOUR_CAMPAIGN_CODE"></givebutter-button>`
5. Test on live site (widgets don't render in editor for some platforms)

### 2. Create and Manage a Campaign via API
1. Get API key from Dashboard → Settings → Integrations → API Keys
2. Create campaign: `POST /v1/campaigns` with `type`, `title`, `goal`, `timezone`
3. Update campaign: `PUT /v1/campaigns/{id}` with new settings
4. List campaigns: `GET /v1/campaigns` with optional `scope` filter
5. Verify response includes `id`, `code`, `raised`, `donors`, `status`

### 3. Record a Donation Transaction
1. Identify contact: use existing contact ID or create new contact first
2. Create transaction: `POST /v1/transactions` with:
   - `method` (card, ach, check, cash, etc.)
   - `amount` (required)
   - `transacted_at` (ISO 8601 datetime)
   - `campaign_code` or `campaign_id` (optional)
   - `contact_id` or contact details (first_name, last_name, email)
3. Verify response includes transaction `id`, `amount`, `status`
4. Update if needed: `PUT /v1/transactions/{id}` for notes, check info, or dedications

### 4. Set Up a Webhook for Real-Time Events
1. Prepare endpoint: Create HTTPS endpoint that accepts POST requests
2. Create webhook: `POST /v1/webhooks` with:
   - `url` (your endpoint)
   - `events` (array of event types to listen for)
   - `name` (optional, for identification)
   - `enabled: true`
3. Test webhook: Trigger event in Givebutter (e.g., make donation)
4. Verify: Check webhook activities at `GET /v1/webhooks/{id}/activities`
5. Update if needed: `PUT /v1/webhooks/{id}` to change events or URL

### 5. Manage Donor Contacts
1. Create contact: `POST /v1/contacts` with `type` (individual/company), name, emails, phones, addresses
2. Update contact: `PUT /v1/contacts/{id}` with new fields (partial updates allowed)
3. Add tags: `POST /v1/contacts/{id}/tags/add` with array of tag strings
4. Remove tags: `POST /v1/contacts/{id}/tags/remove` with array of tag strings
5. Log activity: `POST /v1/contacts/{id}/activities` with `type` (email, meeting, note, phone_call, sms, etc.)
6. Restore deleted: `PATCH /v1/contacts/{id}/restore` if contact was archived

## Common Gotchas

- **API key format**: Must use `Authorization: Bearer YOUR_API_KEY`, not just `Authorization: YOUR_API_KEY`. Missing "Bearer" returns 401.
- **Campaign code vs ID**: Campaign code is 6-character string (e.g., "abc123"), ID is numeric. Use code for widgets, either for API.
- **Account ID vs Campaign Code**: Account ID is for signup forms and library loading; Campaign Code is for donation widgets. Don't mix them.
- **Contact type is immutable**: Once created as `individual` or `company`, cannot change type. Delete and recreate if needed.
- **Individual contacts require name**: `first_name` and `last_name` are required for `type: "individual"`. Company contacts require `company_name` instead.
- **Pagination is 1-indexed**: Page numbers start at 1, not 0. `page=1` is first page.
- **Max per_page is 100**: Requesting `per_page > 100` returns error. Use pagination loop instead of large single request.
- **Webhook signature verification**: Webhooks include `signature` field for verification. Validate to prevent spoofing.
- **Transacted_at timezone**: Provide `timezone` field when creating transactions to ensure correct timestamp interpretation.
- **Deleted contacts cannot be queried**: Deleted contacts don't appear in list endpoints. Use restore endpoint if needed.
- **Tags are case-sensitive**: "Major Donor" and "major donor" are different tags.
- **Discount code uses field**: `uses` field tracks how many times code was used; `uses: null` means unlimited.
- **Offline transactions need method**: Check payments require `check_number` and optionally `check_deposited_at`.
- **Rate limit resets per minute**: 500 requests per minute is a rolling window, not per calendar minute.
- **Webhook events are case-sensitive**: Use exact event names like `transaction.succeeded`, not `transaction_succeeded`.

## Verification Checklist

Before submitting work with Givebutter:

- [ ] API key is stored securely (not in code, not in version control)
- [ ] All API requests include `Authorization: Bearer YOUR_API_KEY` header
- [ ] Campaign code (6 characters) is correct for widgets; campaign ID is correct for API calls
- [ ] Contact type (`individual` or `company`) matches required fields (name vs company_name)
- [ ] Pagination loop handles `links.next` correctly and stops when `next` is `null`
- [ ] Webhook endpoint is HTTPS and responds with 2xx status within timeout
- [ ] Transaction `method` is valid enum value (card, ach, check, cash, paypal, etc.)
- [ ] DateTime fields use ISO 8601 format (e.g., `2024-01-15T10:30:00Z`)
- [ ] Webhook events array uses exact event names (e.g., `transaction.succeeded`)
- [ ] Error responses are handled: check for 401 (auth), 403 (permissions), 404 (not found), 422 (validation)
- [ ] Rate limiting is handled: implement retry logic for 429 responses
- [ ] Contact/campaign IDs are numeric; campaign codes are 6-character strings

## Resources

- **Complete API documentation**: [https://docs.givebutter.com/llms.txt](https://docs.givebutter.com/llms.txt) — comprehensive page-by-page navigation
- **API Reference**: [https://docs.givebutter.com/api-reference/authentication](https://docs.givebutter.com/api-reference/authentication) — authentication, pagination, rate limits, error codes
- **Widgets Getting Started**: [https://docs.givebutter.com/widgets/getting-started](https://docs.givebutter.com/widgets/getting-started) — embed donation forms on websites
- **Webhooks Overview**: [https://docs.givebutter.com/webhooks/getting-started](https://docs.givebutter.com/webhooks/getting-started) — set up real-time event listeners

---

> For additional documentation and navigation, see: https://docs.givebutter.com/llms.txt