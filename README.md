# X-Road Helm Charts for BB-IM Deployment

This repository contains Helm charts and deployment configurations for X-Road Information Mediator components on Kubernetes.

## Components

### Central Server (CS)
Governance component that manages X-Road member registration, configuration distribution, and trust anchors.

**Chart Location**: `bb-im/x-road-csx/`
**Documentation**: [docs/central-server-deployment.md](docs/central-server-deployment.md)

### Security Server (SS)
Message mediation component that handles secure communication between service providers and consumers.

**Chart Location**: `bb-im/x-road-ssx/`
**Documentation**: [docs/security-server-deployment.md](docs/security-server-deployment.md)

### Test CA
Certificate authority for testing and development environments.

**Chart Location**: `bb-im/test-ca/`

## Prerequisites

- Kubernetes cluster with kubectl access
- MetalLB installed for LoadBalancer services
- Namespace: `xroad-im`
- PersistentVolume support

## Deployment Order

X-Road components must be deployed in this specific order:

1. **Test CA** - Provides certificates for CS and SS
2. **Central Server** - Core governance component
3. **Security Server** - Message mediation component

## Quick Start

See component-specific deployment guides in the `docs/` directory.

## Repository Structure

```
xroad-rollout-helm-charts/
├── bb-im/
│   ├── x-road-csx/          # Central Server Helm chart
│   ├── x-road-ssx/          # Security Server Helm chart
│   ├── test-ca/             # Test CA Helm chart
│   ├── hurl-auto-config/    # Configuration automation scripts
│   └── example-service/     # Example service for testing
├── add-im/                  # Additional IM cluster configurations
├── bastion/                 # Bastion host Helm chart
└── docs/                    # Deployment documentation
```

## Branching Strategy

- **main** - Mirrors the original upstream state from source repository
- **dev** - Active development with infrastructure adaptations for deployment

## Branch Protection

Both **main** and **dev** branches are protected:
- Pull requests required (no direct pushes)
- No approvals required (self-merge allowed)
- Force pushes disabled
- Branch deletions disabled
- Enforced for admins

## Development Workflow

1. Create feature branch from dev
2. Make and test changes
3. Create PR to dev
4. Merge PR
5. When ready to sync upstream, create PR: dev → main

## Support

For deployment issues, refer to component-specific troubleshooting sections in the deployment guides.

## License

X-Road is licensed under the MIT License. See original X-Road repository for details.
