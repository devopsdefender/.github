# DevOps Defender

**Run confidential workloads on hardware-encrypted VMs. Owner-scoped, attested, and wired to the public internet with one call.**

DevOps Defender is an open-source platform for running AI and other
workloads inside Intel TDX enclaves. Each agent lives in a
hardware-encrypted VM, speaks back to a control plane through an
outbound-only Cloudflare tunnel, and exposes any port a workload
needs at `<label>.<agent-hostname>` — ingress is provisioned
dynamically as workloads come and go.

## How it works

| Layer            | Technology                                              | What it does                                                                                                   |
|------------------|---------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| **Trust root**   | [Intel TDX][tdx] + [Intel Trust Authority][ita]         | CPU-signed attestation quote on every register + `/health` poll; the CP drops agents whose ITA token fails.    |
| **Transport**    | Cloudflare Tunnels (outbound-only)                      | No open ports, no TLS certs, no firewall rules. Per-workload ingress rules are added to the tunnel at runtime. |
| **Auth**         | GitHub OAuth (fleet) / shared PAT (individual agents)   | Owner-scoped: a PAT belonging to the agent's `DD_OWNER` org can deploy, view, and mop any agent in the fleet.  |
| **Workloads**    | Raw processes or podman containers via [easyenclave][ee] | Spawned from a tiny JSON `DeployRequest` — no OCI registry assumed, github-release assets work too.            |

## Lifecycle

1. **Boot.** The agent starts as a workload under easyenclave inside a
   TDX VM, mints an ITA attestation token, and POSTs it to the control
   plane at `$DD_CP_URL/register`. On success it gets a Cloudflare
   tunnel token and a JWT signing secret.
2. **Attest.** Every `/health` poll carries a fresh ITA-signed JWT. The
   CP re-verifies each token and drops agents whose attestation has
   expired, been revoked, or drifted.
3. **Connect.** Cloudflared comes up outbound-only; the agent is
   reachable at `<tunnel-name>.devopsdefender.com`. Workloads that
   declare `expose: {hostname_label, port}` get a public hostname
   `<label>.<tunnel-name>.devopsdefender.com` routed to
   `localhost:<port>` — added at register time for boot workloads, at
   `/deploy` time for runtime workloads.
4. **Run.** POST JSON specs to `/deploy` on the agent. Status, logs,
   and a live web terminal are on the dashboard at the agent's
   hostname. Owner-scoped GitHub PAT is all the auth you need.

## Repositories

| Repo                              | Description                                                                              |
|-----------------------------------|------------------------------------------------------------------------------------------|
| [dd](https://github.com/devopsdefender/dd) | The `devopsdefender` Rust binary. One crate, two modes (`DD_MODE=management` for the control plane; `DD_MODE=agent` for VMs). Ships the fleet dashboard, the `/register` + `/deploy` + `/ingress/replace` API, the ITA verifier, and the Cloudflare tunnel CRUD. |

## Ecosystem

- [easyenclave][ee] — the bare runtime that hosts dd-agent (and
  anything else) inside a TDX VM: unix-socket workload API, PID 1, no
  networking, no HTTP.
- [slopandmop][smp] — catalog of reference workloads you can compose
  and deploy onto a DD agent. The home of the ollama + openclaw LLM
  stack that used to live in this repo's `apps/`.

## Links

- [devopsdefender.com](https://devopsdefender.com)
- [Production fleet dashboard](https://app.devopsdefender.com)
- [PR preview (active PRs only)](https://app.devopsdefender.com)

[tdx]: https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html
[ita]: https://trustauthority.intel.com/
[ee]: https://github.com/easyenclave/easyenclave
[smp]: https://github.com/slopandmop/slopandmop
