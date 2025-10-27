# Lab 08 – EOS Automation

**Objective:** Demonstrate EOS API access.

### 🧩 Topology
vEOS — API Client

### ⚙️ Steps
1. Enable eAPI:
   ```
   management api http-commands
   ```
2. Test with curl:
   ```
   curl -u admin:admin http://10.0.0.1/command-api -d '{"commands":["show interfaces"]}'
   ```

### 🔍 Verification
- API returns valid data.
- Automation potential observed.

### 💡 Reflection
- How can this integrate with Python or Ansible?
- Why is automation key?

### 📝 Student Notes
> Write your observations here.