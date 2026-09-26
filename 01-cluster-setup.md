# Cluster Setup

## Overview

This document covers installing Proxmox VE on both nodes and joining them into a single cluster.

## Step 1: Installing Proxmox VE

Proxmox VE was installed on each laptop from a USB drive prepared with [Ventoy](https://www.ventoy.net/), which allows the ISO to simply be copied onto the drive without needing to re-flash it for each attempt.

1. Downloaded the Proxmox VE ISO from the official site
2. Copied it onto a Ventoy-prepared USB drive
3. Booted each laptop from the USB and ran the Proxmox VE installer
4. Set a root password and hostname per node (`pve` and `pve02`)

**Note:** the Proxmox installer only configures wired Ethernet during install — it does not support WiFi configuration at all. This matters for clustering specifically, since cluster communication (via Corosync) is sensitive to the latency and occasional drops inherent to WiFi, which can cause a cluster to falsely detect a node failure. A wired connection is required for a stable cluster.

### Adapter issue

The original second laptop did not have a built-in Ethernet port, and an available USB-to-Ethernet adapter did not work reliably. Rather than debug driver support for that adapter under time pressure, the second laptop was swapped for another spare (the Mecer) that had a built-in LAN port, avoiding the issue entirely.

## Step 2: Confirming Network Connectivity

After install, each node's IP was confirmed via the console:

```bash
ip a
ip route
ping -c 4 8.8.8.8
```

Both nodes were reachable at their assigned static IPs (`192.168.8.50` for pve, `192.168.8.52` for pve02) and had working internet connectivity, confirmed before proceeding.

## Step 3: Creating the Cluster

**Important lesson learned:** a node can only **join** a cluster if it has zero existing VMs/containers on it. A node **creating** a cluster can have existing guests without issue.

The first attempt at clustering was done backwards — the cluster was created on the empty node (pve02), and an attempt was made to *join* from the node that already had Pi-hole running on it (pve). This failed with:

```
detected the following error(s):
* this host already contains virtual guests
TASK ERROR: Check if node may join a cluster failed!
```

**Fix:** the cluster configuration was reset on pve02 (the node that had incorrectly created it), and the cluster was recreated on **pve** instead — the node with the existing guest — since creation isn't blocked by existing guests. pve02 (empty) then successfully joined.

To reset a half-formed cluster state on a node:

```bash
systemctl stop pve-cluster
systemctl stop corosync
killall -9 pmxcfs
systemctl start pve-cluster

# If /etc/pve files are locked/read-only, force local mode first:
systemctl stop pve-cluster
pmxcfs -l
rm -f /etc/pve/corosync.conf
rm -rf /etc/corosync/*
killall pmxcfs
systemctl start pve-cluster
systemctl reset-failed corosync
```

### Correct process:

1. On **pve** (node with existing Pi-hole guest): **Datacenter → Cluster → Create Cluster**, named it `remmy-homelab`
2. On **pve**: **Datacenter → Cluster → Join Information**, copied the generated join info
3. On **pve02** (empty node): **Datacenter → Cluster → Join Cluster**, pasted the join info, entered pve's root password, confirmed

The join briefly showed a "Connection error" popup in the browser — this is expected and transient, caused by the node's own cluster services restarting mid-join. Refreshing the page after ~30-60 seconds confirmed the join had actually succeeded.

## Result

```
Cluster Name: remmy-homelab
Config Version: 2
Number of Nodes: 2

Nodename   ID   Votes   Link 0
pve        1    1       192.168.8.50
pve02      2    1       192.168.8.52
```

Both nodes now appear under a single Datacenter view in either node's web UI, and guests can be managed/migrated across the cluster.

*(See `/assets` for screenshots of the cluster nodes list and join task output.)*
