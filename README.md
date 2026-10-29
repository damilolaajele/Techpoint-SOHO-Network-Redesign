# TechPoint Solutions SOHO Network Redesign

## Project Overview

This project is a network redesign for **TechPoint Solutions**, a small business with 15 employees working from a two-floor office. It was done for the IT Village Network Clan October Design Challenge (Beginner Track).

The company currently uses one ISP-provided wireless router for almost everything. Employees, guests, printers, IP cameras and the NAS/server all share the same flat network. The business is growing and wants a network that is more reliable, secure, manageable and scalable.

The design replaces the flat network with separate VLANs and subnets. Two multilayer switches route between the VLANs and enforce access rules with ACLs. Two edge routers connect the office to the ISP-provided router. A wireless LAN controller manages the access points. The network was built and documented in Cisco Packet Tracer.

**Author:** Damilola Ajele

## Project Objectives

- Separate employees, guests, IP cameras, network management and the NAS/server into their own networks.
- Give guests internet access only. Guests must not reach employee devices, the NAS or the cameras.
- Isolate the IP cameras from employee devices so recording traffic does not slow down staff.
- Restrict NAS access to employees (and let the cameras reach it to save recordings).
- Set up a way to manage network devices remotely and securely.
- Keep internet access working if one edge router fails.
- Size the IP addressing so the company can grow from 15 to about 30 employees, and from 4 to 10 cameras, without redesigning the network.

## Requirements and Scope

### Problems with the Current Network

| # | Problem | Why It Matters | Priority |
|---|---------|----------------|----------|
| 1 | Network is slow when IP cameras are connected | Employees should be able to use internal resources and the internet without issue | Medium |
| 2 | Employees and guests share one Wi-Fi password | Guests and employees should not share a password, for security | High |
| 3 | NAS is accessible to everyone, including guests | The NAS is an internal company resource | High |
| 4 | No network management strategy | Network devices cannot be easily managed remotely | Medium |
| 5 | The whole office loses internet access when the connection goes down | It can stop business operations | High |
| 6 | Congested network | Hurts performance | High |
| 7 | Slow performance | Hurts business operations | High |

### What the Redesigned Network Must Achieve

| Requirement | How the Design Addresses It | Where It Appears |
|-------------|-----------------------------|------------------|
| Segmented network | Split the flat network into separate subnets | VLANs 10, 20, 30, 50 and 99 |
| Reliable internet access | Redundant edge routers linking to the ISP-provided router, so there is no single router to fail | 2 multilayer switches and 2 edge routers |
| Wi-Fi isolation between guests and employees | Separate SSIDs mapped to separate VLANs, with client isolation on the guest network | Wireless LAN controller and access points |
| Guests kept away from internal resources, including the NAS | Guests only get internet access | ACL policy |

### Scope

The design covers:

- Network topology and device roles.
- VLAN and IPv4 addressing plan.
- DHCP, inter-VLAN routing, NAT/PAT and ACL-based access control.
- Wireless design (SSIDs, VLAN mapping, guest isolation).
- Growth to about 30 employees and 10 cameras.
- A troubleshooting approach for a laptop that cannot reach the NAS.

Limits of the design:

- There is only one ISP line. The design does not protect against an ISP outage.
- The project was built in Cisco Packet Tracer. It is a simulation, not a physical deployment.

## Network Environment

TechPoint Solutions is a small business with a two-floor office.

| Item | Current | Future (about two years) |
|------|---------|--------------------------|
| Employees | 15 | About 30 |
| Wired desktop computers | 8 | |
| Laptops (Wi-Fi) | 7 | |
| Network printers | 2 | |
| IP cameras | 4 | 10 (6 more cameras were added in the challenge twist) |
| NAS / server | 1 | |
| Wireless access points | 2 | More if Wi-Fi gets slow |
| ISP connections | 1 | |

Guests also visit the office and use Wi-Fi.

## Hardware and Software Requirements

### Hardware

All devices in this project are simulated in Packet Tracer. Device models are not specified in the source documentation.

