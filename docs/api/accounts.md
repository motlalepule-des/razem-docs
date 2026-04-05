---
layout: default
title: Accounts
parent: API Reference
nav_order: 2
---

# Accounts API
{: .no_toc }

Manage account information, balances, and settings.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Get Account

Retrieve details for the authenticated account.

```
GET /v1/account
```

### Response

```json
{
  "id": "acc_1234567890",
  "type": "personal",
  "status": "active",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+27821234567",
  "kyc_status": "verified",
  "balance": {
    "available": 12500.50,
    "pending": 300.00,
    "currency": "ZAR"
  },
  "limits": {
    "daily_payment": 50000.00,
    "monthly_payment": 500000.00
  },
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-04-05T08:00:00Z"
}
```

---

## Get Balance

Retrieve current account balance.

```
GET /v1/account/balance
```

### Response

```json
{
  "available": 12500.50,
  "pending": 300.00,
  "reserved": 0.00,
  "total": 12800.50,
  "currency": "ZAR",
  "last_updated": "2025-04-05T08:00:00Z"
}
```

---

## Update Account

Update account profile information.

```
PATCH /v1/account
```

### Request Body

```json
{
  "name": "John A. Doe",
  "phone": "+27821234568",
  "address": {
    "street": "123 Main Street",
    "city": "Cape Town",
    "province": "Western Cape",
    "postal_code": "8001",
    "country": "ZA"
  }
}
```

---

## Account Statements

### List Statements

```
GET /v1/account/statements
```

**Query Parameters:**

| Parameter | Type | Description |
|:---|:---|:---|
| `year` | integer | Filter by year (e.g., `2025`) |
| `month` | integer | Filter by month (1-12) |

### Download Statement

```
GET /v1/account/statements/{statement_id}/download
```

Returns a PDF file of the statement.

---

## Account Types

| Type | Description |
|:---|:---|
| `personal` | Individual account for personal use |
| `business` | Business account with additional features |
| `joint` | Shared account with multiple owners |

---

## Account Status Values

| Status | Description |
|:---|:---|
| `pending` | Account created, awaiting KYC verification |
| `active` | Fully verified and operational |
| `restricted` | Limited functionality (see restrictions field) |
| `suspended` | Account suspended by Razem compliance |
| `closed` | Account has been permanently closed |
