# Network Topology — GovCloud Environment

This page documents the fictional network architecture for our OpenShift clusters running in AWS GovCloud.

## High-Level Network Overview

![Network Overview](images/network-overview.svg)

## Service Mesh Traffic Flow

This diagram shows how requests flow through the service mesh within the OpenShift cluster.

![Service Mesh Traffic Flow](images/service-mesh.svg)

## Deployment Pipeline

The CI/CD pipeline from code commit to production deployment.

![Deployment Pipeline](images/deployment-pipeline.svg)

## Cluster Node Architecture

![Cluster Node Architecture](images/cluster-nodes.svg)

## Disaster Recovery Architecture

This shows the failover setup between primary and DR regions.

![Disaster Recovery Architecture](images/disaster-recovery.svg)

## Monitoring & Alerting Flow

![Monitoring and Alerting Flow](images/monitoring.svg)

## Network Security Zones

![Network Security Zones](images/security-zones.svg)

## Related Pages

- [Architecture Overview](overview.md) — system components and data flow
- [Deployment Architecture](deployment.md) — CI/CD and scaling
- [Incident Response](../runbooks/incident-response.md) — what to do when things break
