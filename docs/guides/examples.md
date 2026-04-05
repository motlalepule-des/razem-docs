---
layout: default
title: Code Examples
parent: Guides
nav_order: 2
---

# Code Examples
{: .no_toc }

Practical examples for common Razem integration patterns.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Authentication

```bash
# Authenticate and get a session token
curl -X POST https://api.razem.co.za/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "api_secret": "YOUR_API_SECRET"
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

---

## Account Operations

### Get Account Balance

```javascript
const response = await razem.get('/account/balance');

// Response:
// {
//   "available": 12500.50,
//   "pending": 300.00,
//   "currency": "ZAR",
//   "last_updated": "2025-04-05T08:00:00Z"
// }
```

### Get Transaction History

```python
params = {
    'page': 1,
    'limit': 20,
    'from_date': '2025-01-01',
    'to_date': '2025-04-05',
    'type': 'debit'  # 'debit', 'credit', or omit for all
}

response = requests.get(
    'https://api.razem.co.za/v1/transactions',
    headers=headers,
    params=params
)

transactions = response.json()
```

---

## Payments

### Send Money to a Beneficiary

```javascript
const payment = await razem.post('/payments', {
  beneficiary_id: 'ben_abc123',
  amount: 500.00,
  currency: 'ZAR',
  reference: 'Invoice #INV-2025-001',
  description: 'Payment for services rendered'
});

console.log(`Payment ${payment.data.id} created with status: ${payment.data.status}`);
```

### Instant EFT Payment

```bash
curl -X POST https://api.razem.co.za/v1/payments/eft \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "recipient": {
      "account_number": "1234567890",
      "bank_code": "632005",
      "account_type": "current",
      "name": "Acme Corp"
    },
    "amount": 1500.00,
    "currency": "ZAR",
    "reference": "PAY-20250405-001",
    "notification_email": "payments@acme.co.za"
  }'
```

---

## Webhooks

### Register a Webhook

```python
webhook = requests.post(
    'https://api.razem.co.za/v1/webhooks',
    headers=headers,
    json={
        'url': 'https://yourapp.com/webhooks/razem',
        'events': [
            'payment.completed',
            'payment.failed',
            'account.balance_low'
        ],
        'secret': 'your_webhook_secret_here'
    }
)
```

### Verify Webhook Signature (Node.js)

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(`sha256=${expected}`)
  );
}

// In your webhook handler:
app.post('/webhooks/razem', (req, res) => {
  const signature = req.headers['x-razem-signature'];
  const isValid = verifyWebhookSignature(req.body, signature, WEBHOOK_SECRET);
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process the event
  const { event, data } = req.body;
  console.log(`Received event: ${event}`);
  
  res.status(200).json({ received: true });
});
```

---

## Error Handling

```javascript
try {
  const payment = await razem.post('/payments', paymentData);
} catch (error) {
  if (error.response) {
    const { status, data } = error.response;
    
    switch (status) {
      case 400:
        console.error('Invalid request:', data.errors);
        break;
      case 401:
        console.error('Unauthorized - check your API key');
        break;
      case 402:
        console.error('Insufficient funds:', data.message);
        break;
      case 422:
        console.error('Validation error:', data.errors);
        break;
      case 429:
        const retryAfter = error.response.headers['retry-after'];
        console.error(`Rate limited. Retry after ${retryAfter} seconds`);
        break;
      case 500:
        console.error('Server error - please try again later');
        break;
    }
  }
}
```
