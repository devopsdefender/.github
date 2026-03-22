# DevOps Defender

**Secure your CI/CD pipeline with confidential computing.**

Deploy unmodified Docker applications to hardware-verified Intel TDX enclaves — no code changes, no long-lived secrets, no exposed ports.

## How It Works

DevOps Defender uses a zero-secret trust model built on three layers:

| Layer | Technology | What It Does |
|-------|-----------|--------------|
| **Trust Root** | Intel TDX | Hardware attestation proves each VM's identity via cryptographic quotes from the CPU |
| **Authentication** | GitHub OIDC | Short-lived JWT tokens from GitHub Actions — no stored API keys or credentials |
| **Networking** | Cloudflare Tunnels | Outbound-only tunnels per agent — no open ports, no TLS certs, no firewall rules |

## Architecture

```
GitHub Actions ──OIDC──▶ Control Plane ◀──Attestation──▶ Intel Trust Authority
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                 Agent 1   Agent 2   Agent N
                 (TDX VM)  (TDX VM)  (TDX VM)
                    │         │         │
                 Cloudflare Tunnels (outbound-only)
```

1. **Register** — Agent boots on a TDX-enabled VM, generates a hardware attestation quote, and registers with the control plane
2. **Verify** — Control plane validates the quote through Intel Trust Authority, checking measurements (MRTD) and runtime state (RTMRs)
3. **Connect** — Agent receives a Cloudflare Tunnel token and establishes an outbound-only secure tunnel
4. **Deploy** — GitHub Actions authenticates via OIDC and submits a Docker Compose workload to the control plane, which assigns it to a verified agent

## Repositories

| Repo | Description |
|------|-------------|
| [agent](https://github.com/devopsdefender/agent) | Rust daemon that runs on TDX VMs — handles attestation, container management, and heartbeats |
| [control-plane](https://github.com/devopsdefender/control-plane) | Rust Axum API server — orchestrates agents, deployments, and attestation verification |
| [images](https://github.com/devopsdefender/images) | Packer definitions for building agent VM images (GCP and baremetal) |
| [private-llm](https://github.com/devopsdefender/private-llm) | Example app: self-hosted LLM chat (Ollama + web UI) deployed to the platform |
| [website](https://github.com/devopsdefender/website) | Landing page at [devopsdefender.com](https://devopsdefender.com) |

## Key Concepts

- **Intel TDX** — Trusted Domain Extensions. VMs run in encrypted memory with hardware-generated attestation quotes
- **MRTD** — Measurement of the TDX domain, a hash of the initial VM state. Whitelisted in the control plane before deployment
- **GitHub OIDC** — GitHub Actions issues short-lived JWTs at deploy time, verified by the control plane. No secrets to rotate or leak
- **Cloudflare Tunnels** — Agents establish outbound tunnels, eliminating inbound port exposure entirely

## Quick Start

```bash
# Run the control plane locally
DD_CP_DATABASE_URL=sqlite://dev.db DD_CP_ADMIN_PASSWORD=changeme cargo run --bin dd-cp

# Run an agent (skip attestation for non-TDX environments)
DD_AGENT_SKIP_ATTESTATION=true DD_CP_URL=http://localhost:8080 cargo run --bin dd-agent

# Deploy the example LLM app
cd private-llm && docker compose up
```

## Links

- [Website](https://devopsdefender.com)
- [Documentation](https://github.com/devopsdefender/.github)
