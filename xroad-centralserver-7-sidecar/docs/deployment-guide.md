# X-Road Central Server Deployment Guide

## Overview

This guide covers deploying the X-Road Central Server to a Kubernetes cluster using the sidecar-based Helm chart. The Central Server is the governance component of X-Road, managing member registration, configuration distribution, and trust anchors.

## Architecture

### Network Access

The X-Road Central Server is accessible via DNS at `cs.im.assembly.govstack.global`:

**External Access** (for Security Servers in other clusters):
- DNS: `cs.im.assembly.govstack.global`
- Ports: 4001 (registration), 8443 (client HTTPS)
- Traffic: External → Gateway NAT → MetalLB LoadBalancer → Central Server pod

**Internal Access** (within cluster):
- Same DNS name resolves via CoreDNS to MetalLB VIP
- Direct pod-to-pod communication

### Components

- **Database Container**: PostgreSQL for member data
- **Signer Container**: Key and certificate management
- **Central Server Container**: Application logic and REST API
- **Admin UI**: Web-based administration
- **Configuration Proxy**: Distributes configuration to Security Servers

## Prerequisites

1. Kubernetes cluster with kubectl access
2. MetalLB installed and configured
3. DNS configured for `*.im.assembly.govstack.global`
4. Gateway NAT rules (for external access)
5. PersistentVolume support
6. Test CA deployed (provides certificates)

## Deployment

### 1. Configure values.yaml

```yaml
service:
  type: LoadBalancer
  annotations:
    metallb.universe.tf/address-pool: default

database:
  host: "xroad-db"
  port: 5432
  name: "centerui_production"

admin:
  username: "xrd"
  password: "secret"  # CHANGE THIS

instance:
  identifier: "GOVSTACK"
```

### 2. Install Chart

```bash
helm install xroad-cs . -f values.yaml -n im-ns --create-namespace
```

### 3. Verify

```bash
# Pods
kubectl get pods -n im-ns

# Services - should show LoadBalancer with MetalLB IP
kubectl get svc -n im-ns
```

### 4. Access UI

```bash
https://cs.im.assembly.govstack.global:4000

# Default: xrd / secret
```

## Verification

```bash
# Test admin UI
curl -k https://cs.im.assembly.govstack.global:4000

# Test registration API
curl -k https://cs.im.assembly.govstack.global:4001

# Test client HTTPS
curl -k https://cs.im.assembly.govstack.global:8443
```

## Troubleshooting

### Cannot Access UI

```bash
kubectl get pods -n im-ns
kubectl logs -n im-ns xroad-cs-xxxxx -c centerserver
```

Solutions:
1. Verify pod Running (3/3)
2. Verify MetalLB assigned IP
3. Check DNS resolution
4. Check gateway NAT rules

### External Access Blocked

Check NAT rules on gateway:
```bash
sudo iptables -t nat -L PREROUTING -n -v | grep 4001
```

Check security groups allow TCP 4001, 8443 ingress.

## Ports

| Port | Purpose | External |
|------|---------|----------|
| 4000 | CS admin UI | Yes |
| 4001 | CS registration API | Yes |
| 8443 | CS client HTTPS | Yes |
| 5500 | Config distribution | Optional |
| 5432 | Database | No |

## References

- X-Road: https://x-road.global/documentation
- NIIS: https://github.com/nordic-institute/X-Road
