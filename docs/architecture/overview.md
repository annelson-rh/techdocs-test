# Architecture Overview

This page demonstrates how to document system architecture using TechDocs. It also shows off several Markdown formatting features.

## System Components

The platform consists of several interconnected services:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│  API Gateway │────▶│  Auth Service│
│   (React)    │     │   (Envoy)    │     │   (Keycloak) │
└──────────────┘     └──────┬───────┘     └──────────────┘
                            │
                  ┌─────────┼─────────┐
                  ▼         ▼         ▼
           ┌──────────┐ ┌────────┐ ┌──────────┐
           │ Service A│ │Svc B   │ │ Service C│
           │ (Go)     │ │(Python)│ │ (Java)   │
           └────┬─────┘ └───┬────┘ └────┬─────┘
                │           │           │
                ▼           ▼           ▼
           ┌──────────────────────────────────┐
           │         PostgreSQL / Redis       │
           └──────────────────────────────────┘
```

## Component Details

### API Gateway

The API Gateway handles routing, rate limiting, and TLS termination.

| Property | Value |
|----------|-------|
| Technology | Envoy Proxy |
| Port | 8443 (HTTPS) |
| Rate Limit | 1000 req/min per client |
| Health Check | `/healthz` |

### Service A — Order Processing

Handles order creation, validation, and lifecycle management.

**Responsibilities:**

- Validate incoming orders against business rules
- Persist orders to the database
- Emit events to the message bus for downstream consumers

**Key endpoints:**

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/orders` | List orders with pagination |
| `POST` | `/api/v1/orders` | Create a new order |
| `GET` | `/api/v1/orders/{id}` | Get order by ID |
| `PUT` | `/api/v1/orders/{id}` | Update an existing order |
| `DELETE` | `/api/v1/orders/{id}` | Cancel an order |

### Service B — Notifications

Sends email, SMS, and push notifications based on events.

### Service C — Analytics

Aggregates metrics and generates reports. See the [API Reference](../reference/api.md) for query parameters.

## Data Flow

1. User submits a request through the **Frontend**
2. Request passes through the **API Gateway** for auth and rate limiting
3. Gateway routes to the appropriate backend **Service**
4. Service processes the request and writes to the **database**
5. Service emits an event to the **message bus**
6. Downstream services (Notifications, Analytics) consume the event

## Infrastructure

All services run on OpenShift in AWS GovCloud. For cluster management details, see the [Runbooks](../runbooks/incident-response.md).

### Environment Matrix

| Environment | Cluster | Region | Purpose |
|-------------|---------|--------|---------|
| Development | dev-cluster-01 | us-gov-west-1 | Feature development |
| Staging | stg-cluster-01 | us-gov-west-1 | Pre-production testing |
| Production | prd-cluster-01 | us-gov-west-1 | Live traffic |
| Production | prd-cluster-02 | us-gov-east-1 | DR / failover |

## Related Pages

- [Deployment Architecture](deployment.md) — how services are deployed and scaled
- [API Reference](../reference/api.md) — detailed endpoint documentation
- [Incident Response](../runbooks/incident-response.md) — what to do when things break
