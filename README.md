# TechPoint Solutions SOHO Network Redesign

A network redesign for **TechPoint Solutions**, a small business with 15 employees in a two-floor office. Built in Cisco Packet Tracer 9.0.1

**Author:** Damilola Ajele

## Overview

The company uses one ISP-provided wireless router, and everything shares one flat network: employees, guests, printers, IP cameras and the NAS. This causes slowdowns when cameras record, a shared Wi-Fi password for guests and staff, open access to the NAS, no management strategy, and a full internet loss when the connection fails.

The design splits the network into five VLANs. Two multilayer switches route between them and enforce ACLs. Two edge routers connect to the ISP-provided router. A wireless LAN controller (WLC) manages the access points.

## Objectives

- Separate employees, guests, cameras, management and the NAS.
- Give guests internet access only.
- Isolate cameras so recording traffic does not slow down staff.
- Allow only employees (and the cameras, for recordings) to reach the NAS.
- Manage devices securely from a dedicated management VLAN.
- Keep internet access if one edge router fails.
- Support growth from 15 to about 30 employees and from 4 to 10 cameras.

## Environment and Devices

Current: 8 wired PCs, 7 Wi-Fi laptops, 2 printers, 4 IP cameras, 1 NAS, 2 access points, 1 ISP connection.

All devices are simulated in Packet Tracer. Device models are not specified. The cameras are represented by PCs (Packet Tracer limitation).

| Device | Qty | Role |
|--------|-----|------|
| ISP-provided router | 2 | Connects to the ISP and holds the default route |
| Edge routers | 2 | Route to the ISP-provided router. Both are active and both perform PAT |
| Multilayer switches (Switch 1, Switch 2) | 2 | VLANs, default gateways (SVIs), inter-VLAN routing, ACLs. Switch 2 also runs DHCP |
| Wireless LAN controller | 1 | Manages the access points and maps SSIDs to VLANs |
| Access points | 2 | Wi-Fi |
| PCs / laptops / printers | 8 / 7 / 2 | Employee devices (VLAN 10) |
| IP cameras | 4 | VLAN 30, connected to Switch 1 |
| NAS | 1 | VLAN 50, connected to Switch 2 |

## Topology

<!-- Add network topology diagram here -->

One ISP connection goes to the ISP-provided router, which connects to both edge routers. Edge Router 1 connects to Switch 1 and Edge Router 2 connects to Switch 2. The switches are linked to each other. Endpoints and access points connect to the switches. The WLC manages the access points.

## VLANs and IP Addressing

| VLAN | Name | Subnet | Gateway | DHCP Range | Static / Reserved |
|------|------|--------|---------|------------|-------------------|
| `10` | Employees | `10.0.1.0/25` | `10.0.1.126` | `10.0.1.11 - .125` | `10.0.1.1 - .10` |
| `20` | Guests | `10.0.4.0/25` | `10.0.4.126` | `10.0.4.11 - .125` | `10.0.4.1 - .10` |
| `30` | IP cameras | `10.0.3.0/27` | `10.0.3.30` | `10.0.3.11 - .29` | `10.0.3.1 - .10` |
| `50` | NAS/Server | `10.0.5.0/27` | `10.0.5.30` | `10.0.5.11 - .29` | `10.0.5.1 - .10` |
| `99` | Management | `10.0.2.0/27` | `10.0.2.30` | `10.0.2.11 - .29` | `10.0.2.1 - .10` |

- **Static IPs:** gateways, NAS, printers, access points, WLC and switches.
- **DHCP reservations:** cameras, so the NAS can identify each camera.
- **Normal DHCP:** PCs and laptops.
- Other /30 networks listed in the design (purpose not specified): `203.0.113.0/30`, `203.0.113.4/30` (ISP connection) and `192.168.1.0/30`, `192.168.1.4/30` (edge routers).

## Services and Security

- **Inter-VLAN routing:** SVIs on both active multilayer switches.
- **DHCP:** Switch 2 only, for all VLANs.
- **DNS:** listed as "Edge Router/ISP DNS" for every VLAN.
- **NAT/PAT:** on both edge routers.
- **Routing:** edge routers advertise routes using OSPF. The ISP-provided router holds the default route.
- **Firewall function:** ACLs on the switches and routers. No separate firewall.
- **Management:** from VLAN 99 only, using SSH (not Telnet), strong passwords and unused ports turned off. For now, VLAN 99 can only reach Switch 1 and Switch 2 remotely.

### Access Policy

