# X-Road Central Server Deployment Guide

## Overview

The Central Server is the governance component of X-Road, managing member registration, configuration distribution, and trust anchors.

## Prerequisites

1. Kubernetes cluster with kubectl access
2. Namespace `xroad-im` created
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
./install-xroad-im.sh
```

### Step 2: Verify Deployment

Check pod status:
```bash
kubectl get pods -n xroad-im -l app.kubernetes.io/component=central-server
```

Expected output:
- Pod: `cs-1-*` in Running state
- StatefulSet: `cs-1-db` (PostgreSQL database)

Check services:
```bash
kubectl get svc -n xroad-im | grep cs-1
```

### Step 3: Deploy LoadBalancer Services

Apply LoadBalancer services for external access:

```bash
kubectl apply -f xroad-cs-admin-ui-4000-lb.yaml
kubectl apply -f xroad-cs-admin-4001-lb.yaml
kubectl apply -f xroad-cs-client-8443-lb.yaml
kubectl apply -f xroad-cs-testca-9998-lb.yaml
```

Verify LoadBalancer services have external IPs:
```bash
kubectl get svc -n xroad-im | grep xroad-cs
```

All services should show `EXTERNAL-IP: 10.0.0.100`

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
kubectl describe pod <pod-name> -n xroad-im
kubectl logs <pod-name> -n xroad-im
```

### LoadBalancer Has No Endpoints

Verify service selector matches pod labels:
```bash
kubectl get svc <service-name> -n xroad-im -o yaml | grep -A 3 "selector:"
kubectl get pods -n xroad-im --show-labels
```

If selectors don't match, update LoadBalancer service selectors.

### Cannot Access Admin UI

Verify connectivity:
```bash
# Check endpoints
kubectl get endpoints -n xroad-im | grep admin-ui

# Test from within cluster
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- curl -k https://cs-1:4000
```

## Post-Deployment

After successful deployment:

1. Configure trust services
2. Add member organizations
3. Generate Security Server registration codes
4. Proceed to Security Server deployment

## Next Steps

Continue with [Security Server Deployment](security-server-deployment.md).
