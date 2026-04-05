---
layout: default
title: Getting Started
parent: Guides
nav_order: 1
---

# Getting Started with Razem
{: .no_toc }

This guide walks you through setting up and integrating with the Razem platform.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

Before you begin, ensure you have:

- A Razem account (register at [razem.co.za](https://razem.co.za))
- An API key (available in your [My Razem Account]({% link docs/account/setup.md %}) portal)
- A supported HTTP client (e.g., `curl`, Postman, or any HTTP library)

---

## Step 1: Create Your Account

Visit the [My Razem Account]({% link docs/account/setup.md %}) portal to register. You will need:

1. A valid South African ID or passport number
2. A verified email address
3. A South African mobile number for OTP verification

After registration, verify your email and complete the KYC (Know Your Customer) process to unlock all platform features.

---

## Step 2: Obtain Your API Key

1. Log in to the [My Razem Account portal]({% link docs/account/setup.md %})
2. Navigate to **Settings → Developer → API Keys**
3. Click **Generate New API Key**
4. Copy and securely store your API key — it will only be shown once

{: .warning }
Never share your API key or commit it to version control. Store it in environment variables or a secrets manager.

---

## Step 3: Make Your First API Call

Test your credentials with a simple account info request:

```bash
curl -X GET https://api.razem.co.za/v1/account \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

**Expected response:**

```json
{
  "id": "acc_1234567890",
  "name": "John Doe",
  "email": "john@example.com",
  "status": "active",
  "balance": {
    "available": 5000.00,
    "currency": "ZAR"
  },
  "created_at": "2025-01-15T10:30:00Z"
}
```

---

## Step 4: Explore the API

Now that you're authenticated, explore what Razem can do:

| Feature | Endpoint | Description |
|:---|:---|:---|
| Account Info | `GET /v1/account` | Get account details and balance |
| Transactions | `GET /v1/transactions` | List transaction history |
| Send Money | `POST /v1/payments` | Initiate a payment |
| Beneficiaries | `GET /v1/beneficiaries` | Manage saved beneficiaries |
| Statements | `GET /v1/statements` | Download account statements |

See the full [API Reference]({% link docs/api/index.md %}) for all available endpoints.

---

## SDK & Client Libraries

Razem provides official client libraries for popular programming languages:

{: .note }
Official SDK packages are currently in development. In the meantime, any HTTP client can be used with the REST API.

**Using JavaScript/Node.js:**

```javascript
const axios = require('axios');

const razem = axios.create({
  baseURL: 'https://api.razem.co.za/v1',
  headers: {
    'Authorization': `Bearer ${process.env.RAZEM_API_KEY}`,
    'Content-Type': 'application/json'
  }
});

// Get account info
const account = await razem.get('/account');
console.log(account.data);
```

**Using Python:**

```python
import requests
import os

headers = {
    'Authorization': f'Bearer {os.environ["RAZEM_API_KEY"]}',
    'Content-Type': 'application/json'
}

# Get account info
response = requests.get('https://api.razem.co.za/v1/account', headers=headers)
print(response.json())
```

---

## Environment URLs

| Environment | Base URL | Description |
|:---|:---|:---|
| **Production** | `https://api.razem.co.za/v1` | Live production environment |
| **Staging** | `https://api-staging.razem.co.za/v1` | Pre-production testing |
| **Sandbox** | `https://api-sandbox.razem.co.za/v1` | Development and testing (no real money) |

{: .highlight }
Always use the **Sandbox** environment for development and testing. The sandbox uses test credentials and no real funds are moved.

---

## Next Steps

- [Authentication Deep Dive]({% link docs/api/authentication.md %})
- [Code Examples & Integrations]({% link docs/guides/examples.md %})
- [Webhooks Setup]({% link docs/guides/webhooks.md %})
- [Error Handling]({% link docs/reference/errors.md %})
