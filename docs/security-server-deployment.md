# X-Road Security Server Deployment Guide

## Overview

The Security Server mediates messages between service providers and consumers, providing secure communication channels.

## Prerequisites

1. Kubernetes cluster with kubectl access
2. Namespace `xroad-im` created
3. MetalLB installed and configured
4. Test CA deployed and running
5. **Central Server deployed and configured**
6. Security Server registration code from Central Server

## Component Architecture

The Security Server consists of:
- **Proxy Container**: Message routing and mediation
- **Signer Container**: Key and certificate management
- **Admin UI**: Web-based administration interface
- **Database**: PostgreSQL for local configuration

## Deployment

### Step 1: Deploy Security Server

Navigate to the chart directory and deploy:

```bash
cd bb-im/x-road-ssx/
./install-xroad-im.sh
```

### Step 2: Verify Deployment

Check pod status:
```bash
kubectl get pods -n xroad-im -l app.kubernetes.io/component=security-server
```

Expected output:
- Pod: `mss-0-*` in Running state

Check services:
```bash
kubectl get svc -n xroad-im | grep mss-0
```

### Step 3: Deploy LoadBalancer Services

Apply LoadBalancer services for external access:

```bash
kubectl apply -f xroad-ss-admin-ui-40001-lb.yaml
kubectl apply -f xroad-ss-messaging-5500-lb.yaml
kubectl apply -f xroad-ss-ocsp-5577-lb.yaml
kubectl apply -f xroad-ss-client-8443-lb.yaml
```

Verify LoadBalancer services have external IPs:
```bash
kubectl get svc -n xroad-im | grep xroad-ss
```

All services should show `EXTERNAL-IP: 10.0.0.100` with different ports.

## Configuration

### Step 1: Access Security Server Admin UI

```
https://10.0.0.100:40001
```

Note: External port 40001 maps to container port 4000 to avoid conflict with Central Server Admin UI.

### Step 2: Initialize Security Server

1. Configure server owner information
2. Generate signing key
3. Generate authentication certificate request
4. Note the registration code displayed

### Step 3: Register with Central Server

1. Access Central Server Admin UI (https://10.0.0.100:4000)
2. Navigate to Security Servers section
3. Add new Security Server using the registration code from Step 2
4. Approve the registration request

### Step 4: Complete Security Server Registration

1. Return to Security Server Admin UI
2. Verify Central Server approved registration
3. Configure member subsystems
4. Add service descriptions

## Port Reference

| Port | Purpose | Access |
|------|---------|--------|
| 40001 | Admin UI | VPN only |
| 5500 | Message protocol | VPN + Gateway |
| 5577 | OCSP responses | VPN + Gateway |
| 8443 | Service mediation (HTTPS) | VPN + Gateway |

## Troubleshooting

### Registration Fails

Verify Central Server connectivity:
```bash
# From Security Server pod
kubectl exec -it <ss-pod-name> -n xroad-im -- curl -k https://cs-1:8443
```

### Service Has No Endpoints

Check service selector matches pod labels:
```bash
kubectl get svc <service-name> -n xroad-im -o yaml | grep -A 3 "selector:"
kubectl get pods -n xroad-im --show-labels
```

### Cannot Access Admin UI

Verify LoadBalancer service:
```bash
# Check endpoints
kubectl get endpoints -n xroad-im | grep ss-admin

# Verify pod is running
kubectl get pods -n xroad-im -l app.kubernetes.io/component=security-server
```

## Post-Deployment

After successful deployment and registration:

1. Configure service descriptions
2. Add access rights for consumers
3. Test service mediation
4. Set up monitoring

## Testing

Deploy example services for testing:
```bash
cd bb-im/example-service/
helm install example-service . -n xroad-im
```

## Next Steps

- Configure additional Security Servers as needed
- Set up service monitoring and logging
- Review operational procedures
