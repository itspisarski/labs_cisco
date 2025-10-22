# Lab 02 – ARP and MAC Addressing

**Objective:** Observe ARP request and MAC resolution within a subnet.

### 🧩 Topology
PC1 — Switch — PC2

| Device | IP Address |
|---------|-------------|
| PC1 | 192.168.10.1/24 |
| PC2 | 192.168.10.2/24 |

### ⚙️ Steps
1. Connect two PCs to a switch.
2. Assign IPs to both PCs.
3. On PC1, run `arp -a` to see empty cache.
-> Expected output : No ARP Entries Found
5. Ping PC2 and re-run `arp -a`.
6. Observe ARP broadcast and reply in Simulation mode.
-> Internet Address      Physical Address      Type
   192.168.10.2          00e0.f9dd.084c        dynamic

### 🔍 Verification
- ARP table updates after ping.
- MAC addresses appear in ARP cache.

### 💡 Reflection
- What happens when ARP cache is cleared?
- Why doesn’t ARP work across routers?

### 📝 Student Notes
> Write your observations here.
