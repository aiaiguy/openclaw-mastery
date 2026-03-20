# OpenClaw Installation and Setup

> Last updated: 2026-03-20 | Source: docs.openclaw.ai/install

## Requirements

- Node 24 (recommended) or Node 22.16+
- macOS, Linux, or Windows (WSL2 preferred)
- 2GB+ RAM for Docker deployments

## Installation Methods

### Method 1: Installer Script (Recommended)

```bash
# macOS / Linux / WSL2
curl -fsSL https://openclaw.ai/install.sh | bash

# Windows PowerShell
iwr -useb https://openclaw.ai/install.ps1 | iex

# Skip onboarding
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
```

### Method 2: npm / pnpm

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon

# pnpm
pnpm add -g openclaw@latest
pnpm approve-builds -g    # Required for pnpm
openclaw onboard --install-daemon
```

### Method 3: Docker

```bash
./scripts/docker/setup.sh
# Or manual:
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

Pre-built images: `ghcr.io/openclaw/openclaw:latest` (also `main`, `<version>` tags)

### Method 4: From Source

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw && pnpm install && pnpm ui:build && pnpm build && pnpm link --global
pnpm openclaw onboard --install-daemon
```

### Method 5: GitHub Main Branch

```bash
npm install -g github:openclaw/openclaw#main
```

### Method 6: Nix

Nix flake support available for declarative configuration.

## Version Management

```bash
openclaw --version                    # Check version
openclaw update                       # Update to latest
openclaw update --channel stable      # Tagged releases (vYYYY.M.D)
openclaw update --channel beta        # Prerelease
openclaw update --channel dev         # Main branch head
openclaw doctor                       # Verify after update
```

npm dist-tags: `latest` (stable), `beta`, `dev`

## Rollback

[Unverified] No dedicated rollback command. Pin to specific version: `npm install -g openclaw@<version>`. Docker users can pull specific image tags.

## Reset and Uninstall

```bash
openclaw reset --scope config                    # Config only
openclaw reset --scope config+creds+sessions     # Config + credentials + sessions
openclaw reset --scope full                      # Everything

openclaw uninstall --service     # Service only
openclaw uninstall --state       # State directory
openclaw uninstall --workspace   # Workspace
openclaw uninstall --all         # Everything
```
