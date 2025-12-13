# Deep In Net – Network Fundamentals

## Project Overview
This project simulates essential network architectures using **Cisco Packet Tracer**. It covers the **OSI Model Layers 1 through 7**, focusing on:

- Physical cabling  
- Switching and routing logic  
- Subnetting (VLSM)  
- Application layer protocol configuration (TCP/IP, DHCP, DNS, HTTP, FTP)

---

## Exercise 1: Peer-to-Peer Communication

### Objective
Establish direct communication between pairs of end devices (PCs) **without intermediary devices** such as switches or hubs.

### 1. Network Topology
The network consists of three isolated pairs of computers. Each pair represents a distinct LAN with specific subnetting requirements:

- **Pair A:** PC0 ⟷ PC1 (Standard `/24` Network)  
- **Pair B:** PC2 ⟷ PC3 (User-Defined Network)  
- **Pair C:** PC4 ⟷ PC5 (Restricted `/29` Subnet)

### 2. Implementation Steps

#### Physical Layer (Layer 1)
- Deployed **6 Generic PCs**
- **Cabling:** Copper Crossover Cables  
- **Reasoning:** PCs are *like devices* (both transmit on pins 1 & 2 and receive on pins 3 & 6), therefore a crossover cable is required to swap TX/RX signals.

#### Network Layer (Layer 3)
- Configured **static IP addresses**
- Calculated the subnet mask for the `/29` requirement on Pair C

### 3. Addressing Table

| Device | Interface        | IP Address        | Subnet Mask         | CIDR | Note |
|------|------------------|-------------------|---------------------|------|------|
| PC0  | FastEthernet0    | 192.168.1.3       | 255.255.255.0       | /24  | Standard Class C |
| PC1  | FastEthernet0    | 192.168.1.4       | 255.255.255.0       | /24  |      |
| PC4  | FastEthernet0    | 192.168.13.254    | 255.255.255.248     | /29  | Subnet calculation below |
| PC5  | FastEthernet0    | 192.168.13.253    | 255.255.255.248     | /29  |      |

#### Calculation for `/29` Subnet
- `/29` → 29 bits set to `1`, **3 bits for hosts**
- **Block Size:** 2³ = 8  
- **Decimal Mask:** `255.255.255.248`

### 4. Knowledge Concepts
- **RJ-45:** Standard 8-pin Ethernet connector (twisted-pair cables)
- **Straight-Through vs. Crossover**
  - **Straight-Through:** Same wiring order on both ends; used for *different devices* (PC ⟶ Switch)
  - **Crossover:** TX/RX pins crossed; used for *same devices* (PC ⟶ PC)

---

## Exercise 2: Hub vs. Switch

### Objective
Differentiate between **Layer 1 (Hub)** and **Layer 2 (Switch)** behavior.

### 1. Network Topology
Two separate star topologies:

- **Cluster A:** 3 PCs connected to a Cisco 2960 Switch  
- **Cluster B:** 3 PCs connected to a Generic Hub

### 2. Implementation Steps
- **Cabling:** Copper Straight-Through  
- **Reasoning:** PCs and Switches/Hubs use different transmit/receive pins (MDI vs MDI-X)
- **Simulation Mode Observations:**
  - **Hub:** Floods packets to all ports
  - **Switch:** Sends packets only to the destination port (Unicast)

### 3. Addressing Table

| Device Group | Connected To | IP Network     | Subnet Mask |
|-------------|--------------|----------------|-------------|
| PC0–PC2     | Switch       | 192.168.1.0    | 255.255.255.0 |
| PC3–PC5     | Hub          | 192.168.2.0    | 255.255.255.0 |

### 4. Knowledge Concepts
- **Hub (Layer 1):**  
  A physical-layer device that repeats signals to all ports, creating a large collision domain and security risks.
- **Switch (Layer 2):**  
  Uses a **MAC Address Table** to forward traffic only to the intended destination.

---

## Exercise 3: Network Services

### Objective
Configure **Application Layer services** (DHCP, DNS, FTP, HTTPS) using a Client–Server architecture.

### 1. Network Topology
A server farm hosting DHCP, DNS, HTTPS, and FTP services connected to clients via a switch.

