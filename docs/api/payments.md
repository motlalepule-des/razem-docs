---
layout: default
title: Payments
parent: API Reference
nav_order: 3
---

# Payments API
{: .no_toc }

Initiate and manage payments, transfers, and transactions.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Create Payment

Initiate a payment to a beneficiary or bank account.

```
POST /v1/payments
```

### Request Body

```json
{
  "type": "eft",
  "amount": 500.00,
  "currency": "ZAR",
  "reference": "INV-2025-001",
  "description": "Payment for consulting services",
  "recipient": {
    "beneficiary_id": "ben_abc123"
  }
}
```

Or pay directly to a bank account:

```json
{
  "type": "eft",
  "amount": 500.00,
  "currency": "ZAR",
  "reference": "INV-2025-001",
  "recipient": {
    "name": "Acme Corp",
    "account_number": "1234567890",
    "bank_code": "632005",
    "account_type": "current"
  }
}
```

### Response

```json
{
  "id": "pay_9x8y7z6w",
  "status": "pending",
  "type": "eft",
  "amount": 500.00,
  "currency": "ZAR",
  "reference": "INV-2025-001",
  "recipient": {
    "name": "Acme Corp",
    "account_number": "****7890"
  },
  "fee": 2.50,
  "estimated_completion": "2025-04-05T16:00:00Z",
  "created_at": "2025-04-05T10:30:00Z"
}
```

---

## Get Payment

Retrieve details of a specific payment.

```
GET /v1/payments/{payment_id}
```

---

## List Payments

```
GET /v1/payments
```

**Query Parameters:**

| Parameter | Type | Description |
|:---|:---|:---|
| `status` | string | Filter by status |
| `from_date` | date | Start date (ISO 8601) |
| `to_date` | date | End date (ISO 8601) |
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Results per page (default: 20, max: 100) |

---

## Cancel Payment

Cancel a pending payment before it is processed.

```
DELETE /v1/payments/{payment_id}
```

{: .note }
Only payments in `pending` status can be cancelled. Once processing begins, contact Razem support for reversals.

---

## Payment Types

| Type | Description | Processing Time |
|:---|:---|:---|
| `eft` | Electronic Funds Transfer | 1-2 business days |
| `rtp` | Real-Time Payment | Immediate (24/7) |
| `internal` | Transfer between Razem accounts | Immediate |
| `batch` | Bulk payment batch | 1-2 business days |

---

## Payment Status Values

| Status | Description |
|:---|:---|
| `pending` | Payment queued for processing |
| `processing` | Payment is being processed |
| `completed` | Payment successfully delivered |
| `failed` | Payment could not be completed |
| `cancelled` | Payment was cancelled |
| `reversed` | Payment was reversed after completion |

---

## Transactions

### List Transactions

```
GET /v1/transactions
```

**Query Parameters:**

| Parameter | Type | Description |
|:---|:---|:---|
| `type` | string | `debit`, `credit`, or omit for all |
| `from_date` | date | Start date (ISO 8601) |
| `to_date` | date | End date (ISO 8601) |
| `min_amount` | number | Minimum transaction amount |
| `max_amount` | number | Maximum transaction amount |
| `page` | integer | Page number |
| `limit` | integer | Results per page (max: 100) |

### Get Transaction

```
GET /v1/transactions/{transaction_id}
```

### Transaction Object

```json
{
  "id": "txn_1a2b3c4d",
  "type": "debit",
  "amount": 500.00,
  "fee": 2.50,
  "currency": "ZAR",
  "balance_after": 12000.50,
  "description": "Payment to Acme Corp",
  "reference": "INV-2025-001",
  "counterparty": {
    "name": "Acme Corp",
    "account_number": "****7890"
  },
  "payment_id": "pay_9x8y7z6w",
  "status": "completed",
  "created_at": "2025-04-05T10:30:00Z",
  "completed_at": "2025-04-07T09:15:00Z"
}
```
