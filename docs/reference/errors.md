---
layout: default
title: Error Reference
parent: Reference
nav_order: 1
---

# Error Reference
{: .no_toc }

Complete reference for all Razem API error codes and how to handle them.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Error Response Format

All error responses follow a consistent structure:

```json
{
  "error": {
    "code": "INSUFFICIENT_FUNDS",
    "message": "Account balance is insufficient to complete this payment.",
    "details": {
      "available_balance": 100.00,
      "required_amount": 500.00,
      "currency": "ZAR"
    },
    "request_id": "req_1a2b3c4d5e"
  }
}
```

---

## HTTP Status Codes

| Status Code | Meaning | Common Causes |
|:---|:---|:---|
| `200 OK` | Success | Request completed successfully |
| `201 Created` | Resource created | New payment, beneficiary, or webhook created |
| `204 No Content` | Success (no body) | Resource deleted |
| `400 Bad Request` | Invalid request | Missing required field, invalid format |
| `401 Unauthorized` | Authentication failed | Invalid or expired token |
| `402 Payment Required` | Payment error | Insufficient funds, limit exceeded |
| `403 Forbidden` | Access denied | Insufficient permissions/scope |
| `404 Not Found` | Resource not found | Invalid ID |
| `409 Conflict` | Conflict | Duplicate request, idempotency conflict |
| `422 Unprocessable Entity` | Validation error | Invalid field values |
| `429 Too Many Requests` | Rate limited | Exceeded rate limit |
| `500 Internal Server Error` | Server error | Razem-side issue |
| `503 Service Unavailable` | Service down | Maintenance or overload |

---

## Error Codes

### Authentication Errors

| Code | HTTP | Description |
|:---|:---|:---|
| `INVALID_TOKEN` | 401 | JWT token is invalid or malformed |
| `EXPIRED_TOKEN` | 401 | JWT token has expired |
| `INVALID_API_KEY` | 401 | API key does not exist |
| `REVOKED_API_KEY` | 401 | API key has been revoked |
| `INSUFFICIENT_SCOPE` | 403 | API key lacks required permission |

### Account Errors

| Code | HTTP | Description |
|:---|:---|:---|
| `ACCOUNT_NOT_FOUND` | 404 | Account with given ID does not exist |
| `ACCOUNT_SUSPENDED` | 403 | Account has been suspended |
| `ACCOUNT_FROZEN` | 403 | Account has been frozen |
| `KYC_REQUIRED` | 403 | KYC verification is required for this operation |
| `KYC_PENDING` | 402 | KYC verification is still in progress |

### Payment Errors

| Code | HTTP | Description |
|:---|:---|:---|
| `INSUFFICIENT_FUNDS` | 402 | Not enough available balance |
| `DAILY_LIMIT_EXCEEDED` | 402 | Daily payment limit exceeded |
| `MONTHLY_LIMIT_EXCEEDED` | 402 | Monthly payment limit exceeded |
| `MINIMUM_AMOUNT` | 400 | Payment below minimum allowed amount |
| `MAXIMUM_AMOUNT` | 400 | Payment above maximum allowed amount |
| `INVALID_BANK_CODE` | 400 | Bank code is not valid |
| `INVALID_ACCOUNT_NUMBER` | 400 | Account number format invalid |
| `PAYMENT_ALREADY_EXISTS` | 409 | Duplicate X-Request-ID detected |
| `PAYMENT_NOT_CANCELLABLE` | 409 | Payment is no longer in cancellable state |
| `INVALID_CURRENCY` | 400 | Currency not supported (only ZAR) |

### Validation Errors

| Code | HTTP | Description |
|:---|:---|:---|
| `REQUIRED_FIELD` | 400 | A required field is missing |
| `INVALID_FORMAT` | 400 | Field value format is invalid |
| `INVALID_DATE` | 400 | Date is in invalid format or range |
| `FIELD_TOO_LONG` | 422 | Field exceeds maximum length |
| `FIELD_TOO_SHORT` | 422 | Field is below minimum length |

### Rate Limiting Errors

| Code | HTTP | Description |
|:---|:---|:---|
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests in time window |
| `CONCURRENT_LIMIT` | 429 | Too many concurrent requests |

---

## Handling Errors

```javascript
async function makePayment(paymentData) {
  try {
    const response = await razem.post('/payments', paymentData);
    return response.data;
  } catch (error) {
    const { status, data } = error.response;
    
    // Handle retryable errors
    if (status === 503 || status === 429) {
      const retryAfter = parseInt(
        error.response.headers['retry-after'] || '60'
      );
      throw new RetryableError(`Service unavailable, retry after ${retryAfter}s`);
    }
    
    // Handle specific business errors
    switch (data.error.code) {
      case 'INSUFFICIENT_FUNDS':
        throw new InsufficientFundsError(data.error.details);
        
      case 'DAILY_LIMIT_EXCEEDED':
        throw new LimitExceededError('Daily payment limit exceeded');
        
      case 'EXPIRED_TOKEN':
        await refreshToken();
        return makePayment(paymentData); // retry once
        
      default:
        throw new RazemAPIError(data.error.message, data.error.code);
    }
  }
}
```

---

## Idempotency

To prevent duplicate payments when retrying failed requests, include a unique `X-Request-ID` header:

```bash
curl -X POST https://api.razem.co.za/v1/payments \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-Request-ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

If the same `X-Request-ID` is received again within 24 hours, Razem returns the original response instead of creating a duplicate payment.
