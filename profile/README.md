# DevOps Defender

**Confidential compute marketplace — buy enclave capacity with BTC, deploy any Docker app to hardware-verified VMs.**

DevOps Defender combines confidential computing (Intel TDX today, AMD SEV planned) with a decentralized compute marketplace. Node operators supply GPU and CPU capacity; buyers pay with BTC and get workloads running inside hardware-attested enclaves — no trust required.

## How It Works

| Layer | Technology | What It Does |
|-------|-----------|--------------|
| **Trust Root** | Intel TDX (AMD SEV planned) | Hardware attestation proves each VM's identity via cryptographic quotes from the CPU |
| **Authentication** | GitHub OIDC (more providers planned) | Short-lived JWT tokens — no stored API keys or credentials |
| **Networking** | Cloudflare Tunnels (Tailscale planned) | Outbound-only tunnels per agent — no open ports, no TLS certs, no firewall rules |
| **Payments** | BTC | Buyers purchase enclave capacity with Bitcoin; node operators earn by supplying compute |

## Architecture

```
Auth (OIDC, ...) ──────▶ Control Plane ◀──Attestation──▶ Intel TDX / AMD SEV
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                 Agent 1   Agent 2   Agent N
                 (TDX VM)  (TDX VM)  (TDX VM)
                    │         │         │
                 Cloudflare Tunnels (outbound-only)
                              │
                        Marketplace
                  (BTC payments, node mgmt)
```

1. **Register** — Agent boots on a confidential VM, generates a hardware attestation quote, and registers with the control plane
2. **Verify** — Control plane validates the quote through the hardware vendor's trust authority, checking measurements and runtime state
3. **Connect** — Agent receives a Cloudflare Tunnel token and establishes an outbound-only secure tunnel
4. **Trade** — Marketplace matches buyers with verified node capacity; payments settle in BTC
5. **Deploy** — Buyer submits a Docker Compose workload to the control plane, which assigns it to a verified agent

## Repositories

| Repo | Description |
|------|-------------|
| [dd](https://github.com/devopsdefender/dd) | Monorepo — agent, control plane, VM images, infra, and website |
| [marketplace](https://github.com/devopsdefender/marketplace) | Compute marketplace — buy TDX enclave capacity with BTC, manage local GPU nodes |

## Quick Start

```bash
# Run the control plane locally
DD_CP_DATABASE_URL=sqlite://dev.db DD_CP_ADMIN_PASSWORD=changeme cargo run --bin dd-cp

# Run an agent (skip attestation for non-TDX environments)
DD_AGENT_SKIP_ATTESTATION=true DD_CP_URL=http://localhost:8080 cargo run --bin dd-agent
```

## Links

- [Website](https://devopsdefender.com)
