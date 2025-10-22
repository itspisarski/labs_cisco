# Lab 07 – Arista EOS Basics

**Objective:** Explore Arista vEOS CLI and configuration.

### 🧩 Topology
vEOS — PC

### ⚙️ Steps
1. Connect to vEOS console.
2. Run:
   ```
   enable
   configure terminal
   interface Ethernet1
   ip address 10.0.0.1/24
   no shutdown
   ```
3. Verify with `show interfaces status`.

### 🔍 Verification
- Interface active and IP assigned.

### 💡 Reflection
- How does EOS differ from IOS?
- What is Linux’s role in EOS?

### 📝 Student Notes
> Write your observations here.