---
title: 'LXC Containers for Developers: What They Are and When to Use Them'
published: 2025-09-01
draft: false
description: 'A practical overview of LXC: what it is, how to build containers, and when to choose LXC over VMs or Docker.'
tags: ['lxc', 'containers', 'linux', 'virtualization', 'homelab', 'devops', 'proxmox']
---

LXC feels like running tiny Linux machines that boot in seconds. You get a full Linux userspace with `systemd`, packages, and services—without VM overhead—because containers share the host kernel. I use it a lot in my homelab on Proxmox, and on plain Debian/Ubuntu when I want something that behaves like a normal OS without the weight of a VM.

## TL;DR

- LXC runs full OS userspace with the host kernel; faster and lighter than VMs
- Great for long‑running services, lab environments, and CI runners
- Prefer unprivileged containers for better isolation; use privileged only if needed
- Snapshots and templates make rebuilds fast and predictable
- Not as isolated as VMs; kernel bugs affect all containers on the host

## What LXC is (and how it differs)

- **LXC vs Docker**: Docker focuses on app containers and image layers. LXC is a full system container—you get `systemd` and traditional init flows. Docker is ideal for microservices; LXC is ideal for long‑lived services that expect a normal OS.
- **LXC vs VMs**: VMs emulate hardware and run their own kernels. LXC shares the host kernel, so it uses fewer resources and starts much faster. The trade‑off is weaker isolation than VMs.

## Where LXC fits in my work

- Dev databases I can break: I keep Postgres/MySQL in LXC with snapshots so I can try risky migrations, then roll back in seconds.
- Small, always‑on services: reverse proxies, uptime checks, runners, tiny APIs. They want a real OS but not a whole VM.
- Preview environments: clone from a template, tweak `.env`, test, destroy. Feels fast and disposable.
- Lab networks: a handful of CTs on VLANs behind a load balancer to rehearse changes before production.

## Creating LXC containers on Proxmox (UI)

1. In the sidebar, click your node → Templates → download a base image (e.g., Debian/Ubuntu).
2. Click Create CT and choose:
   - **Unprivileged**: default and safer for most workloads
   - **CPU/RAM**: set limits per container
   - **Storage**: container root disk on SSD; mount extra volumes as needed
   - **Network**: attach to `vmbr0` (bridge) and set DHCP or static IP
   - **Features**: enable `nesting` only if you must run Docker/Podman
3. Start the container and open the console to configure packages and services.

## Creating LXC on Debian/Ubuntu (CLI)

```bash
sudo apt update && sudo apt install -y lxc lxc-templates uidmap
sudo lxc-create -n web -t download -- -d debian -r bookworm -a amd64
sudo lxc-start -n web
sudo lxc-attach -n web   # shell inside the container
sudo lxc-stop -n web
sudo lxc-destroy -n web  # remove when done
```

If your user is set up for unprivileged containers, prefer `unprivileged` profiles. They map container root to non‑root on the host (via idmap), which reduces blast radius.

### Pros

- **Lightweight**: lower CPU/RAM overhead than VMs; near‑instant boots
- **Full OS**: run `systemd`, cron, syslog, and regular packages
- **Simple ops**: easy backups, snapshots, and clones (especially in Proxmox)
- **Density**: pack many services per host with clear quotas
- **Networking**: bridges and VLANs make lab topologies straightforward

### Cons

- **Kernel sharing**: a host kernel bug can impact all containers
- **Isolation**: weaker security boundaries than VMs
- **Privileged CTs**: convenient but risky; avoid unless required
- **Kernel modules**: workloads needing custom modules may be blocked
- **Nested containers**: Docker‑in‑LXC needs `nesting=1` and cgroup v2 tweaks

## Gotchas from my own setups

- Docker inside LXC: turn on `nesting` and stick to cgroup v2. Some images still expect privileged features; if it fights me, I move that workload to the host or a VM.
- Privileged vs unprivileged: default to unprivileged. If device passthrough forces privileged, I isolate it on a trusted node and write down the reason.
- Backups that actually restore: I keep mutable data on bind mounts and snapshot the rootfs. I’ve restored the wrong thing before—testing restores saved me.
- Networking surprises: macvlan makes host↔container traffic awkward; bridges are simpler in Proxmox. VLANs are great, but I double‑check firewall rules.
- Kernel is shared: a host reboot takes every container down. I schedule maintenance windows like I would for VMs.
- UID/GID mapping: unprivileged idmap can make host file ownership look odd. Consistent `subuid/subgid` ranges and mounted data directories keep me sane.

## Common workflows

- Always‑on services: reverse proxies, small APIs, runners, schedulers
- Databases for development and testing (with regular snapshots)
- Preview environments: clone from template → configure → test → destroy
- Lab clusters: multiple CTs on realistic networks with VLANs and load balancers

## Best practices

- Create **unprivileged** containers by default; switch to privileged only when necessary
- Set CPU/RAM/disk quotas up front; avoid noisy neighbor issues
- Keep base images minimal; automate post‑create steps with cloud‑init or Ansible
- Put mutable data on separate mounts so backups and restores are clean
- Test snapshots and restores; schedule backups (Proxmox makes this easy)
- If you must run Docker inside LXC, enable `nesting`, use cgroup v2, and expect some friction

::::tip
Prefer unprivileged containers. If something demands privileged mode (e.g., device passthrough), isolate it on a node you trust and document why.
::::

## LXC, Docker, or a VM?

- **Use LXC** when you want a light, fast, full Linux environment with `systemd` and long‑running services.
- **Use Docker/Podman** for app packaging, image distribution, and microservices.
- **Use a VM** when you need strong isolation, a different kernel, or kernel modules.

---

LXC containers give you a middle ground: faster than VMs, more “OS‑like” than Docker. For labs and many services, it’s a clean, reliable default.
