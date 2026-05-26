**Accept payments, manage subscriptions, and automate billing workflows all through natural language.**

A Model Context Protocol (MCP) server that exposes Stripe's API for payment processing, customer management, subscriptions, and financial operations.


## Overview

The Stripe MCP Server provides a complete interface to Stripe's payment infrastructure:

- Full customer lifecycle management — create, search, update, and delete customers
- Payment processing via charges and payment intents
- Product & pricing catalog management
- Subscription billing with trial support
- Real-time account balance visibility

Perfect for:

- Automating billing workflows and subscription management without writing code
- Looking up customer payment history and account status through AI assistants
- Setting up products and pricing plans for new features or services


## Tools

<details>
<summary><code>stripe_create_customer</code> — Create a new Stripe customer</summary>

Creates a customer record in Stripe that can hold payment methods, subscriptions, and invoices.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key (starts with sk_)
- `email` (string, optional) — Customer's email address
- `name` (string, optional) — Customer's full name
- `description` (string, optional) — Internal description for the customer
- `metadata` (string, optional) — JSON string of metadata key-value pairs (e.g. '{"plan": "enterprise"}')
```

**Output:**
```json
{
  "success": true,
  "customer_id": "cus_xxx",
  "email": "user@example.com",
  "name": "Jane Doe",
  "description": "Enterprise customer",
  "created": 1716720000,
  "metadata": {}
}
```
</details>


<details>
<summary><code>stripe_get_customer</code> — Retrieve a customer by ID</summary>

Returns full customer details including balance, delinquency status, and metadata.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `customer_id` (string, required) — Customer ID (e.g. cus_xxx)
```

**Output:**
```json
{
  "success": true,
  "customer_id": "cus_xxx",
  "email": "user@example.com",
  "name": "Jane Doe",
  "balance": 0,
  "delinquent": false,
  "created": 1716720000,
  "metadata": {}
}
```
</details>


<details>
<summary><code>stripe_list_customers</code> — List customers with pagination</summary>

Returns up to 100 customers per page with cursor-based pagination.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `limit` (integer, optional) — Number of customers to return, 1–100 (default: 10)
- `starting_after` (string, optional) — Customer ID to paginate from
```

**Output:**
```json
{
  "success": true,
  "has_more": true,
  "count": 10,
  "customers": [{ "id": "cus_xxx", "email": "...", "name": "...", "created": 1716720000 }]
}
```
</details>


<details>
<summary><code>stripe_search_customers</code> — Search customers by email or name</summary>

Uses Stripe's search query language to find customers matching specific criteria.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `query` (string, required) — Search query (e.g. 'email:"user@example.com"' or 'name:"Jane"')
- `limit` (integer, optional) — Number of results, 1–100 (default: 10)
```

**Output:**
```json
{
  "success": true,
  "has_more": false,
  "count": 1,
  "customers": [{ "id": "cus_xxx", "email": "user@example.com", "name": "Jane Doe" }]
}
```
</details>


<details>
<summary><code>stripe_create_charge</code> — Charge a customer's payment method</summary>

Creates a direct charge on a customer. Amount is in the smallest currency unit (cents for USD).

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `amount` (integer, required) — Amount in cents (e.g. 1000 = $10.00)
- `currency` (string, optional) — Currency code: usd, eur, gbp, etc. (default: usd)
- `customer_id` (string, optional) — Customer ID to charge
- `description` (string, optional) — Description of the charge
- `metadata` (string, optional) — JSON string of metadata
```

**Output:**
```json
{
  "success": true,
  "charge_id": "ch_xxx",
  "amount": 1000,
  "currency": "usd",
  "status": "succeeded",
  "paid": true
}
```
</details>


<details>
<summary><code>stripe_create_payment_intent</code> — Create a payment intent</summary>

Creates a payment intent and returns a `client_secret` for frontend confirmation via Stripe.js.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `amount` (integer, required) — Amount in cents
- `currency` (string, optional) — Currency code (default: usd)
- `customer_id` (string, optional) — Customer ID
- `description` (string, optional) — Description
- `metadata` (string, optional) — JSON string of metadata
- `confirm` (boolean, optional) — Confirm immediately (default: false)
```

**Output:**
```json
{
  "success": true,
  "payment_intent_id": "pi_xxx",
  "client_secret": "pi_xxx_secret_xxx",
  "amount": 2000,
  "currency": "usd",
  "status": "requires_payment_method"
}
```
</details>


<details>
<summary><code>stripe_create_product</code> — Create a product</summary>

Creates a product in your Stripe catalog. Products require a price before they can be sold.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `name` (string, required) — Product name
- `description` (string, optional) — Product description
- `metadata` (string, optional) — JSON string of metadata
```

**Output:**
```json
{
  "success": true,
  "product_id": "prod_xxx",
  "name": "Pro Plan",
  "description": "Access to all pro features",
  "active": true
}
```
</details>


<details>
<summary><code>stripe_create_price</code> — Create a price for a product</summary>

Defines the billing amount and interval for a product. Supports one-time and recurring (subscription) pricing.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `product_id` (string, required) — Product ID to attach the price to
- `unit_amount` (integer, required) — Amount in cents
- `currency` (string, optional) — Currency code (default: usd)
- `recurring_interval` (string, optional) — 'month' or 'year' for subscription pricing
```