| Device or Component | Quantity | Purpose |
|---------------------|----------|---------|
| ISP connection | 1 | Provides internet access |
| ISP-provided router | 1 | Connects the office to the ISP and holds the default route toward the ISP |
| Edge routers | 2 | Route traffic between the internal network and the ISP-provided router. Both perform PAT |
| Multilayer switches (Switch 1 and Switch 2) | 2 | Provide the VLANs, act as default gateways (SVIs), route between VLANs and enforce ACLs. Switch 2 also runs DHCP |
| Wireless access points | 2 | Provide Wi-Fi |
| Wireless LAN controller (WLC) | 1 | Manages the access points and maps the SSIDs to VLANs |
| Employee PCs | 8 | Wired employee devices (VLAN 10) |
| Employee laptops | 7 | Wireless employee devices (VLAN 10) |
| Printers | 2 | Shared printers (VLAN 10) |
| IP cameras | 4 (10 after the twist) | Surveillance cameras (VLAN 30). In Packet Tracer they are represented by PCs |
| NAS / server | 1 | Central storage for company data (VLAN 50) |

### Software and Tools

| Software or Tool | Purpose |
|------------------|---------|
| Cisco Packet Tracer 9.0.1 | Used to build and simulate the network |
| WLC GUI | Used to map the WLANs to VLANs and manage the access points |

## Network Architecture and Design

### Overall Structure

The network has no separate core, distribution and access layers. Two multilayer switches do the switching and routing for the whole office. Two edge routers sit between the switches and the ISP-provided router.

- **Internet edge:** One ISP connection goes to one ISP-provided router. That router connects to both edge routers.
- **Edge routers:** Routes between the internal network and the internet. Routes are advertised using OSPF, and point-to-point links are configured. Both edge routers are active and both perform PAT.
- **Multilayer switches:** Both switches are active. Their SVIs are the default gateways for the VLANs, so they also route traffic between VLANs. ACLs are applied here. DHCP runs on Switch 2 only.
- **Endpoints:** Wired devices plug into the switches. The IP cameras connect to Switch 1. The NAS connects to Switch 2.
- **Wireless:** The access points connect to the switches by Ethernet. The WLC manages them from one place, although they can also be managed individually.

### Redundancy

The two edge routers protect against a router failure. Both are active and both have internet access.

This does not protect against an ISP outage, because there is only one ISP line. A second ISP line or a 4G/5G backup is listed as a future step.

### Segmentation

| VLAN | Name | Why It Exists |
|------|------|---------------|
| 10 | Employees | Staff devices, wired and wireless. Access to the internet, the NAS and the cameras |
| 20 | Guests | Guest Wi-Fi. Internet only |
| 30 | IP cameras | Keeps camera traffic away from staff devices |
| 50 | NAS/Server | Gives the NAS its own VLAN so one ACL can allow employees and block guests and cameras |
| 99 | Management | Network device management |

Separating the networks puts each group of devices in its own broadcast domain. This reduces congestion and slow performance on the old flat network.

## Network Topology Diagram

<!-- Add network topology diagram here -->

The topology was built in Cisco Packet Tracer. It shows:

- The internet cloud connected to the ISP-provided router.
- Edge Router 1 and Edge Router 2, both connected to the ISP-provided router.
- Switch 1 connected to Edge Router 1, and Switch 2 connected to Edge Router 2, with a link between the two switches.
- The IP cameras (VLAN 30) connected to Switch 1.
- The NAS (VLAN 50) connected to Switch 2.
- Employee PCs, laptops and printers (VLAN 10).
- Management PCs (VLAN 99).
- The wireless LAN controller and two access points, with guest smartphones and laptops (VLAN 20) on the wireless side.

The IP cameras are shown as PCs because of a Packet Tracer limitation.

## IP Addressing and VLAN Design

### VLANs

| VLAN ID | VLAN Name | Purpose | Associated Subnet |
|---------|-----------|---------|-------------------|
| `10` | Employees | Employee wired and wireless connections | `10.0.1.0/25` |
| `20` | Guests | Guest wireless connections | `10.0.4.0/25` |
| `30` | IP cameras | IP camera isolation | `10.0.3.0/27` |
| `50` | NAS/Server | NAS server | `10.0.5.0/27` |
| `99` | Management | Managing network devices | `10.0.2.0/27` |

### IPv4 Addressing Plan

