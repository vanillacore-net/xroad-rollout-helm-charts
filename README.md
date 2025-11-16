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

**ONLY main branch pushes sync to the original server:**
```bash
git checkout main
git merge dev --no-ff  # Merge changes from dev
git push-main          # Pushes main to BOTH GitHub AND original server
```

**Remotes:**
- **origin** (GitHub): `https://github.com/vanillacore-net/xroad-rollout-helm-charts.git`
- **server** (Original): `ssh://karsten@167.71.79.238/home/karsten/im-karsten`
- **Alias**: `git push-main` = pushes main to both remotes

### Dev Branch (Infrastructure Adaptations - GitHub Only)
The **dev** branch contains our infrastructure-specific changes:
- Namespace templating (im-ns → {{ .Release.Namespace }})
- MetalLB LoadBalancer configurations
- Infrastructure-specific adaptations

**Dev branch is NOT synced to server** - it only exists on GitHub for our infrastructure work.

**Development workflow:**
1. Make changes in dev branch
2. Test thoroughly
3. Push to GitHub: `git push origin dev` (GitHub only, NOT to server)
4. Create PR to merge dev → main
5. Merge and push: `git push-main` (syncs to BOTH GitHub and server)

## License

X-Road is licensed under the MIT License. See original X-Road repository for details.