**Output:**
```json
{
  "success": true,
  "price_id": "price_xxx",
  "product": "prod_xxx",
  "unit_amount": 999,
  "currency": "usd",
  "recurring": { "interval": "month" }
}
```
</details>


<details>
<summary><code>stripe_create_subscription</code> — Subscribe a customer to a plan</summary>

Creates a recurring subscription for a customer using a price ID. Supports trial periods.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `customer_id` (string, required) — Customer ID
- `price_id` (string, required) — Price ID for the subscription plan
- `trial_period_days` (integer, optional) — Number of free trial days
- `metadata` (string, optional) — JSON string of metadata
```

**Output:**
```json
{
  "success": true,
  "subscription_id": "sub_xxx",
  "customer": "cus_xxx",
  "status": "active",
  "current_period_start": 1716720000,
  "current_period_end": 1719312000
}
```
</details>


<details>
<summary><code>stripe_cancel_subscription</code> — Cancel a subscription</summary>

Cancels a subscription either at the end of the current billing period or immediately.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
- `subscription_id` (string, required) — Subscription ID to cancel
- `cancel_at_period_end` (boolean, optional) — true = cancel at period end, false = cancel immediately (default: true)
```

**Output:**
```json
{
  "success": true,
  "subscription_id": "sub_xxx",
  "status": "active",
  "cancel_at_period_end": true
}
```
</details>


<details>
<summary><code>stripe_get_balance</code> — Get account balance</summary>

Returns available and pending balances across all currencies for the Stripe account.

**Inputs:**
```
- `api_key` (string, required) — Stripe API key
```

**Output:**
```json
{
  "success": true,
  "available": [{ "amount": 50000, "currency": "usd" }],
  "pending": [{ "amount": 12000, "currency": "usd" }],
  "livemode": true
}
```
</details>


## Getting Your Stripe API Key

<details>
<summary><strong>Steps</strong></summary>

1. Go to [Stripe Dashboard](https://dashboard.stripe.com)
2. Click **Developers** in the top navigation
3. Select **API keys** from the left sidebar
4. Copy your **Secret key** (starts with `sk_live_` for production or `sk_test_` for testing)

> Use `sk_test_` keys during development — they work with test card numbers and won't charge real cards.

</details>


## Troubleshooting

<details>
<summary><strong>Missing or Invalid Headers</strong></summary>

- **Cause:** API key not provided in request headers or incorrect format
- **Solution:**
  1. Verify `Authorization: Bearer YOUR_API_KEY` and `X-Mewcp-Credential-Id: CREDENTIAL-ID` headers are present
  2. Check API key is active in your MewCP account

</details>

<details>
<summary><strong>Insufficient Credits</strong></summary>

- **Cause:** API calls have exceeded your request limits
- **Solution:**
  1. Check credit usage in your Curious Layer dashboard
  2. Upgrade to a paid plan or add credits for higher limits
  3. Contact support for credit adjustments

</details>

<details>
<summary><strong>Credential Not Connected</strong></summary>

- **Cause:** No Stripe credential linked to your account
- **Solution:**
  1. Go to **Credentials** in your MewCP dashboard
  2. Add your Stripe API key (static auth)
  3. Retry the request with the correct `X-Mewcp-Credential-Id` header

</details>

<details>
<summary><strong>Malformed Request Payload</strong></summary>

- **Cause:** JSON payload is invalid or missing required fields
- **Solution:**
  1. Validate JSON syntax before sending
  2. Ensure all required tool parameters are included
  3. Check parameter types match expected values

</details>

<details>
<summary><strong>Server Not Found</strong></summary>

- **Cause:** Incorrect server name in the API endpoint
- **Solution:**
  1. Verify endpoint format: `{server-name}/mcp/{tool-name}`
  2. Use correct server name from documentation
  3. Check available servers in your Curious Layer account

</details>

<details>
<summary><strong>Stripe API Error</strong></summary>

- **Cause:** Upstream Stripe API returned an error
- **Solution:**
  1. Check Stripe service status at [Stripe Status Page](https://status.stripe.com)
  2. Verify your API key has the required permissions (live vs. test mode)
  3. Review the error message for specific details — Stripe errors are descriptive

</details>

---

<details>
<summary><strong>Resources</strong></summary>

- **[Stripe API Documentation](https://stripe.com/docs/api)** — Official API reference
- **[Stripe API Reference](https://stripe.com/docs/api/authentication)** — Complete endpoint reference
- **[FastMCP Docs](https://gofastmcp.com/v2/getting-started/welcome)** — FastMCP specification
- **[FastMCP Credentials](https://pypi.org/project/fastmcp-credentials/)** — FastMCP Credentials package for credential handling

</details>
