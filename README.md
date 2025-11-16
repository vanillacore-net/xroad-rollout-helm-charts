# X-Road Helm Charts - VanillaCore Mirror

This repository contains Helm charts and configurations for deploying X-Road Information Mediator components.

## Repository Structure

- `bb-im/` - Main X-Road components for BB-IM cluster
  - `test-ca/` - Test Certificate Authority (trust anchor)
  - `x-road-csx/` - Central Server Helm chart
  - `x-road-ssx/` - Security Server Helm chart
  - `hurl-auto-config/` - Automated configuration scripts
  - `example-service/` - Example API services for testing
- `add-im/` - Additional IM cluster configurations
- `bastion/` - Bastion host Helm chart

## Branching Strategy

- **main** - Mirrors the original upstream state from source repository
- **dev** - Active development with infrastructure adaptations for our deployment

## Usage

See `DEPLOYMENT_GUIDE.md` for deployment instructions.

## Repository Sync Workflow

### Main Branch (Upstream Mirror)
The **main** branch mirrors the original X-Road deployment from the source server.

**When you push to main on GitHub, it automatically syncs to the original server:**
```bash
git checkout main
git merge dev --no-ff  # Merge changes from dev
git push origin main   # Pushes to BOTH GitHub AND original server
```

The remote is configured to push to both locations:
- **GitHub**: `https://github.com/vanillacore-net/xroad-rollout-helm-charts.git`
- **Server**: `ssh://karsten@167.71.79.238/home/karsten/im-karsten`

### Dev Branch (Infrastructure Adaptations)
The **dev** branch contains our infrastructure-specific changes:
- Namespace templating (im-ns → {{ .Release.Namespace }})
- MetalLB LoadBalancer configurations
- Infrastructure-specific adaptations

**Development workflow:**
1. Make changes in dev branch
2. Test thoroughly
3. Create PR to merge dev → main
4. Merge and push main (auto-syncs to server)

## License

X-Road is licensed under the MIT License. See original X-Road repository for details.
