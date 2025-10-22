# Lab 05 – Cisco IOS Basics

**Objective:** Learn CLI configuration of routers.

### 🧩 Topology
Router — PC

### ⚙️ Steps
1. Access router CLI.
2. Run:
   ```
   enable
   configure terminal
   hostname R1
   interface g0/0
   ip address 192.168.1.1 255.255.255.0
   no shutdown
   ```
3. Save config with `copy run start`.

### 🔍 Verification
- Interface up and pingable.
- Configuration saved.

### 💡 Reflection
- What are IOS modes?
- Why save running config?

### 📝 Student Notes
> Write your observations here.