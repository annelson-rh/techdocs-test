# Deployment Architecture

This page documents the deployment model, scaling policies, and CI/CD pipeline.

## Deployment Model

All services are deployed as containers on OpenShift using a GitOps workflow managed through [app-interface](https://gitlab.cee.redhat.com).

### Deployment Pipeline

```
Developer pushes code
        │
        ▼
┌───────────────┐
│   CI Build    │  ← Unit tests, linting, container build
└───────┬───────┘
        │
        ▼
┌───────────────┐
│  Image Push   │  ← Push to Quay.io registry
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ SaaS File PR  │  ← Update image ref in app-interface
└───────┬───────┘
        │
        ▼
┌───────────────┐
│   Merge &     │  ← Reconciliation deploys to cluster
│   Reconcile   │
└───────────────┘
```

## Namespace Layout

Each service gets its own namespace per environment:

```
my-service--dev       # Development
my-service--stage     # Staging
my-service--prod      # Production
```

## Resource Quotas

Default resource allocations per service instance:

| Resource | Request | Limit |
|----------|---------|-------|
| CPU | 100m | 500m |
| Memory | 128Mi | 512Mi |
| Replicas (dev) | 1 | 1 |
| Replicas (stage) | 2 | 3 |
| Replicas (prod) | 3 | 10 |

## Scaling

Horizontal Pod Autoscaler (HPA) is configured for production workloads:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-service
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## Rollback Procedure

If a deployment causes issues:

1. Identify the bad commit or image tag
2. Revert the SaaS file change in app-interface
3. Merge the revert PR — reconciliation will roll back the deployment
4. Investigate the root cause before re-deploying

For production incidents, follow the [Incident Response](../runbooks/incident-response.md) runbook.

## Related Pages

- [Architecture Overview](overview.md) — system components and data flow
- [Configuration Reference](../reference/configuration.md) — how to configure mkdocs.yml
