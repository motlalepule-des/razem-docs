---
layout: default
title: API Reference
parent: Developer Documentation
nav_order: 2
has_children: true
permalink: /docs/api
---

# Razem API Reference

The Razem REST API enables developers to integrate financial services into their applications. All API access is over HTTPS and data is sent and received as JSON.

## Base URL

```
https://api.razem.co.za/v1
```

## API Versioning

The current API version is `v1`. The version is included in the URL path. When breaking changes are introduced, a new version will be released while the previous version continues to be supported.

## Request Format

All requests must include:

| Header | Value | Required |
|:---|:---|:---|
| `Authorization` | `Bearer YOUR_ACCESS_TOKEN` | Yes |
| `Content-Type` | `application/json` | Yes (for POST/PUT/PATCH) |
| `X-Request-ID` | UUID for idempotency | Recommended |

## Response Format

All successful responses return JSON with HTTP `2xx` status codes. Error responses return JSON with the appropriate `4xx` or `5xx` status codes.

```json
{
  "data": { ... },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 150
  }
}
```

## Rate Limiting

| Plan | Requests per minute | Requests per day |
|:---|:---|:---|
| Basic | 60 | 10,000 |
| Standard | 300 | 100,000 |
| Enterprise | Custom | Custom |

Rate limit headers are included in every response:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1712312345
```
