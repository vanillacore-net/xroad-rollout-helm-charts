# X-Road Helm Charts - VanillaCore Mirror

This repository contains Helm charts and configurations for deploying X-Road Information Mediator components to Kubernetes clusters.

## Repository Structure

- `bb-im/` - Main X-Road components for BB-IM cluster
  - `test-ca/` - Test Certificate Authority (trust anchor)
  - `x-road-csx/` - Central Server Helm chart
  - `x-road-ssx/` - Security Server Helm chart
  - `loadbalancer-services/` - MetalLB LoadBalancer service definitions
  - `hurl-auto-config/` - Automated configuration scripts
  - `example-service/` - Example API services for testing
- `add-im/` - Additional IM cluster configurations
- `bastion/` - Bastion host Helm chart
- `xroad-centralserver-7-sidecar/` - Sidecar-based Central Server chart
  - `docs/deployment-guide.md` - Central Server deployment instructions
- `docs/` - General documentation

## Documentation

- **Central Server Deployment**: See `xroad-centralserver-7-sidecar/docs/deployment-guide.md`
- **Complete Deployment Guide**: See `DEPLOYMENT_GUIDE.md` for all components (Test CA, CS, SS)
- **Migration Analysis**: See `MIGRATION_ANALYSIS.md`

## Quick Start

1. Ensure prerequisites (VPN access, kubectl configured, MetalLB installed)
2. Deploy Test CA first (provides certificate services)
3. Deploy Central Server (governance component)
4. Deploy Security Servers (message mediation)

See chart-specific documentation for detailed instructions.

## Branching Strategy

- **main** - Mirrors the original upstream state from source repository
- **dev** - Active development with infrastructure adaptations for our deployment

## Upstream Sync

The main branch tracks the original X-Road deployment configurations. Changes should be made in dev branch and merged via pull requests.

To sync from upstream:
```bash
# On source server
git pull origin main

# Push to this mirror
git push vanillacore main
```

## License

X-Road is licensed under the MIT License. See original X-Road repository for details.
