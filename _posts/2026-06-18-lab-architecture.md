---
title: "Designing a Cybersecurity Home Lab"
date: 2026-06-18 +1030
categories: [Cybersecurity, Networking, Homelab, CTF]
tags: [cybersecurity, pentesting, ethical hacking, kali linux, CTF]
---

## Designing a Cybersecurity Home Lab: Architecture, Network Segmentation, and Design Principles

### Introduction

Building a cybersecurity home lab is often misunderstood as simply installing Kali Linux and a few vulnerable virtual machines. In reality, a proper lab behaves more like a **small enterprise network**, where segmentation, routing, and controlled access matter just as much as the tools used to attack it.

This article documents the **architecture and design decisions** behind my setup. Instead of focusing on commands or configuration files, the goal is to understand:

> **Why the lab is built this way, and how all components interact.**

This lab will be used throughout the upcoming penetration testing series.

### Design Goals

Before building the lab, I defined a few core objectives:

- Simulate a real-world segmented enterprise network.
- Isolate vulnerable systems from the main home network.
- Practice VLAN-based network segmentation.
- Use real networking hardware where possible.
- Build a scalable environment for future expansion.
- Enable safe testing using snapshots and rollback.
- Keep the design cost-effective using second-hand hardware.

These goals shaped every architectural decision.

### High-Level Architecture

The lab is divided into four conceptual layers:

- Internet Access Layer  
- Edge & Routing Layer  
- Virtualization Layer  bundl
- Physical Access Layer  

### Full Network Architecture

#### Cybersecurity Home Lab Network Architecture

![Network Architecture](/assets/img/blog14/1-network-architecture.webp)
*Figure 1: Cybersecurity Home Lab Network Architecture*

This architecture illustrates:

- Internet connectivity entering through a wireless bridge
- A Proxmox-based virtualization host
- pfSense acting as the central firewall and router
- VLAN-based segmentation for isolation
- Physical separation between attacker and management devices

### 🖥️ Physical Network Infrastructure

![Physical topology](/assets/img/blog14/2-physical-network-topology.webp)
*Figure 2: Physical Network Topology*

The physical layer connects the lab to the outside world and provides access to the virtualization environment.

### Key Hardware Components

|             Component                |                  Purpose                  |
|--------------------------------------|-------------------------------------------|
|         ISP Provider Router          |           ISP internet gateway            |
|        Netgear Nighthawk M2          | Wireless bridge due to access limitations |
| TP-Link PCI express Ethernet Adapter |       WAN connection to Proxmox host      |
|          Dell OptiPlex 5050          |        Primary virtualization server      |
|         Cisco Catalyst 2960X         |       Managed switch for VLAN learning    |
|          Kali Linux Laptop           |          Dedicated attack machine         |
|         Management Desktop           |        Administrative access system       |

### Core Virtualization Platform (Proxmox)

![Virtualization architecture](/assets/img/blog14/3-virtualization-architecture.webp)
*Figure 3: Virtualization Architecture (Proxmox Environment)*

All virtual machines and containers run on **Proxmox VE** installed on the Dell OptiPlex.

### Why Proxmox?

- Open-source and free  
- Strong community support  
- Supports both VMs and containers  
- Built-in snapshot and rollback functionality  
- Lightweight and efficient on consumer hardware  

Proxmox serves as the foundation of the entire lab.

### Network Routing (pfSense)

A virtual **pfSense firewall** acts as the central network brain of the lab.

#### Responsibilities

- WAN → LAN routing  
- DHCP services  
- Inter-VLAN routing  
- Firewall rules and segmentation  

#### Interfaces

- **WAN (PCIe NIC)** → Internet via TP-Link adapter  
- **LAN (Onboard NIC)** → Cisco switch trunk connection  

pfSense enforces all traffic rules between VLANs.

#### Extended Capability (Suricata IDS/IPS)

Suricata is installed on pfSense and is actively used to monitor network traffic.

It generates alerts for suspicious activity such as:

- Port scans
- Brute force attempts
- Known attack signatures

These alerts will become important in later stages of this series when we move into attack detection and analysis.

#### VLAN-Based Network Segmentation

The lab uses VLANs to simulate enterprise-level network separation.

| VLAN |    Name    |             Purpose              |
|------|------------|----------------------------------|
|  10  |    USERS   |    Windows VMs for future use    |
|  20  |   SERVERS  | Vulnerable machines and services |
|  30  |   ATTACK   |     Kali Linux attack network    |
|  99  | MANAGEMENT |   Administrative access network  |

