# Troubleshooting Guide

Common issues and their resolutions. If your issue isn't listed here, check the [Incident Response](incident-response.md) runbook for escalation procedures.

## Pod Issues

### Pod stuck in CrashLoopBackOff

**Symptoms:** Pod repeatedly crashes and restarts.

**Diagnosis:**

```bash
oc describe pod <pod-name> -n <namespace>
oc logs <pod-name> -n <namespace> --previous
```

**Common causes:**

1. **Missing environment variable** — Check that all required secrets and config maps are mounted
2. **Database connection failure** — Verify the database is reachable from the pod's namespace
3. **OOM Killed** — Check `Last State: Terminated - Reason: OOMKilled` and increase memory limits
4. **Liveness probe failing** — Verify the health endpoint is responding

### Pod stuck in Pending

**Symptoms:** Pod never gets scheduled to a node.

**Diagnosis:**

```bash
oc describe pod <pod-name> -n <namespace>
# Look for events like "Insufficient cpu" or "Insufficient memory"
```

**Resolution:** Either reduce resource requests or request a node capacity increase.

---

## Networking Issues

### Service returning 503

**Symptoms:** Intermittent or consistent 503 errors from the service.

**Check these in order:**

1. Are pods running? `oc get pods -n <namespace>`
2. Are pods passing readiness checks? Look for `Ready: False` in pod describe
3. Is the Service selector matching pod labels?
4. Is the Route/Ingress configured correctly?

### DNS Resolution Failures

**Symptoms:** Pods cannot resolve internal service names.

```bash
oc exec <pod-name> -n <namespace> -- nslookup kubernetes.default
oc exec <pod-name> -n <namespace> -- cat /etc/resolv.conf
```

**Resolution:** Check CoreDNS pods in `openshift-dns` namespace.

---

## Database Issues

### Connection Pool Exhausted

**Symptoms:** Application logs show "too many connections" errors.

**Quick fix:**

```bash
# Check current connections
oc exec <db-pod> -- psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"
# Kill idle connections
oc exec <db-pod> -- psql -U postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle' AND query_start < now() - interval '10 minutes';"
```

**Long-term fix:** Tune `max_connections` in PostgreSQL config and connection pool size in the application.

### Slow Queries

**Symptoms:** Elevated response times in the API.

```bash
# Find long-running queries
oc exec <db-pod> -- psql -U postgres -c "SELECT pid, now() - query_start AS duration, query FROM pg_stat_activity WHERE state = 'active' ORDER BY duration DESC LIMIT 10;"
```

---

## TechDocs-Specific Issues

### Docs not building in RHDH

**Symptoms:** TechDocs panel shows "Documentation not found" or build errors.

**Checklist:**

1. Verify the `backstage.io/techdocs-ref: dir:.` annotation exists in `catalog-info.yaml`
2. Confirm `mkdocs.yml` is in the repo root
3. Check that `docs/index.md` exists
4. Ensure `techdocs-core` is listed in plugins
5. Verify the entity is properly registered in the catalog

### Broken links in TechDocs

Use relative paths for cross-page links:

```markdown
<!-- From docs/runbooks/troubleshooting.md linking to docs/architecture/overview.md -->
See the [Architecture Overview](../architecture/overview.md)
```

## Related Pages

- [Incident Response](incident-response.md) — full incident procedure
- [Architecture Overview](../architecture/overview.md) — system component details
