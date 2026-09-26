# Proxmox Server Project

A personal homelab infrastructure project built using **Proxmox VE**.

The project consists of **two physical Proxmox nodes configured as a cluster**, with multiple services deployed using **Linux Containers (LXC)**.

The primary services hosted in the environment are:

- **Pi-hole** – Network-wide DNS filtering
- **File Server** – Centralized network file storage

The project was built to gain practical, hands-on experience with virtualization, Linux administration, networking, storage, DNS, and infrastructure troubleshooting.

---

## Architecture

```text
                         Internet
                            │
                         Router
                            │
                     ┌──────▼──────┐
                     │   Network   │
                     │   Switch    │
                     └──────┬──────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
        ┌─────▼─────┐               ┌─────▼─────┐
        │ Proxmox   │               │ Proxmox   │
        │  Node 01  │◄── Cluster ──►│  Node 02  │
        │   4 GB    │               │   4 GB    │
        │    HDD    │               │   HDD     │
        └─────┬─────┘               └─────┬─────┘
              │                           │
              └─────────────┬─────────────┘
                            │
                       LXC Containers
                            │
              ┌─────────────┴─────────────┐
              │                           │
        ┌─────▼─────┐               ┌─────▼─────┐
        │  Pi-hole  │               │File Server│
        │    LXC    │               │    LXC    │
        └───────────┘               └───────────┘
```

---

## Hardware

The cluster consists of two physical systems with limited resources.

| Node             | RAM  | Storage |
|------------------|------|---------|
| Proxmox Node 01  | 4 GB | HDD     |
| Proxmox Node 02  | 4 GB | HDD     |

---

## Proxmox Cluster

Both physical systems run Proxmox VE and are configured as members of the same cluster.

The cluster provides centralized management of the virtualization environment and allows the infrastructure to be managed from the Proxmox web interface.

The environment was built using relatively low-resource hardware, making resource allocation and workload management an important part of the project.

---

## Services

### Pi-hole

Pi-hole was deployed inside an LXC container.

Pi-hole provides network-wide DNS filtering and allows DNS queries to be monitored and managed from a centralized service.

The deployment provided practical experience with:

- Linux administration
- DNS
- Network configuration
- Service management
- LXC containers
- Troubleshooting

### File Server

A file server was deployed inside an LXC container to provide centralized network storage.

The file server provides a shared location for storing and accessing files across the local network.

This provided practical experience with:

- Linux file systems
- Storage management
- File permissions
- Network file sharing
- User access
- LXC containers

---

## Technologies Used

**Virtualization**
- Proxmox VE
- LXC

**Networking**
- TCP/IP
- DNS
- DHCP
- Static IP addressing
- LAN networking
- Virtual networking

**Linux**
- Linux server administration
- Package management
- Service management
- File permissions
- Networking
- Storage

**Services**
- Pi-hole
- Network File Server

**Hardware**
- HDD storage
- Ethernet networking
- Repurposed PC hardware

---

## Project Objectives

The project was created to develop practical experience in:

- Deploying Proxmox VE
- Building a multi-node cluster
- Managing physical virtualization hosts
- Deploying LXC containers
- Managing Linux services
- Configuring DNS
- Configuring network services
- Managing storage
- Configuring network file sharing
- Troubleshooting infrastructure
- Working with limited hardware resources
- Documenting an infrastructure environment

---

## Implementation

### 1. Proxmox VE

Proxmox VE was installed on both physical systems.

Each node was configured with its own:

- Hostname
- Network configuration
- Static IP address
- Local storage
- Proxmox management interface

### 2. Cluster

The two Proxmox hosts were joined together into a single Proxmox cluster.

This allowed the nodes and their workloads to be managed through the Proxmox management interface.

### 3. LXC Containers

Instead of running the services directly on the Proxmox hosts, dedicated LXC containers were created.

The environment uses containers for:

- Pi-hole
- File Server

Using LXC allowed the services to run independently while consuming fewer resources than full virtual machines.

### 4. Pi-hole

Pi-hole was installed and configured inside its own LXC container.

The container was configured to provide DNS services to the network.

### 5. File Server

A separate LXC container was configured as a file server.

Storage and network access were configured to allow other devices on the network to access shared files.

---

## Resource Constraints

One of the challenges of this project was working with only 4 GB of RAM per Proxmox node.

Because of the limited resources, the environment required careful consideration of:

- Container resource allocation
- Storage usage
- Workload placement
- System overhead
- Service requirements

This made the project useful for understanding how infrastructure behaves when operating under constrained resources.

---

## Documentation

The `Assets` directory contains screenshots, images, configuration evidence, and other documentation from the project.

These assets document the different stages of the implementation and configuration.

---

## What I Learned

This project gave me practical experience with:

- Proxmox VE
- Proxmox clustering
- LXC containers
- Linux server administration
- DNS
- Pi-hole
- Network file sharing
- Storage management
- Static IP configuration
- Virtual networking
- Infrastructure troubleshooting
- Resource management

It also provided experience with designing and maintaining infrastructure using hardware that was already available rather than relying on enterprise-grade equipment.

---

## Future Improvements

Possible future improvements to the lab include:

- Increasing RAM on the Proxmox nodes
- Adding additional storage
- Implementing centralized storage
- Implementing automated backups
- Adding network monitoring
- Adding VLAN segmentation
- Deploying additional infrastructure services
- Implementing centralized logging
- Expanding the cluster with additional nodes

---

## Disclaimer

This is a personal homelab project created for educational and practical IT experience.

The environment is intended for experimentation, learning, testing, and infrastructure development rather than production workloads.

---

## Author

**Remeldo Stone**
