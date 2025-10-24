# Lab 09 – IPv6 Configuration and Testing

**Objective:** Configure IPv6 addresses and test communication.

### 🧩 Topology
Router1 — Router2 — PC

### ⚙️ Steps
1. Enable IPv6:
   ```
   ipv6 unicast-routing
   ```
2. Assign:
   ```
   interface g0/0
   ipv6 address 2001:db8:1::1/64
   ```
3. Ping between IPv6 hosts.

### 🔍 Verification
- IPv6 ping success.
- EUI-64 auto-generation observed.

### 💡 Reflection
- Difference between global and link-local?
- How is EUI-64 derived?

### 📝 Student Notes
> Write your observations here.