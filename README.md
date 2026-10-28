# Multi-Branch Enterprise Network Design & Security Implementation (CCNA Capstone Project)

## 📌 Project Overview
This project demonstrates the design, configuration, and security hardening of a robust, scalable enterprise network infrastructure spanning **five geographic branches** in Syria: **Damascus (Headquarters), Homs, Latakia, Aleppo, and Hama**. Each branch is engineered to support up to 60 hosts dynamically, ensuring cross-branch communication, centralized server access, and high-level cybersecurity compliance.

The entire topology is built and validated using **Cisco Packet Tracer**.

---

## 🛠️ Network Architecture & Design

### 1. IP Subnetting Scheme (VLSM)
A custom Subnetting strategy was applied to maximize efficiency and accommodate future expansion (up to 60 hosts per subnet using `/26` masks):
* **Damascus Branch Subnets Example:** `192.168.1.0/26`, `192.168.1.64/26`, `192.168.1.128/26`.
* **Other Branches:** Scaled progressively up to the `192.168.5.0/26` network and its respective subnets.

### 2. Device Roles & Layer Specifications
* **Routers:** Operating at Layer 3 (Network) to enable inter-VLAN routing, WAN connectivity, and dynamic path selection.
* **Switches:** Operating at Layer 2 (Data Link) to facilitate intra-LAN connectivity within the same Network ID (NID).
* **End Devices:** PC/Laptops representing organizational departments.

---

## ⚙️ Core Switching & Routing Implementations

### Layer 2 Infrastructure
* **VTP (VLAN Trunking Protocol):** Configured in a Server/Client architecture (Server Domain: `ammar`, Password: `12345`). VLANs are provisioned centrally on the Core Server Switch and automatically propagated to Client switches, eliminating repetitive configurations.
* **EtherChannel:** Implemented link aggregation between switches using PagP/LACP to bundle active ports into a single logical `Port-channel 1` interface, drastically increasing inter-switch bandwidth.
* **STP & PortFast:** Spanning Tree Protocol is optimized to break Layer 2 loops and prevent broadcast storms. **PortFast** is explicitly enabled on access ports tied to PCs/Laptops to bypass listening/learning states, bringing ports to a forwarding state instantly.
* **Inter-VLAN Routing:** Achieved using Router-on-a-Stick config, assigning the first usable IP of each subnet to its respective sub-interface (e.g., `.10`, `.20`, `.30` tags).

### Layer 3 Routing
* **EIGRP & RIPv2:** Dynamic routing protocols configured across the topology. Metric weights and network statements are precisely bound to wildcard masks matching the subnets.
* **Static NAT (Network Address Translation):** Configured on the Damascus HQ edge router. A public IP mapping is designated for the central web/TFTP server, allowing branches to hit the server natively.
* **Loopback Interface:** Configured on the HQ router to advertise the server's public IP inside the dynamic routing protocol.

---

## 🔒 Advanced Infrastructure Security Hardening

To establish a defense-in-depth framework, multiple Layer 2 and Layer 3 security features were deployed:

1. **Port Security:** Configured on all access switchports. Allowed MAC address limits are set to `maximum 1` with `mac-address sticky` enabled. Any unauthorized device change triggers a `violation shutdown` state, instantly disabling the interface.
2. **DHCP Snooping:** Mitigates rogue DHCP server attacks. All internal infrastructure uplink ports are explicitly set as `trust`, while user-facing access ports are untrusted and rate-limited.
3. **EIGRP Message Authentication:** Configured using a synchronized `key chain` setup (`key-string 12345`) utilizing MD5 hashing to prevent malicious route-injection or spoofing attacks.
4. **PPP CHAP (Challenge Handshake Authentication Protocol):** Configured over Serial WAN links to enforce cryptographic authentication during link negotiation between branch routers.
5. **IPsec VPN Tunnel:** Established a secure encrypted tunnel between the Homs and Hama branches via ISAKMP policies (AES-128 encryption, MD5 hashing, Diffie-Hellman Group 2) to safely bridge remote LAN traffic over an untrusted WAN public zone.
6. **Device Access Hardening (AAA & VTY):** 
   * **AAA (Authentication, Authorization, and Accounting):** Enabled to manage administrative privileges natively.
   * **VTY Security:** Protected telnet/SSH terminal lines with an encrypted local login schema (`service password-encryption` and `enable secret`).

---

## 🛠️ Real-world Engineering Troubleshooting Notes

* **Access-List (ACL) Side Effects:** During initial implementation, standard/extended ACLs accidentally blocked DHCP Broadcast requests (UDP 67/68) and administrative Telnet traffic (TCP 23). This was resolved by re-engineering the ACLs to explicitly `permit udp any any eq 67 68` and `permit tcp any any eq 23` before the implicit deny block.
* **TFTP Backup Loopbacks:** Implementing remote centralized configuration backups via TFTP over the public IP caused a localized loopback routing conflict on the Damascus HQ edge router due to direct-connected interface precedence in Packet Tracer. The architecture was successfully optimized to push backups smoothly over designated private paths or DNS structures.

---

## 📂 Deliverables in this Repository
* `*.pkt` - Fully interactive Cisco Packet Tracer Network Topology file.
* `*.vsdx` - High-level architectural network schematic diagram created using Microsoft Visio.
* `README.md` - Complete technical overview.
