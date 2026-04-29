# Incident Response

This runbook covers the standard operating procedure for responding to production incidents.

## Severity Levels

| Severity | Description | Response Time | Example |
|----------|-------------|---------------|---------|
| **SEV-1** | Complete outage or data loss | 15 minutes | All APIs returning 500s |
| **SEV-2** | Major feature degraded | 30 minutes | Order processing delayed |
| **SEV-3** | Minor feature affected | 4 hours | Dashboard loading slowly |
| **SEV-4** | Cosmetic or low-impact | Next business day | Typo in error message |

## Response Checklist

### Immediate Actions (first 15 minutes)

- [ ] Acknowledge the alert in PagerDuty
- [ ] Join the incident channel
- [ ] Assess severity using the table above
- [ ] Identify the affected service(s)
- [ ] Check recent deployments — was anything rolled out in the last hour?

### Investigation

- [ ] Review dashboards and metrics
- [ ] Check pod logs: `oc logs -f deployment/my-service -n my-service--prod`
- [ ] Look for error spikes in the application logs
- [ ] Check upstream dependencies for issues
- [ ] Review recent configuration changes in app-interface

### Resolution

- [ ] Apply the fix (rollback, config change, or hotfix)
- [ ] Verify the fix resolves the issue
- [ ] Monitor for 30 minutes to confirm stability
- [ ] Update the incident channel with resolution status

## Useful Commands

### Check pod status

```bash
oc get pods -n my-service--prod
oc describe pod <pod-name> -n my-service--prod
```

### View recent logs

```bash
oc logs deployment/my-service -n my-service--prod --tail=100
oc logs deployment/my-service -n my-service--prod --previous
```

### Restart a deployment

```bash
oc rollout restart deployment/my-service -n my-service--prod
oc rollout status deployment/my-service -n my-service--prod
```

### Check resource usage

```bash
oc adm top pods -n my-service--prod
oc adm top nodes
```

## Escalation Path

1. **On-call SRE** — first responder, handles SEV-3 and SEV-4
2. **SRE Team Lead** — escalation for SEV-2, coordinates response
3. **Engineering Manager** — escalation for SEV-1, manages communications
4. **VP Engineering** — executive notification for extended SEV-1 outages

## Post-Incident

After every SEV-1 or SEV-2 incident:

1. Schedule a blameless post-mortem within 48 hours
2. Document the timeline, root cause, and contributing factors
3. Create action items to prevent recurrence
4. Share the post-mortem with the broader team

See the [Troubleshooting Guide](troubleshooting.md) for common issues and fixes.

## Related Pages

- [Troubleshooting Guide](troubleshooting.md) — common problems and solutions
- [Architecture Overview](../architecture/overview.md) — system components
- [Deployment Architecture](../architecture/deployment.md) — rollback procedures
