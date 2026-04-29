# API Reference

This page documents the REST API for the example service, demonstrating how to use TechDocs for API documentation.

## Base URL

| Environment | Base URL |
|-------------|----------|
| Development | `https://api.dev.example.com/v1` |
| Staging | `https://api.stage.example.com/v1` |
| Production | `https://api.example.com/v1` |

## Authentication

All API requests require a Bearer token in the `Authorization` header:

```bash
curl -H "Authorization: Bearer <token>" https://api.example.com/v1/orders
```

Tokens are issued by the [Auth Service](../architecture/overview.md#api-gateway) via OAuth2.

---

## Endpoints

### Orders

#### List Orders

```
GET /v1/orders
```

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | integer | No | 1 | Page number |
| `per_page` | integer | No | 20 | Items per page (max 100) |
| `status` | string | No | — | Filter by status: `pending`, `active`, `completed`, `cancelled` |
| `created_after` | string | No | — | ISO 8601 datetime filter |

**Example request:**

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api.example.com/v1/orders?status=active&per_page=5"
```

**Example response:**

```json
{
  "data": [
    {
      "id": "ord-12345",
      "status": "active",
      "created_at": "2026-04-15T10:30:00Z",
      "total": 149.99
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 5,
    "total": 42
  }
}
```

#### Create Order

```
POST /v1/orders
```

**Request body:**

```json
{
  "items": [
    {
      "sku": "WIDGET-001",
      "quantity": 2
    }
  ],
  "shipping_address": {
    "street": "123 Main St",
    "city": "Springfield",
    "state": "VA",
    "zip": "22150"
  }
}
```

**Response codes:**

| Code | Meaning |
|------|---------|
| `201` | Order created successfully |
| `400` | Invalid request body |
| `401` | Missing or invalid auth token |
| `422` | Validation error (e.g., invalid SKU) |
| `500` | Internal server error |

#### Get Order by ID

```
GET /v1/orders/{id}
```

Returns the full order object including line items, shipping status, and audit trail.

#### Cancel Order

```
DELETE /v1/orders/{id}
```

Cancels an order. Only orders in `pending` or `active` status can be cancelled.

---

### Health

#### Health Check

```
GET /healthz
```

Returns `200 OK` if the service is healthy. Used by Kubernetes liveness probes.

#### Readiness Check

```
GET /readyz
```

Returns `200 OK` when the service is ready to accept traffic. Checks database connectivity and downstream dependencies.

---

## Error Format

All errors follow a consistent format:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid SKU: WIDGET-999 not found",
    "request_id": "req-abc123"
  }
}
```

Include the `request_id` when reporting issues to the SRE team.

## Rate Limits

| Tier | Requests/min | Burst |
|------|-------------|-------|
| Default | 100 | 150 |
| Elevated | 1000 | 1500 |
| Internal | Unlimited | — |

Rate limit headers are included in every response:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1714400000
```

## Related Pages

- [Architecture Overview](../architecture/overview.md) — system design and component details
- [Configuration Reference](configuration.md) — mkdocs.yml options
- [Formatting Guide](formatting.md) — Markdown features supported in TechDocs
