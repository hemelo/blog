---
title: 'Proxmox for Developers: Why You Should Run a Local Lab'
published: 2025-08-31
draft: false
description: 'How a local Proxmox server accelerates development with fast VMs, snapshots, CI, and safe infrastructure experiments.'
tags: ['proxmox', 'virtualization', 'homelab', 'devops', 'linux', 'kvm', 'infrastructure']
---

I run a small Proxmox box at home. It replaced old laptop setups and pointless cloud sandboxes. I keep clean VM templates, clone in seconds with cloud‑init, and lean on snapshots when I want to try a risky database migration or kernel upgrade. I also run self‑hosted CI runners and short‑lived preview environments right next to my code. With a simple bridge and a couple of VLANs, I can model “real” networks without touching the cloud.

Proxmox VE is a free, Debian/KVM‑based virtualization platform. One machine becomes a lab where you can iterate quickly, test safely, and match production‑like conditions without surprise bills or a fragile laptop.

## TL;DR

- Fresh VMs in seconds from templates no more polluting your laptop
- Snapshots for safe experiments; great for database and OS changes
- Test across distros (and Windows) on one box
- Local CI runners for faster builds and private secrets
- Realistic networks with subnets, load balancers, and firewalls
- No idle cloud costs; works offline on your hardware

## Why I switched

After years of juggling Docker, random cloud VMs, and scripts that drift, a local Proxmox lab made my week smoother.

- **Fast feedback**: I keep one solid template per distro. Need a clean env? Clone, boot, done.
- **Low‑stress experiments**: Snap, try the upgrade, and roll back if it goes sideways.
- **Closer to reality**: Multi‑VM topologies, real networking, and proper load balancers.
- **Cross‑platform**: I test installers on Ubuntu, Debian, and Rocky Linux. Windows when needed.
- **CI that stays local**: Dedicated runners cut my build times and keep tokens off the internet.
- **Costs under control**: I stopped paying for idle cloud sandboxes.

## What you need

You don’t need a server rack. My current box is a refurbished workstation (8 cores, 32 GB RAM) and it handles a lot. Aim for:

- **CPU/RAM**: 6–12 cores, 32–64 GB RAM
- **Storage**: NVMe for VM disks, a separate HDD or slower SSD for backups/ISOs
- **Virtualization**: Make sure VT‑x/AMD‑V is enabled in BIOS

For storage, I use ZFS. Snapshots and replication are handy, and compression saves space. For networking, Proxmox creates `vmbr0`, a bridge that connects VMs to your LAN. Add VLANs later if you want separation.

Set up cloud‑init templates for the distros you use most. I keep Ubuntu, Debian, and Rocky Linux ready.

## Getting it running

Here’s the flow I use:

1. **BIOS**: Enable Intel VT‑x or AMD‑V. Turn on IOMMU if you may do passthrough.
2. **Installer**: Download the Proxmox VE ISO and write it to a USB drive (Rufus, Balena Etcher, or `dd`).
3. **Install**: Boot from USB. I pick ZFS for easy snapshots. Use a static IP.
4. **Network**: Check that `vmbr0` exists and is linked to your NIC. Add VLANs if you plan to segment later.
5. **First login**: Go to `https://your-proxmox-ip:8006` and log in as `root@pam` with the password you set.
6. **Updates**: Run updates in the UI, or use `apt update && apt full-upgrade`.
7. **Storage**: Create pools for ISOs/backups and a fast SSD pool for VM disks. Turn on ZFS compression.
8. **Templates**: Upload ISOs or grab cloud images (Ubuntu/Debian). Enable cloud‑init and save as templates.

If you like automation, add Terraform (Proxmox provider) and Ansible. Treat the lab like code so you can rebuild it or share it with teammates.

### Quick app installs (LXC/VM)

The Proxmox VE Helper‑Scripts project has hundreds of scripts to stand up common services in LXC or VMs. Great for bootstrapping a lab.

::::tip
 Open and visualize the script before you run it. Know what it’s doing and what it’s changing.
::::

## Common workflows

These are the ones I repeat most weeks:

- Feature envs: clone template → provision (cloud‑init/Ansible) → test → destroy
- Database changes: snapshot → migrate/benchmark → roll back
- Systems work: multi‑VM clusters with real subnets and load balancers
- Security testing: isolated networks, packet capture, and firewalls away from your home LAN

## What keeps it stable

The habits that prevent drift:

- Automate with Terraform + Ansible so rebuilds are consistent
- Back up on a schedule; test restores; replicate ZFS to another disk/NAS
- Keep templates minimal, patched, and tagged with notes
- Turn on email alerts and collect metrics/logs

## When to skip it

If you live fully on a laptop, don’t touch infra, or only build front‑end apps, WSL or Docker Desktop may be enough. For most other cases, a small Proxmox box pays for itself in speed, safety, and learning.

---

A local Proxmox server gives you a reliable, production‑like playground that you control. Build once, clone cleanly, break things safely, and ship with confidence.
