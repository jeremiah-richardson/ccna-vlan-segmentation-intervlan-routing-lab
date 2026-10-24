# CCNA Lab: VLAN Segmentation and Inter-VLAN Routing Lab

Configured three VLANs based on dept. using manual IPv4 subnetting, switchport access assignments, legacy inter-VLAN routing, and packet-level verification.

## Project Overview

This project demonstrates the configuration and verification of three departmental VLANs using Cisco Packet Tracer.

The objective was to segment a network into separate Layer 2 broadcast domains, implement manual IPv4 subnetting, configure router interfaces for communication between VLANs, and analyze network traffic using Simulation Mode.

The network represents three departments: Engineering, Human Resources (HR), and Sales.

## Network Topology

The lab consists of:

- 1 Cisco router (R1)
- 1 Cisco Layer 2 switch (SW1)
- 6 end-user PCs
- 3 VLANs, each assigned a separate IPv4 subnet
- 3 physical router interfaces for inter-VLAN routing

<img width="1101" height="642" alt="Lab Topology" src="https://github.com/user-attachments/assets/6bd3f677-56d0-4a0b-b463-b768b13d4b24" />

## IPv4 Subnetting and Addressing

I manually calculated the subnet ranges and configured the router interfaces and PC network settings to match the addressing scheme.

| VLAN | Department | Network | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| 10 | Engineering | 10.0.0.0/26 | 255.255.255.192 | 10.0.0.62 |
| 20 | HR | 10.0.0.64/26 | 255.255.255.192 | 10.0.0.126 |
| 30 | Sales | 10.0.0.128/26 | 255.255.255.192 | 10.0.0.190 |

Each `/26` subnet supports 62 usable host addresses.

## Layer 2: VLAN Configuration

Three VLANs were created on SW1 to logically separate devices by department.

**VLAN 10 — Engineering**

```text
interface range fa0/1, fa0/4, fa0/5
 switchport mode access
 switchport access vlan 10
```

**VLAN 20 — HR**

```text
interface range fa0/2, fa0/6, fa0/7
 switchport mode access
 switchport access vlan 20
```

**VLAN 30 — Sales**

```text
interface range fa0/3, fa0/8, fa0/9
 switchport mode access
 switchport access vlan 30
```

The VLANs were named ENGINEERING, HR, and SALES to improve identification and organization.

Configuration was verified using:

```text
show vlan brief
```

This confirmed the VLAN assignments and associated switch ports.

## Layer 3: Inter-VLAN Routing

Legacy inter-VLAN routing was implemented using three physical router interfaces, each connected to an access port in its corresponding VLAN.

**R1 Interface Configuration**

<img width="928" height="772" alt="VLANs and VLAN names configured" src="https://github.com/user-attachments/assets/bef672ab-45ac-4001-93bc-ced5248e6362" />

| Interface | IP Address | Department |
|---|---|---|
| G0/0 | 10.0.0.62/26 | Engineering |
| G0/1 | 10.0.0.126/26 | HR |
| G0/2 | 10.0.0.190/26 | Sales |

The interfaces were configured with appropriate subnet masks and descriptive labels.

I verified the operational status using:

```text
show ip interface brief
```

All three GigabitEthernet interfaces displayed an up/up status.

This design allows R1 to route traffic between the three VLAN subnets without using router subinterfaces or an 802.1Q trunk.

## Connectivity Verification

I initiated ICMP ping tests from PC1 in the Engineering VLAN toward devices in the HR and Sales VLANs.

These tests were designed to verify that devices in different IP subnets could communicate through R1.

ARP supports this communication by resolving local next-hop IPv4 addresses to MAC addresses. When communicating with a remote subnet, a PC forwards its traffic to the MAC address of its default gateway.

## Packet Analysis and VLAN Segmentation

Cisco Packet Tracer Simulation Mode was used to examine traffic forwarding between devices and observe the role of the router in inter-VLAN communication.

