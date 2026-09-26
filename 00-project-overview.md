# Remmy Homelab — 2-Node Proxmox Cluster

## Purpose

This project documents the build of a small, functional homelab environment using two repurposed laptops running clustered Proxmox VE. The goal was to demonstrate practical virtualization, clustering, and network service deployment skills using modest, older hardware — not enterprise-grade equipment — to prove the concepts hold up regardless of budget.

The build includes:
- A 2-node Proxmox VE cluster
- Pi-hole for network-wide DNS ad/tracker blocking
- A Samba-based file server for local network file sharing

This was built under a self-imposed deadline while preparing for a technical interview, so the documentation also captures the real troubleshooting process — including mistakes made and corrected — rather than only the clean final result.

## Hardware

| Node | Model | CPU | RAM | Storage | Role |
|---|---|---|---|---|---|
| pve | Dell Inspiron 15 | Intel Core i3 (4th Gen) | 4GB | 500GB HDD | Cluster node 1, Pi-hole host |
| pve02 | Mecer laptop | Intel Celeron | 4GB | 500GB HDD | Cluster node 2, file server host |

Neither machine is high-spec hardware. The intent was to show that a working cluster, DNS filtering, and file sharing don't require powerful or expensive equipment — just correct configuration.

## Software Stack

- **Proxmox VE** — hypervisor and clustering layer
- **Pi-hole** (LXC container) — network-wide DNS-based ad/tracker blocking
- **Samba** (LXC container) — local network file sharing

## Network Overview

| Device | IP Address | Notes |
|---|---|---|
| Router (Zyxel) | 192.168.8.1 | DHCP + DNS configuration |
| pve (node 1) | 192.168.8.50 | Static |
| pve02 (node 2) | 192.168.8.52 | Static |
| Pi-hole (LXC on pve) | 192.168.8.212 | Static, DHCP-reserved on router |
| File server (LXC on pve02) | 192.168.8.245 | Samba share |

## Documentation Structure

- [`01-cluster-setup.md`](./01-cluster-setup.md) — Proxmox installation and clustering the two nodes
- [`02-pihole-setup.md`](./02-pihole-setup.md) — Deploying and configuring Pi-hole
- [`03-file-server-setup.md`](./03-file-server-setup.md) — Deploying and configuring the Samba file server
- [`/assets`](./assets) — Screenshots referenced throughout the documentation

## Key Takeaways

- Clustering requires a stable, wired network connection between nodes — WiFi is not supported for this by Proxmox and was not attempted
- LXC containers are a far lighter option than full VMs for lightweight services on constrained RAM (4GB per node), and were used for both Pi-hole and the file server
- Real-world troubleshooting (DNS resolution inside containers, cluster join order, Windows SMB guest-access restrictions) is documented as encountered, since working through these issues is as much a part of the skillset as the final working state