| VLAN | Purpose | Network Address | Prefix | Usable Host Range | Default Gateway | DHCP Range | Reserved / Static |
|------|---------|-----------------|--------|-------------------|-----------------|------------|-------------------|
| `10` | Employees | `10.0.1.0` | `/25` | `10.0.1.1 - 10.0.1.126` | `10.0.1.126` | `10.0.1.11 - 10.0.1.125` | `10.0.1.1 - 10.0.1.10` |
| `20` | Guests | `10.0.4.0` | `/25` | `10.0.4.1 - 10.0.4.126` | `10.0.4.126` | `10.0.4.11 - 10.0.4.125` | `10.0.4.1 - 10.0.4.10` |
| `30` | IP cameras | `10.0.3.0` | `/27` | `10.0.3.1 - 10.0.3.30` | `10.0.3.30` | `10.0.3.11 - 10.0.3.29` | `10.0.3.1 - 10.0.3.10` |
| `50` | NAS/Server | `10.0.5.0` | `/27` | `10.0.5.1 - 10.0.5.30` | `10.0.5.30` | `10.0.5.11 - 10.0.5.29` | `10.0.5.1 - 10.0.5.10` |
| `99` | Management | `10.0.2.0` | `/27` | `10.0.2.1 - 10.0.2.30` | `10.0.2.30` | `10.0.2.11 - 10.0.2.29` | `10.0.2.1 - 10.0.2.10` |

### Other Networks Listed in the Design

These /30 networks are listed against the ISP connection and the edge routers. The documentation does not say which link each one belongs to.

| Listed With | Networks |
|-------------|----------|
| ISP connection | `203.0.113.0/30`, `203.0.113.4/30` |
| Edge routers | `192.168.1.0/30`, `192.168.1.4/30` |

### Subnet Sizing

- **Employees (/25):** 126 usable addresses. Even if 30 employees each use a PC, a laptop and a phone, that is about 90 devices. The subnet can handle this without changes.
- **Cameras (/27):** 30 usable addresses. The business may buy 10 or more cameras as it grows or if it moves to a bigger office.
- **Management and NAS/Server (/27):** Enough space for current and potential growth.

### Static Addresses and DHCP Reservations

- **Static IPs:** gateways, the NAS, printers, access points, the WLC and switches. These devices need to stay at the same address so people and rules can find them.
- **DHCP reservations:** cameras. This way the NAS always knows which camera is which.
- **Normal DHCP:** PCs and laptops.

## Network Services and Technologies

### VLANs and Inter-VLAN Routing

Each VLAN has an SVI on the multilayer switches. That SVI is the default gateway. Traffic between VLANs goes through these gateways on the two active switches, where the ACLs are applied.

### DHCP

DHCP runs on Switch 2 only, for all VLANs. The DHCP ranges are in the addressing table above.

### DNS

The documentation lists "Edge Router/ISP DNS" as the DNS source for every VLAN.

### NAT / PAT

PAT is configured on both edge routers. Both are active and both have access to the internet connection.

### Routing

The edge routers advertise routes using OSPF, and point-to-point links are configured. The ISP-provided router holds the default route toward the ISP.

### Access Control (ACLs)

ACLs on the multilayer switches (and on the routers) act as the firewall. There is no separate firewall device. The full policy is in the table below.

| Source | Destination | Action | Protocol / Service | Reason |
|--------|-------------|--------|--------------------|--------|
| Guest | Internet | Allow | HTTP, HTTPS, DNS | Guests only need the internet |
| Guest | Employee network | Deny | Any | Protect staff devices |
| Guest | NAS | Deny | Any | Company data |
| Guest | Cameras / IoT | Deny | Any | Privacy and security |
| Guest | Management | Deny | Any | Protect network devices |
| Employees | NAS | Allow | File sharing | Their work files |
| Employees | Cameras / IoT | Allow | HTTPS, RTSP | To view the cameras |
| Cameras / IoT | Employees | Deny | Any | Cameras should never start a connection to staff |
| Cameras | NAS | Allow | Recording traffic | Cameras need to save recordings |
| Management | Network devices | Allow | SSH | Only management manages devices |

### Wireless