### VLAN Assignment

Each VM is assigned a VLAN tag at the virtual NIC level. This ensures segmentation is enforced at the virtualization layer before traffic reaches the physical network.

### Virtual Machines & Containers

All vulnerable systems run inside Proxmox.

#### Virtual Machines

- Metasploitable 2  
- Metasploitable 3  
- Ubuntu Server  
- Windows Server 2022  
- Windows 10  
- Windows 11  

#### Containers (LXC)

- DVWA  
- OWASP Juice Shop  

### Physical Access Layer

The Cisco switch connects physical devices into the lab network.

### Port Mapping

#### Why Dedicated Switch Ports?

The Kali Linux laptop and management desktop are connected to dedicated access ports on the Cisco switch, each assigned to a specific VLAN. This ensures the devices automatically join the correct network, simplifies administration, and accurately reflects enterprise networking practices where attacker workstations, administrative systems, and servers are isolated into separate security zones. It also makes firewall rule management and troubleshooting more predictable as the lab continues to grow.

|    Device     |   Port   |   VLAN  |
|---------------|----------|---------|
|  Proxmox Host | Gi1/0/1  |  Trunk  |
|  Kali Laptop  | Gi1/0/10 | VLAN 30 |
| Management PC | Gi1/0/23 | VLAN 99 |

### Network Flow Example

![Attack traffic flow](/assets/img/blog14/4-attack-traffic-flow.webp)
*Figure 4: Attack Traffic Flow Example*

### Understanding the Attack Traffic Flow

The attack traffic flow illustrates how communication travels through the lab during a typical penetration testing exercise. Rather than connecting the attacker directly to the target system, every packet follows the same path it would in a properly segmented enterprise network.

When the Kali Linux laptop sends traffic towards a vulnerable machine, the request first reaches the Cisco Catalyst switch through its dedicated access port on VLAN 30 (Attack). The switch forwards the traffic over the trunk link to the Proxmox host, where it is processed by the pfSense virtual firewall.

Because the target systems reside on VLAN 20 (Servers), pfSense performs inter-VLAN routing and applies any configured firewall rules before allowing the traffic to continue. Once approved, the traffic is forwarded to the destination virtual machine or container hosted within Proxmox.

This design provides several advantages:

- Realistic network segmentation – The attacker and target systems are placed on separate VLANs, closely resembling how enterprise environments are structured.
- Centralised traffic control – All communication between VLANs passes through pfSense, allowing routing decisions and security policies to be enforced in a single location.
- Improved visibility – Since all inter-VLAN traffic traverses the firewall, monitoring tools such as Suricata can inspect network activity and generate alerts for suspicious behaviour.
- Safer experimentation – Vulnerable machines remain isolated from the management network and the rest of the home environment, reducing the risk of accidental exposure.

Although the traffic path is relatively simple, it demonstrates an important concept that will be used throughout this series: every stage of a penetration test—from information gathering and scanning to exploitation and post-exploitation—follows the same underlying network path. Understanding how traffic moves through the lab makes it easier to interpret scan results, troubleshoot connectivity issues, and appreciate the role of each component in the overall architecture.

### Snapshots & Recovery Strategy

Snapshots are a key part of the workflow.

They are taken:

- After clean VM setup  
- Before penetration testing  
- Before major configuration changes  

This allows instant rollback if something breaks.

### Internet Access Strategy

All VLANs currently have internet access.

This allows:

- Package updates  
- Tool installation  
- Easier lab setup and maintenance  

Future versions may restrict access for realism.

### Design Philosophy

The lab is built around a few core principles:

- Isolation is more important than complexity  
- Real hardware improves learning  
- Virtualization improves flexibility  
- VLANs simulate enterprise environments  
- Snapshots enable safe experimentation  

### Future Expansion

This lab is designed to grow over time.

Planned additions include:

- Active Directory environment  
- Wazuh SIEM integration  
- Suricata monitoring improvements  
- Attack detection dashboards  
- More enterprise simulation scenarios  

### Conclusion

This lab is not just a collection of virtual machines. It is a structured cybersecurity training environment designed to simulate real-world enterprise networks. Understanding this architecture is essential before moving into penetration testing techniques, because every scan, enumeration, and exploit in future articles will operate within this environment.

Happy Hacking and stay secure.
