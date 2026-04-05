---
layout: default
title: Webhooks
parent: Guides
nav_order: 3
---

# Webhooks
{: .no_toc }

Receive real-time notifications for events happening in your Razem account.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

Webhooks allow Razem to push real-time event notifications to your application. When an event occurs — such as a payment being completed or a new deposit arriving — Razem sends an HTTP POST request to your configured webhook URL.

---

## Setting Up Webhooks

### 1. Create a Public Endpoint

Your webhook endpoint must be:
- Publicly accessible over HTTPS
- Return a `2xx` status code within 10 seconds
- Idempotent (safe to receive the same event multiple times)

### 2. Register Your Webhook

```bash
curl -X POST https://api.razem.co.za/v1/webhooks \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://yourapp.com/webhooks/razem",
    "events": ["payment.completed", "payment.failed"],
    "secret": "your_signing_secret"
  }'
```

### 3. Verify the Signature

Every webhook request includes an `X-Razem-Signature` header. Always verify this before processing:

```python
import hmac
import hashlib

def verify_signature(payload_bytes, signature, secret):
    expected = 'sha256=' + hmac.new(
        secret.encode(),
        payload_bytes,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

---

## Event Types

| Event | Description |
|:---|:---|
| `payment.created` | A new payment has been initiated |
| `payment.completed` | A payment completed successfully |
| `payment.failed` | A payment failed |
| `payment.reversed` | A payment was reversed |
| `deposit.received` | Funds received into account |
| `account.balance_low` | Balance dropped below threshold |
| `account.frozen` | Account was frozen |
| `account.unfrozen` | Account was unfrozen |
| `beneficiary.added` | New beneficiary added |
| `kyc.approved` | KYC verification approved |
| `kyc.rejected` | KYC verification rejected |

---

## Webhook Payload

```json
{
  "id": "evt_1a2b3c4d5e",
  "event": "payment.completed",
  "created_at": "2025-04-05T10:30:00Z",
  "data": {
    "payment_id": "pay_9x8y7z6w",
    "amount": 500.00,
    "currency": "ZAR",
    "status": "completed",
    "reference": "INV-2025-001",
    "completed_at": "2025-04-05T10:30:00Z"
  }
}
```

---

## Retry Policy

If your endpoint returns a non-`2xx` status or times out, Razem retries delivery:

| Attempt | Delay |
|:---|:---|
| 1st retry | 5 minutes |
| 2nd retry | 30 minutes |
| 3rd retry | 2 hours |
| 4th retry | 8 hours |
| 5th retry | 24 hours |

After 5 failed attempts, the webhook is marked as failed and no further retries are made. You can manually redeliver failed events from the My Razem Account portal.

---

## Testing Webhooks

Use the Razem sandbox environment to test your webhook integration:

```bash
# Send a test webhook event
curl -X POST https://api-sandbox.razem.co.za/v1/webhooks/test \
  -H "Authorization: Bearer YOUR_SANDBOX_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook_id": "wh_abc123",
    "event": "payment.completed"
  }'
```

For local development, use a tunneling tool like [ngrok](https://ngrok.com) to expose your local server.
