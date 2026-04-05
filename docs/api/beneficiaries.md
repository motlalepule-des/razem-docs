---
layout: default
title: Beneficiaries
parent: API Reference
nav_order: 4
---

# Beneficiaries API
{: .no_toc }

Manage saved payment recipients for quick and easy payments.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## List Beneficiaries

```
GET /v1/beneficiaries
```

### Response

```json
{
  "data": [
    {
      "id": "ben_abc123",
      "name": "Acme Corp",
      "account_number": "****7890",
      "bank_name": "Standard Bank",
      "bank_code": "051001",
      "account_type": "current",
      "reference": "Rent Payment",
      "created_at": "2025-01-20T09:00:00Z"
    }
  ],
  "meta": {
    "total": 5,
    "page": 1,
    "per_page": 20
  }
}
```

---

## Add Beneficiary

```
POST /v1/beneficiaries
```

### Request Body

```json
{
  "name": "Jane Smith",
  "account_number": "9876543210",
  "bank_code": "250655",
  "account_type": "savings",
  "reference": "Monthly Transfer",
  "notification": {
    "email": "jane@example.com",
    "sms": "+27821234567"
  }
}
```

---

## Get Beneficiary

```
GET /v1/beneficiaries/{beneficiary_id}
```

---

## Update Beneficiary

```
PATCH /v1/beneficiaries/{beneficiary_id}
```

---

## Delete Beneficiary

```
DELETE /v1/beneficiaries/{beneficiary_id}
```

---

## South African Bank Codes

| Bank | Code |
|:---|:---|
| ABSA | 632005 |
| Capitec | 470010 |
| Discovery Bank | 679000 |
| FNB | 250655 |
| Investec | 580105 |
| Nedbank | 198765 |
| Standard Bank | 051001 |
| TymeBank | 678910 |
| Bidvest Bank | 462005 |
| African Bank | 430000 |

---

## Account Types

| Value | Description |
|:---|:---|
| `current` | Current / cheque account |
| `savings` | Savings account |
| `transmission` | Transmission account |
| `credit` | Credit card account |
