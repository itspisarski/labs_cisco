# Lab 01 – OSI and Encapsulation Visualization

**Objective:** Understand how data is encapsulated through OSI and TCP/IP layers.

### 🧩 Topology
PC1 — Switch — PC2

| Device | Interface | IP Address |
|---------|------------|-------------|
| PC1 | NIC | 192.168.1.1/24 |
| PC2 | NIC | 192.168.1.2/24 |

### ⚙️ Steps
1. Open Cisco Packet Tracer and add two PCs and one switch.
2. Connect PCs using Copper Straight-Through cables.
3. Assign IPs to both PCs.
4. Ping from PC1 to PC2.
5. Switch to **Simulation Mode** and observe ICMP encapsulation.

### 🔍 Verification
- Ping succeeds between PCs.
- The PDU shows Ethernet, IP, and ICMP headers.

### 💡 Reflection
- How does encapsulation differ between layers?
- Why does Ethernet use MACs while IP uses logical addresses?

### 📝 Student Notes
> Write your observations here.