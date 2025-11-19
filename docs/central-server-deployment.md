# X-Road Central Server Deployment Guide

## Overview

The Central Server is the governance component of X-Road, managing member registration, configuration distribution, and trust anchors.

## Prerequisites

1. Kubernetes cluster with kubectl access
2. Namespace `im-ns` created (automatically created by install script if missing)
3. MetalLB installed and configured
4. Test CA deployed and running
5. PersistentVolume support available

## Component Architecture

The Central Server consists of:
- **Database Container**: PostgreSQL for member data
- **Signer Container**: Key and certificate management
- **Central Server Container**: Application logic and REST API
- **Admin UI**: Web-based administration interface
- **Configuration Proxy**: Distributes configuration to Security Servers

## Deployment

### Step 1: Deploy Central Server

Navigate to the chart directory and deploy:

```bash
cd bb-im/x-road-csx/
./install_cs_1.sh
```

### Step 2: Verify Deployment

Check pod status:
```bash
kubectl get pods -n im-ns -l app.kubernetes.io/component=central-server
```

Expected output:
- Pod: `xroad-cs1-*` in Running state
- StatefulSet/Deployment: `cs1-db` (PostgreSQL database)

Check services:
```bash
kubectl get svc -n im-ns | grep cs
```

### Step 3: Verify LoadBalancer Services

The Helm chart automatically creates LoadBalancer services when deployed. Verify they have been assigned external IPs:

```bash
kubectl get svc -n im-ns | grep -E 'xroad-cs-(admin|reg)-lb'
```

Expected LoadBalancer services:
- `xroad-cs-admin-lb`: Port 4000 (Admin UI)
- `xroad-cs-reg-lb`: Port 4001 (Member registration)

All services should show `EXTERNAL-IP: 10.0.0.100` (configured via MetalLB annotations in the Helm chart)

## Access

### Admin UI

Access the Central Server Admin UI:
```
https://10.0.0.100:4000
```

Login with default credentials (check deployed secrets).

### Initial Configuration

1. Complete Central Server initialization wizard
2. Configure member classes
3. Configure trust services (CA, TSA, OCSP)
4. Generate Security Server registration codes

## Port Reference

| Port | Purpose | Access |
|------|---------|--------|
| 4000 | Admin UI | VPN only |
| 4001 | Member registration | VPN + Gateway |
| 8443 | Configuration distribution (HTTPS) | VPN + Gateway |
| 9998 | Test CA | VPN + Gateway |

## Troubleshooting

### Pod Not Starting

Check pod status and logs:
```bash
kubectl describe pod <pod-name> -n im-ns
kubectl logs <pod-name> -n im-ns
```

### LoadBalancer Has No Endpoints

Verify service selector matches pod labels:
```bash
kubectl get svc <service-name> -n im-ns -o yaml | grep -A 3 "selector:"
kubectl get pods -n im-ns --show-labels
```

If selectors don't match, the Helm chart configuration may need adjustment.

### Cannot Access Admin UI

Verify connectivity:
```bash
# Check endpoints
kubectl get endpoints -n im-ns | grep admin

# Test from within cluster
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -n im-ns -- curl -k https://xroad-cs-admin-lb:4000
```

## Post-Deployment

After successful deployment:

1. Configure trust services
2. Add member organizations
3. Generate Security Server registration codes
4. Proceed to Security Server deployment

## Next Steps

Continue with [Security Server Deployment](security-server-deployment.md).
