# X-Road Security Server Deployment Guide

## Overview

The Security Server mediates messages between service providers and consumers, providing secure communication channels.

## Prerequisites

1. Kubernetes cluster with kubectl access
2. Namespace `im-ns` created (automatically created by install script if missing)
3. MetalLB installed and configured
4. Test CA deployed and running
5. **Central Server deployed and configured**
6. Security Server registration code from Central Server
7. Certificate files prepared in `certs/` directory

## Component Architecture

The Security Server consists of:
- **Proxy Container**: Message routing and mediation
- **Signer Container**: Key and certificate management
- **Admin UI**: Web-based administration interface
- **Database**: PostgreSQL for local configuration

## Deployment

### Step 1: Prepare Certificate Files

Ensure certificate files are in the `certs/` directory:

```bash
cd bb-im/x-road-ssx/
ls -l certs/
# Required files:
#   certificate-ss.crt
#   certificate-ss.key
#   certificate-ss-ui.p12
#   certificate-ss-internal.p12
```

### Step 2: Deploy Security Server

Deploy using the installation script:

```bash
./install_ss_1.sh
```

This script will:
- Verify certificate files exist
- Deploy the Helm chart to namespace `im-ns`
- Create LoadBalancer services automatically via Helm templates
- Wait for successful deployment (15 minute timeout)

### Step 3: Verify Deployment

Check pod status:
```bash
kubectl get pods -n im-ns -l app=mss-1
```

Expected output:
- Pod: `mss-1-*` in Running state

Check services:
```bash
kubectl get svc -n im-ns | grep mss-1
```

### Step 4: Verify LoadBalancer Services

The Helm chart automatically creates LoadBalancer services when deployed. Verify they have been assigned external IPs:

```bash
kubectl get svc -n im-ns | grep -E 'xroad-ss-(messaging|ocsp)-lb'
```

Expected LoadBalancer services:
- `xroad-ss-messaging-lb`: Port 5500 (Message exchange)
- `xroad-ss-ocsp-lb`: Port 5577 (OCSP responses)

All services should show `EXTERNAL-IP: 10.0.0.100` (configured via MetalLB annotations in the Helm chart).

Note: The main service `mss-1` uses ClusterIP with Ingress for the Admin UI (port 4000).

## Configuration

### Step 1: Access Security Server Admin UI

Access via the Ingress hostname (if Ingress is enabled):

```
https://mss-1.im.assembly.govstack.global
```

Or port-forward for direct access:

```bash
kubectl port-forward -n im-ns svc/mss-1 4001:4000
# Then access: https://localhost:4001
```

Note: The service uses port 4000 internally. Port-forwarding to 4001 avoids conflict with Central Server Admin UI.

### Step 2: Initialize Security Server

1. Configure server owner information
2. Generate signing key
3. Generate authentication certificate request
4. Note the registration code displayed

### Step 3: Register with Central Server

1. Access Central Server Admin UI:
   - Via Ingress: `https://xroad-cs1.im.assembly.govstack.global`
   - Or via LoadBalancer: `https://10.0.0.100:4000`
2. Navigate to Security Servers section
3. Add new Security Server using the registration code from Step 2
4. Approve the registration request

### Step 4: Complete Security Server Registration

1. Return to Security Server Admin UI
2. Verify Central Server approved registration
3. Configure member subsystems
4. Add service descriptions

## Port Reference

| Port | Purpose | Access | Service Type |
|------|---------|--------|--------------|
| 4000 | Admin UI | Ingress/ClusterIP | ClusterIP with Ingress |
| 5500 | Message protocol | LoadBalancer | LoadBalancer (10.0.0.100:5500) |
| 5577 | OCSP responses | LoadBalancer | LoadBalancer (10.0.0.100:5577) |
| 5588 | Proxy UI | ClusterIP | ClusterIP |
| 8080 | Proxy HTTP | ClusterIP | ClusterIP |
| 8443 | Service mediation (HTTPS) | ClusterIP | ClusterIP |

Note: LoadBalancer services are automatically created by the Helm chart for ports 5500 and 5577.

## Troubleshooting

### Registration Fails

Verify Central Server connectivity:
```bash
# From Security Server pod
kubectl exec -it -n im-ns deployment/mss-1 -- curl -k https://xroad-cs1:8443
```

### Service Has No Endpoints

Check service selector matches pod labels:
```bash
kubectl get svc <service-name> -n im-ns -o yaml | grep -A 3 "selector:"
kubectl get pods -n im-ns --show-labels
```

### Cannot Access Admin UI

Verify service and Ingress:
```bash
# Check endpoints
kubectl get endpoints -n im-ns | grep mss-1

# Verify pod is running
kubectl get pods -n im-ns -l app=mss-1

# Check Ingress
kubectl get ingress -n im-ns
```

### LoadBalancer Services Not Getting External IP

Verify MetalLB is running and check service annotations:
```bash
# Check MetalLB
kubectl get pods -n metallb-system

# Verify LoadBalancer service annotations
kubectl get svc xroad-ss-messaging-lb -n im-ns -o yaml | grep -A 5 "annotations:"
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
helm install example-service . -n im-ns
```

## Next Steps

- Configure additional Security Servers as needed
- Set up service monitoring and logging
- Review operational procedures