| Source | Destination | Action | Service |
|--------|-------------|--------|---------|
| Guest | Internet | Allow | HTTP, HTTPS, DNS |
| Guest | Employees, NAS, Cameras, Management | Deny | Any |
| Employees | NAS | Allow | File sharing |
| Employees | Cameras | Allow | HTTPS, RTSP |
| Cameras | Employees | Deny | Any |
| Cameras | NAS | Allow | Recording traffic |
| Management | Network devices | Allow | SSH |

### Wireless

| SSID | VLAN | Access | Security |
|------|------|--------|----------|
| `TechPoint-Staff` | `10` | Internet, NAS, printers | WPA2/WPA3, strong password |
| `TechPoint-Guest` | `20` | Internet only | Separate password, client isolation |

## Key Design Decisions

- **Five VLANs:** each group gets its own broadcast domain, which reduces congestion.
- **Separate NAS VLAN:** one ACL allows employees and blocks guests and cameras.
- **Cameras in their own VLAN:** keeps recording traffic away from staff.
- **Two active edge routers:** the office keeps internet access if one fails.
- **Subnet sizes:** the employee `/25` holds about 90 devices (30 staff with a PC, laptop and phone) with room to spare. The camera `/27` holds 30 addresses.

## How Traffic Flows

- **Internet to employee PC:** ISP-provided router, then an edge router, then a multilayer switch, then the VLAN 10 gateway, then the PC.
- **Between VLANs:** through the SVI gateways on the switches, where ACLs apply.
- **Guest to internal resources:** the ACL drops the traffic and the guest gets no response.
- **Camera to employee PC:** denied. Cameras can only reply to connections that employees start, and can talk to the NAS.

## Scalability

| Item | Current | Future | Handled? | Change needed |
|------|---------|--------|----------|---------------|
| Employees | 15 | About 30 | Yes | None |
| IP cameras | 4 | 10 | Yes | None, the `/27` has space |
| Wi-Fi / access points | 2 | More | Yes, for now | Add an AP if Wi-Fi gets slow |
| Switch ports | About 48 | More | Yes, for now | Add a switch |
| Internet traffic | | Higher | Maybe | Upgrade the ISP plan |
| NAS storage | | More recordings | No | Bigger disks or a second NAS |

**Twist (10 cameras):** plug the 6 new cameras into VLAN 30 and add them to the ACLs. No VLAN change is needed. Check PoE on Switch 1 (about 150 W for 10 cameras at 15 W each). If it is short, add a PoE switch. Two cables could be bundled between the switches for more bandwidth.

The biggest concern is camera recordings, which fill the NAS and use the most bandwidth.

**Planned pre-deployment checks:** free switch ports, PoE budget, internet speed, NAS disk space, and VLAN and ACL support on the switches.

## Troubleshooting Approach

Scenario: a laptop reaches the internet but not the NAS, while another employee can reach the NAS.

Check in this order: laptop IP, mask and gateway, then ping the NAS by IP and by name, then cable, Wi-Fi or AP, then switch port VLAN, then ping the gateway, then switch ACLs, then the NAS (IP, VLAN 50 port, permissions), and finally compare with the working PC.

Top causes: wrong VLAN or IP, an ACL blocking the laptop, or NAS permissions.

## Limitations and Future Improvements

**Limitations**

- Only one ISP line. The edge routers protect against a router failure, not an ISP outage.
- DHCP runs only on Switch 2. If it fails, nobody gets an address.
- WLC configuration is not finished (Packet Tracer limitation, still being worked on).
- The management network can only reach Switch 1 and Switch 2 remotely.
- Cameras are simulated as PCs.

**Possible improvements**

- Add a second ISP line or a 4G/5G backup.
- Add a backup DHCP pool on Switch 1.
- Add a PoE switch, bigger disks or a second NAS if needed.

## Status and Testing

The design is documented and the topology is built in Packet Tracer. Wireless (WLC) configuration is still in progress. No test results are recorded.

## Repository Structure

Proposed structure. Configuration `.txt` files will be added separately.

```text
.
├── README.md
├── diagrams/    # network topology diagram
└── configs/     # device configuration .txt files
```

## Conclusion

This project replaces a flat, ISP-router-based network with five VLANs, two multilayer switches for routing and ACLs, two edge routers for PAT and router redundancy, and a WLC with separate staff and guest Wi-Fi. It addresses guest access, camera isolation, NAS access, device management and growth to about 30 employees and 10 cameras. It does not cover an ISP outage and has only been built in a simulator.