### Broadcast Traffic Analysis and VLAN Isolation

<img width="1917" height="683" alt="Broadcast address ping sending to only VLAN 10 hosts showcasing VLAN is functional F" src="https://github.com/user-attachments/assets/dd68169f-4cd8-4850-969c-a840685db66f" />


To examine VLAN broadcast behavior, I used Cisco Packet Tracer Simulation Mode to generate ICMP traffic from PC1 in VLAN 10 (Engineering).

I inspected the inbound PDU details at PC2, which belongs to the same VLAN.

**Captured Packet Information**

| Field | Value |
|---|---|
| Source IPv4 | 10.0.0.1 |
| Destination IPv4 | 255.255.255.255 |
| Destination MAC | FFFF.FFFF.FFFF |
| IPv4 Protocol | 0x01 (ICMP) |
| ICMP Type | 0x08 (Echo Request) |
| Initial TTL | 128 |

The destination IPv4 address `255.255.255.255` identifies a limited broadcast, while `FFFF.FFFF.FFFF` identifies an Ethernet broadcast frame.

The Simulation Mode event list showed the packet being processed by devices associated with VLAN 10, including PC2, without visible delivery to hosts in VLAN 20 or VLAN 30.

This behavior is consistent with VLAN segmentation, where Layer 2 broadcast traffic remains within its assigned VLAN.

**802.1Q Frame Inspection**

The captured Ethernet frame did not display the IEEE 802.1Q Tag Protocol Identifier `0x8100`.

This indicates that the inspected frame was untagged, as expected when traffic is delivered to an end device through a switch access port.

**Key Takeaway**

This exercise reinforced my understanding of Ethernet broadcast addressing, VLAN broadcast domains, access port behavior, and the difference between tagged and untagged Ethernet frames.

Using Simulation Mode allowed me to inspect packet headers and observe forwarding behavior rather than relying solely on connectivity tests.
### Ethernet Frame Tagging

IEEE 802.1Q uses the Tag Protocol Identifier (TPID) `0x8100` to identify VLAN-tagged Ethernet frames.

On access links connecting end devices, Ethernet frames are normally untagged.

The absence of an 802.1Q tag in an inspected Ethernet frame is consistent with untagged access-port communication.

### Broadcast Domain Isolation

Each VLAN represents a separate Layer 2 broadcast domain.

For VLAN 10 (`10.0.0.0/26`), the directed broadcast address is `10.0.0.63`.

A Layer 2 broadcast generated within VLAN 10 should be forwarded only through ports belonging to VLAN 10, rather than being flooded into VLANs 20 or 30.

The Simulation Mode screenshots illustrate packet forwarding through the configured topology. Additional packet-header inspection can confirm the broadcast destination MAC address and VLAN containment.

## Verification Commands

```text
show vlan brief
show ip interface brief
show running-config
ping
```

## Skills Demonstrated

- Manual IPv4 subnetting
- VLAN creation and naming
- Switch access port configuration
- VLAN segmentation and broadcast domains
- Legacy inter-VLAN routing
- Cisco IOS interface configuration
- ARP and ICMP fundamentals
- Ethernet frame analysis
- Cisco Packet Tracer Simulation Mode
- Network configuration verification

## Lessons Learned

This lab reinforced the relationship between Layer 2 segmentation and Layer 3 routing.

VLANs logically separate devices into distinct broadcast domains, while routing enables communication between those networks when permitted.

I also gained additional experience configuring Cisco switch ports, assigning IP addresses according to a subnetting plan, interpreting interface status, and examining packet forwarding through Simulation Mode.

The project strengthened my understanding of how Ethernet switching and IP routing work together in a segmented network.

## Tools

- Cisco Packet Tracer
- Cisco IOS CLI
- IPv4, Ethernet, VLANs, ARP, ICMP
- IEEE 802.1Q fundamentals

---

*Part of my hands-on CCNA networking and cybersecurity portfolio.*
