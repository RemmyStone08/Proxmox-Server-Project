# Pi-hole Setup

## Overview

Pi-hole was deployed as a lightweight LXC container on the **pve** node, providing network-wide DNS-based ad and tracker blocking for every device on the network.

An LXC container was chosen over a full VM specifically because of the constrained hardware — each node has only 4GB of RAM, and LXC containers share the host kernel rather than running a full separate OS, making them far lighter for simple services like this.

## Step 1: Create the LXC Container

In the Proxmox web UI:
1. Downloaded a Debian 12 container template (`local (pve) → CT Templates → Templates → debian-12-standard`)
2. **Create CT**, with:
   - Hostname: `PiHole`
   - Template: debian-12-standard
   - Disk: small (a few GB is sufficient)
   - CPU: 1 core
   - Memory: 512MB
   - Network: bridge `vmbr0`, DHCP (adjusted to static later)

## Step 2: Container Network Troubleshooting

On first boot, the container had no working network connection at all:

```bash
root@PiHole:~# curl -sSL https://install.pi-hole.net | bash
-bash: curl: command not found
```

Installing curl surfaced a deeper issue:

```
Temporary failure resolving 'deb.debian.org'
```

Diagnosis steps:

```bash
ip a
```
Showed `eth0` in a `DOWN` state with no IPv4 address at all.

```bash
ip link set eth0 up
dhclient eth0
ip a
```

After manually bringing the interface up and forcing a DHCP request, the container received an IP (`192.168.8.212`) and internet connectivity was confirmed with `ping -c 4 8.8.8.8`.

## Step 3: Install Pi-hole

```bash
apt update && apt upgrade -y
apt install curl -y
curl -sSL https://install.pi-hole.net | bash
```

The installer walks through an interactive setup: selecting the network interface, choosing an upstream DNS provider, and configuring blocklists.

### Static IP requirement

Pi-hole's installer requires a static IP and will pause with a warning if the interface is still on DHCP. Rather than editing `/etc/network/interfaces` manually inside the container, the static IP was configured cleanly through the **Proxmox web UI** instead:

1. Selected the PiHole container → **Network** tab
2. Edited the `net0` device: switched from DHCP to **Static**
3. Set:
   - IPv4/CIDR: `192.168.8.212/24`
   - Gateway (IPv4): `192.168.8.1`
4. Rebooted the container to apply

The Pi-hole installer was then re-run and completed successfully, confirming the static address:

```
Installation Complete!
Configure your devices to use the Pi-hole as their DNS server using:

IPv4: 192.168.8.212
View the web interface at http://pi.hole:80/admin or http://192.168.8.212:80/admin
```

## Step 4: Configure the Router to Use Pi-hole for DNS

To make every device on the network use Pi-hole automatically (rather than configuring each device individually), the router's DHCP-assigned DNS server was changed:

1. Router admin page → **LAN Setup**
2. Under **DNS Values**, changed from "DNS Proxy" to **Static**
3. Set **DNS Server 1** to `192.168.8.212`
4. Applied the change

### Preventing IP conflicts

Since `192.168.8.212` falls within the router's DHCP pool range (`192.168.8.2`–`192.168.8.254`), there was a risk the router could eventually hand that same address out to a different device via DHCP, causing a conflict with the Pi-hole container.

**Fix:** added a **Static DHCP** reservation on the router, binding `192.168.8.212` permanently to the Pi-hole container's MAC address, and set the entry to **Active**. This guarantees the router will never assign that address to anything else, on top of the address already being statically configured on the container itself.

## Result

Confirmed working via the Pi-hole dashboard, showing live queries and blocked requests from active clients on the network shortly after the router DNS change was applied.

*(See `/assets` for dashboard and Query Log screenshots.)*

## Known Limitation

Pi-hole is DNS-based, meaning it blocks requests to known ad-serving domains before a connection is made. This works well for most websites and apps, but does **not** reliably block in-video YouTube ads, since YouTube serves both video content and ads from the same domain (`googlevideo.com`). Blocking that domain would break video playback entirely, not just ads. This is a known, general limitation of DNS-level ad blocking rather than a misconfiguration.