| SSID | Purpose | VLAN | Who Can Connect | Internet | Internal Access | Security |
|------|---------|------|-----------------|----------|-----------------|----------|
| `TechPoint-Staff` | Employee Wi-Fi | `10` | Employees | Yes | Yes (NAS, printers) | WPA2/WPA3, strong password |
| `TechPoint-Guest` | Guest Wi-Fi | `20` | Guests | Yes | No | Separate password, client isolation |

The WLC bridges traffic from each WLAN to the correct VLAN. The WLAN is mapped to the VLAN in the WLC GUI.

### Secure Device Management

- Devices are managed only from VLAN 99.
- SSH is used instead of Telnet.
- Strong passwords are used.
- Unused ports are turned off.
- The WLC manages the access points from one place.

Currently, the management network can only reach Switch 1 and Switch 2 remotely.

## Design Decisions and Rationale

| Decision | Reason |
|----------|--------|
| Five VLANs instead of one flat network | Each group of devices gets its own broadcast domain. This avoids congestion and slow performance, and lets devices communicate without affecting other networks |
| A separate VLAN for the NAS (VLAN 50) | One ACL can allow employees and block guests and cameras |
| Cameras in their own VLAN | Isolates them from staff devices. Their recording traffic does not slow down staff |
| Multilayer switch SVIs as default gateways | Inter-VLAN routing and ACLs are handled on the switches |
| DHCP on Switch 2 | Documented as the DHCP location. A reason is not given in the source |
| PAT on both edge routers | Both routers are active and both have internet access, so one can fail without the office losing internet |
| Two edge routers | Protects against a router failure. Trade-off: it does not protect against an ISP outage |
| Guest SSID with client isolation and a separate password | Guests cannot talk to each other, and the guest VLAN is blocked from every internal network by ACL |
| WLC for the access points | Central management. New access points are easy to add |
| ACLs as the firewall function | No separate firewall device is used |
| /25 subnets for employees and guests, /27 for the rest | Room for growth to about 30 employees and 10 or more cameras |
| Static IPs for infrastructure, DHCP reservations for cameras | Infrastructure must stay at fixed addresses. Camera reservations keep each camera identifiable to the NAS |

## Network Operation and Traffic Flow

### Internet to an Employee PC

1. Traffic arrives from the internet at the ISP-provided router.
2. It goes to one of the company's edge routers (routes are advertised using OSPF).
3. The edge router forwards it to one of the multilayer switches. Both switches are active.
4. The VLAN 10 default gateway (on either switch) forwards it to the employee's PC in VLAN 10.

### Wireless Clients

Access points connect to a switch by Ethernet. The WLC manages them. Staff Wi-Fi traffic is mapped to VLAN 10 and guest Wi-Fi traffic is mapped to VLAN 20.

### Between VLANs

Traffic between VLANs goes through the SVI default gateways on the two multilayer switches. The ACLs decide what is allowed.

### Guest Reaching an Employee PC or the NAS

The ACL on the multilayer switch drops the traffic. The guest gets no response, and the NAS never sees the traffic.

### Camera Initiating a Connection to an Employee PC

The ACL denies it. Cameras can only reply to connections that employees start. They can talk to the NAS to save recordings.

## Implementation and Configuration Files

The device configurations are not included in this README. They will be added to the repository separately as `.txt` files, organized in folders. The configuration files should match the VLANs, addressing and ACL policy documented here.

## Testing and Verification

No test results are recorded in the project documentation. The project does not claim that any tests were completed.

### Documented Troubleshooting Approach

The workbook includes a troubleshooting scenario: an employee laptop can access the internet but cannot access the NAS, while another employee on the same network can. The documented sequence follows the path laptop, access point or switch, VLAN, gateway, ACL, then NAS.

| Step | What to Check | What It Tells You |
|------|---------------|-------------------|
| 1 | Laptop IP, mask and gateway (`ipconfig`) | A wrong or `169.254.x.x` address points to a DHCP problem |
| 2 | Ping the NAS by IP, then by name | IP works but name fails means a DNS problem |
| 3 | Cable, Wi-Fi connection or AP | Connection problem at the first hop |
| 4 | Switch port VLAN | The wrong VLAN means it can't reach the NAS |
| 5 | Ping the gateway | Failure means a problem before the switch SVI |
| 6 | ACLs on the switch | The laptop may be blocked by an ACL |
| 7 | The NAS: its IP, VLAN 50 port and any permission limits | NAS-side problem |
| 8 | Compare with the working PC | The difference shows the cause |

