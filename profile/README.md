# DevOps Defender

**Run AI workloads on hardware-encrypted VMs. Agents ready in seconds.**

DevOps Defender is an open-source confidential computing platform built on Intel TDX. Each agent runs in a hardware-encrypted VM with a web dashboard, terminal access, and automatic Cloudflare Tunnel networking — no open ports, no stored secrets.

## How It Works

| Layer | Technology | What It Does |
|-------|-----------|--------------|
| **Trust Root** | Intel TDX | Hardware attestation proves each VM's identity via cryptographic quotes from the CPU |
| **Auth** | GitHub OAuth / Password | Dashboard login — GitHub OAuth for the fleet, shared password for individual agents |
| **Networking** | Cloudflare Tunnels | Outbound-only tunnels per agent — no open ports, no TLS certs, no firewall rules |
| **Workloads** | Plain processes / Podman | Run shell commands or containers directly on the VM — no container runtime overhead |

## Architecture

```
Browser ──────▶ Fleet Dashboard (register agent)
                      │
            ┌─────────┼─────────┐
            ▼         ▼         ▼
         Agent 1   Agent 2   Agent N
         (TDX VM)  (TDX VM)  (TDX VM)
            │         │         │
         Cloudflare Tunnels (outbound-only)
            │         │         │
         Workloads: OpenClaw, bash, podman containers
```

1. **Boot** — Agent starts on a TDX VM, registers with the fleet via Noise-encrypted WebSocket
2. **Attest** — Hardware generates a TDX quote proving the VM's identity and integrity
3. **Connect** — Agent gets a Cloudflare Tunnel hostname (e.g. `agent-xyz.devopsdefender.com`)
4. **Run** — Workloads run as plain processes or podman containers with a web dashboard and terminal

## Repositories

| Repo | Description |
|------|-------------|
| [dd](https://github.com/devopsdefender/dd) | dd-agent — Rust binary that runs on TDX VMs, manages workloads, serves dashboard |
| [marketplace](https://github.com/devopsdefender/marketplace) | OpenClaw deployment — AI orchestrator with ollama fallback and optional H100 inference |

## Links

- [Website](https://devopsdefender.com)
- [Staging Dashboard](https://app-staging.devopsdefender.com)