### 2. Implementation Steps
- **DHCP:**  
  - Pool assigns IPs starting from `.100`
  - Gateway: `.1`
  - DNS: `.10`
- **DNS:**  
  - `deep-in-net.com` (CNAME) ⟶ `deep-in-net.local` (A Record) ⟶ `192.168.1.99`
- **HTTPS:**  
  - Enabled HTTPS (Port 443)
  - Disabled HTTP (Port 80)
  - Edited `index.html`
- **FTP:**  
  - User: `deepinnet`
  - Permissions: **RWDNL** (Read, Write, Delete, Rename, List)

### 3. Addressing Table

| Device   | Role  | IP Address       | Gateway       | DNS |
|--------|-------|------------------|---------------|-----|
| Web-Svr | HTTPS | 192.168.1.99     | 192.168.1.1   | 192.168.1.10 |
| DNS-Svr | DNS   | 192.168.1.10     | 192.168.1.1   | 127.0.0.1 |
| DHCP-Svr| DHCP  | 192.168.1.11     | 192.168.1.1   | 192.168.1.10 |
| PC0     | Client| DHCP Assigned    | 192.168.1.1   | 192.168.1.10 |

### 4. Knowledge Concepts
- **Server:** Dedicated system providing services to clients
- **DHCP (DORA):** Discover, Offer, Request, Acknowledge
- **DNS:** Translates domain names into IP addresses
- **TCP vs UDP**
  - **TCP:** Reliable, connection-oriented (HTTP, FTP)
  - **UDP:** Fast, connectionless (DNS, DHCP)
- **Ports:**  
  - HTTP: 80  
  - HTTPS: 443  
  - FTP: 20/21  
  - DNS: 53  

---

## Exercise 4: Routing Fundamentals

### Objective
Connect different networks using a **Layer 3 Router**.

### 1. Network Topology
Two PCs connected via a router simulating a WAN link.

### 2. Implementation Steps
- **Subnetting:** Tight `/30` networks
- **Constraint:** `/30` allows only **2 usable IPs**
- **Router Configuration:**  
  - Interfaces `Gig0/0` and `Gig0/1` act as gateways
- **PC Configuration:**  
  - Default gateways manually set

### 3. Addressing Table

| Device | Interface | IP Address     | Subnet Mask       | Note |
|------|-----------|----------------|-------------------|------|
| Router | Left Int  | 192.168.1.1    | 255.255.255.252   | Gateway for Left LAN |
| Router | Right Int | 192.168.2.1    | 255.255.255.252   | Gateway for Right LAN |
| PC0 | NIC | 192.168.1.2 | 255.255.255.252 | /30 Network |
| PC1 | NIC | 192.168.2.2 | 255.255.255.252 | /30 Network |

### 4. Knowledge Concepts
- **Router:** Layer 3 device that forwards packets between networks
- **Default Gateway:** The router’s IP acting as the exit point from a subnet

---

## Exercise 5: Advanced Subnetting (VLSM)

### Objective
Implement **Variable Length Subnet Masking (VLSM)** to support networks of different sizes.

### 1. Network Topology
A central router connects two switches with different subnet requirements.

### 2. Implementation Steps

#### Left Subnet – `192.168.1.0/29`
- Requires 6 IPs (5 PCs + 1 Router)
- Mask: `255.255.255.248`
- No address waste

#### Right Subnet – `192.168.1.192/27`
- Allows room for expansion
- Mask: `255.255.255.224`

### 3. Addressing Table (Sample)

| Network | IP Range | Mask | Gateway |
|-------|----------|------|---------|
| Left (/29) | 192.168.1.1 – 192.168.1.6 | 255.255.255.248 | 192.168.1.1 |
| Right (/27) | 192.168.1.193 – 192.168.1.222 | 255.255.255.224 | 192.168.1.193 |

### 4. Verification
- **Local Test:** Ping between PCs on the same switch (Layer 2)
- **Remote Test:** Ping between Left and Right subnets (Layer 3)
- **Note:** First ping may fail due to ARP learning

### 5. Packet Tracer Verification Checklist
- Open the `.pkt` file
- Open **PC → Command Prompt**
- `ping <destination IP>`
- `nslookup <domain name>` (Exercise 3)
- `ftp <server IP>` (Exercise 3)