The three most likely causes are:

| Possible Cause | Evidence | How to Verify |
|----------------|----------|---------------|
| Wrong VLAN or IP | The laptop is on the guest VLAN, or has a wrong or old IP | Check with `ipconfig` and the switch port |
| ACL blocking | The laptop's IP is blocked or outside the allowed range | Check with `show access-lists` |
| NAS permissions | The NAS doesn't allow that user or device | Check in the NAS settings |

### Planned Pre-Deployment Checks

These checks are listed in the workbook before deployment. No results are recorded for them.

- Free switch ports.
- PoE power budget.
- Internet speed.
- NAS disk space.
- Whether the switches support the number of VLANs and ACLs.

## Scalability

| Growth Item | Current | Future | Handled? | What Would Change |
|-------------|---------|--------|----------|-------------------|
| Employees | 15 | About 30 | Yes | Nothing. The `/25` has space |
| IP cameras | 4 | 10 | Yes | The `/27` has 30 addresses |
| Wireless clients | 7 | More | Yes, for now | Add an AP if Wi-Fi gets slow |
| Switch ports | About 48 | More endpoints | Yes, for now | Add a switch if ports run out |
| Access points | 2 | Possibly more | Yes | The WLC manages new ones |
| Internet traffic | | Higher | Maybe | Upgrade the ISP plan |
| NAS / storage | | More camera recordings | No | Bigger disks or a second NAS |

The biggest scalability concern is camera recordings. They fill up the NAS and use the most bandwidth.

### Twist: 6 More Cameras (10 Total)

- **Design change:** Very little. The 6 new cameras go into VLAN 30 and are added to the ACLs. The VLAN design stays the same.
- **IP addresses:** 10 cameras fit easily in the `/27`, with room for 20 more.
- **Switch ports and PoE:** 6 more free PoE ports are needed. The PoE power budget on Switch 1 should be checked (10 cameras at about 15 W each is about 150 W). If it is short, a PoE switch is added.
- **Traffic:** More recordings use more uplink bandwidth and NAS storage. The cameras are on Switch 1 and the NAS is on Switch 2, so two cables could be bundled between the switches for more speed.

## Limitations and Future Improvements

### Limitations

- **Single ISP line.** The two edge routers protect against a router failure, not an ISP outage.
- **DHCP on Switch 2 only.** If it fails, nobody gets an address.
- **Wireless configuration is not finished.** The workbook notes a Packet Tracer limitation with the WLC configuration and GUI access, and says this is still being worked on.
- **Cameras are simulated as PCs** because of a Packet Tracer limitation.
- **Management access is limited.** For now, the management network can only reach Switch 1 and Switch 2 remotely.
- **Storage.** The NAS will need more capacity as camera recordings grow.

### Possible Future Improvements

- Add a second ISP line or a 4G/5G backup, when the budget allows.
- Add a second DHCP pool on Switch 1 as a backup.
- Bundle two cables between the switches if camera traffic increases uplink load.
- Add a PoE switch if the PoE budget on Switch 1 is not enough.
- Add bigger disks or a second NAS.
- Add an access point, a switch, or a faster ISP plan if Wi-Fi, ports or internet speed become a problem.

## Project Status

The design is documented and a topology has been built in Cisco Packet Tracer. The workbook marks all design review items as complete, except that wireless (WLC) configuration is noted as still in progress. No test results are recorded.

## Repository Structure

This is a proposed structure. The final layout has not been set.

```text
.
├── README.md
├── diagrams/    # network topology diagram
└── configs/     # device configuration .txt files
```

## Conclusion

This project replaces TechPoint Solutions' flat, ISP-router-based network with a segmented design. It uses five VLANs, two multilayer switches for routing and ACLs, two edge routers for PAT and router redundancy, and a wireless LAN controller with separate staff and guest SSIDs. The design addresses guest access, camera isolation, NAS access, device management and growth to about 30 employees and 10 cameras. It does not cover an ISP outage, and it has only been built in a simulator.
