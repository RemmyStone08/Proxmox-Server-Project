# File Server Setup

## Overview

A Samba-based file server was deployed as an LXC container on the **pve02** node, providing a simple network file share accessible from Windows and other devices on the local network.

## Step 1: Create the LXC Container

In the Proxmox web UI, on pve02:
1. Downloaded a Debian 12 container template (`local (pve02) → CT Templates → Templates → debian-12-standard`)
2. **Create CT**, with:
   - Hostname: `fileserver`
   - Template: debian-12-standard
   - Disk: sized based on available free space on the node
   - CPU: 1-2 cores
   - Memory: 512MB-1GB
   - Network: bridge `vmbr0`, DHCP

## Step 2: Install and Configure Samba

```bash
apt update && apt upgrade -y
apt install samba -y

mkdir /srv/share
chmod 777 /srv/share
```

Added a share definition to `/etc/samba/smb.conf`:

```ini
[Shared]
   path = /srv/share
   browseable = yes
   read only = no
   guest ok = yes
```

```bash
systemctl restart smbd
```

## Step 3: Windows Guest Access Issue

Attempting to connect from a Windows machine (`\\192.168.8.245\Shared`) failed with:

```
You can't access this shared folder because your organization's security
policies block unauthenticated guest access.
```

This is a Windows-side security policy (introduced in more recent Windows versions), which blocks unauthenticated SMB guest access by default — even though the Samba config explicitly allowed guest access (`guest ok = yes`). Rather than weaken Windows' own security settings to force guest access through, a proper authenticated Samba user was created instead.

**Fix:**

```bash
useradd -M -s /usr/sbin/nologin shareuser
smbpasswd -a shareuser
```

Updated the share definition to require that user:

```ini
[Shared]
   path = /srv/share
   browseable = yes
   read only = no
   guest ok = no
   valid users = shareuser
```

```bash
systemctl restart smbd
```

Connecting again with the `shareuser` credentials succeeded.

## Result

The share is accessible at `\\192.168.8.245\Shared` from Windows using the `shareuser` account, and was pinned to Quick Access for convenient ongoing use.

*(See `/assets` for the Windows credentials prompt and successful connection screenshots.)*
