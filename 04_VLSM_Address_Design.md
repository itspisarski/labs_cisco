# Lab 04 – VLSM Address Design

**Objective:** Design variable subnet sizes efficiently.

### 🧩 Topology
Router — Switch — PCs

### ⚙️ Steps
1. Given `192.168.100.0/24`, allocate `/26`, `/27`, and `/28` networks for 60, 30, and 10 hosts.
2. Configure router interfaces.
3. Verify using `show ip route` and `ping`.

### 🔍 Verification
- Ping succeeds between subnets.
- Routing table lists variable prefixes.

### 💡 Reflection
- How does VLSM conserve IPs?
- Why allocate largest subnet first?

### 📝 Student Notes
> Write your observations here.