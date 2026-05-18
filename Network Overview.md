
# 1. Introduction

## 1.1 Purpose of the Network
This is a personal networking project built to develop practical networking skills, with a focus on CLI and security principles. It simulates a small enterprise-style network to get hands-on experience with routing, VLAN segmentation, and access control.

The network is split into different zones for internal users and services, guest users, and public services (DMZ). Traffic between these areas is controlled using router and firewall ACLs to enforce separation and basic security rules.

This page provides a basic overview of the network. *Device Configuration.md* contains the full device configurations for all routers, switches, firewalls, and other devices used in the project. It also contains Packet Tracer limitations encountered with different devices. In addition, *Simulated Attacks.md* contains different security tests that were performed to evaluate the network’s ACLs, segmentation, and overall resilience.

Note: When running *limoNetwork.pkt* in Packet Tracer, DHCP addresses may be assigned in the wrong order. To fix this, temporarily set all PCs to static IP configuration, save the DHCP server pools, and then request DHCP addresses again one device at a time in ascending IP order.

---

## 1.2 Topology Overview
The network is structured around a central firewall/router that controls traffic between all zones. It acts as the main security point where inter-VLAN and inter-network traffic is filtered using ACLs.

The internal network is divided into VLANs to separate different departments and user groups. This allows each group to be isolated while still sharing the same physical infrastructure.

A DMZ is used to host a public-facing web server that is accessible from external networks under controlled rules. The guest network is fully isolated from internal systems to ensure security, with the exception of limited access granted to an internal administrator for management purposes.

---

## 1.3 Topology Diagrams

### Logical Topology
![[Network-Project/images/logicaltopology.png]]

#### Traffic Flow

| Source        | Destination  | Status            | Notes                                         |
| ------------- | ------------ | ----------------- | --------------------------------------------- |
| Inside VLANs  | Internet     | Allowed           | Limited access (HTTP/DNS)through NAT/firewall |
| Inside VLANs  | Guest        | Restricted        | Limited to admin access                       |
| Inside VLANs  | DMZ          | Allowed           | Limited to HTTP                               |
| DMZ           | Anywhere     | Denied by default | Only established/related traffic allowed      |
| Guest Network | Internet     | Allowed           | HTTP and DNS access only                      |
| Guest Network | Inside VLANs | Denied            | Blocked using ACLs                            |
| Guest Network | DMZ          | Restricted        | Limited HTTP                                  |

#### Notes

| Component                 | Description                                                        |
| ------------------------- | ------------------------------------------------------------------ |
| R1 (Inside Router)        | Uses ACLs to control traffic between VLANs                         |
| Firewall                  | Uses ACLs with default deny between zones                          |
| Guest Network             | Isolated from internal VLANs                                       |
| Packet Tracer Limitations | Some ACL behaviour differs from real stateful firewall deployments |

### Physical Topology
![[physicaltoplogy.png]]
 
---

## 1.4 IP Addressing Scheme

| Zone             | Network              | Purpose           |
| ---------------- | -------------------- | ----------------- |
| Inside (Admin)   | 192.168.20.11 (host) | Admin privileges  |
| Inside (Servers) | 192.168.10.0/24      | Internal services |
| Inside (IT)      | 192.168.20.0/24      | Staff devices     |
| Inside (FIN)     | 192.168.30.0/24      | Staff devices     |
| Inside (HR)      | 192.168.40.0/24      | Staff devices     |
| Guest            | 192.168.50.0/24      | Internet-only     |
| DMZ              | 192.168.60.0/24      | Public services   |

---

## 1.5 VLAN Overview

| VLAN | Name       | Purpose                |
| ---- | ---------- | ---------------------- |
| 5    | Management | Network administration |
| 10   | Servers    | Internal Services      |
| 20   | IT         | General staff          |
| 30   | FIN        | General staff          |
| 40   | HR         | General staff          |
| 50   | Guest      | Guest users            |
| 60   | DMZ        | Public services        |

---

## 1.6 Naming Conventions

- Routers: R1, R2
- Switches: SW1, SW2, SW3, SW4
- Firewall: FW1
- Servers: DHCP-SRV, WEB-SRV, MAIL-SRV, SYSLOG-SRV, FTP-SRV
- VLANs: based on function (5/10/20/30/40/50/60)
- PCs: IP address

---

# 2. Network Zones

This section defines how traffic is controlled between different security zones.

---

## 2.1 Inside Network

### Purpose
Internal users and administrators.

### VLANs
- VLAN 5: Management (192.168.20.0/24)
- VLAN 10: Servers (192.168.20.0/24)
- VLAN 20: IT (192.168.20.0/24)
- VLAN 30: FIN (192.168.30.0/24)
- VLAN 40: HR (192.168.40.0/24)

### Devices and Servers in Zone
- R1, SW1, SW2
- User PCs
- Admin PC (192.168.20.11)
- DHCP, DNS, Email, Syslog, and FTP servers

### Allowed Traffic
- HTTP, DNS to internet
- DHCP, DNS, SMTP, FTP to internal/external servers
- Limited access to DMZ web services (HTTP only for users)
- Full admin access to DMZ and Guest

---

## 2.2 DMZ (Demilitarized Zone)

### Purpose
Hosts public-facing services while isolating internal network.

### VLANs
- VLAN 60: DMZ (192.168.60.0/24)
### Devices and Servers in Zone
- SW3
- Web Server (192.168.20.11)

### Firewall Policies
- Allow HTTP from outside, internal users, and guest network.
- Block DMZ initiating connections anywhere.
- Allow return traffic automatically (stateful firewall)

---

## 2.3 Guest Network
### Purpose
Provide guests with Internet and web server access.

### VLANs/Subnets
- VLAN 50: Guest

### Devices in Zone
- SW4
- Guest PC
- AP1 (access point)
- Guest smartphone

### Firewall Policies
- Access to Internet
- No access to Inside networks
- Limited access to DMZ web server (HTTP only)

---

## 2.4 Outside / WAN
### Purpose
Simulate ISP Router and Internet.

### VLANs/Subnets
- VLAN 50: Guest

### Devices in Zone
- R2
- PC (Simulated public DNS)

### ISP Connection
- Connection to external internet via ISP router (R2)
- Assigned WAN IP (Public IP): 203.0.113.2

### NAT Overview
- Internal IPs are dynamically converted to Public IP (configuration in Device Configuration page).

---

# 3. Security Policy Overview

This network uses ACLs to control traffic between different segments, only allowing communication that is explicitly permitted. The firewall is configured with stateful inspection to track active connections and allow return traffic automatically. In addition, R1 (inside router) is configured with an ACL to control traffic between VLANs and enforce internal segmentation.

For details on how this was implemented, refer to *Device Configuration.md*. To see how the network handles attacks, refer to *Simulated Attacks.md*
