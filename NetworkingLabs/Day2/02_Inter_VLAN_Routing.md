# Lab 02 – Inter-VLAN Routing (Router-on-a-Stick)

**Objective:**  
Configure inter-VLAN routing using a single router interface with multiple subinterfaces. You will verify communication between VLANs and understand how routers process VLAN-tagged traffic.

---

## 🧩 Topology
```
[PC1]--[SW1]--[Router]--[SW2]--[PC2]
```
yaml
Copy code

| Device | VLAN | Subnet | IP Address |
|---------|------|---------|-------------|
| PC1 | VLAN 10 | 192.168.10.0/24 | 192.168.10.10 |
| PC2 | VLAN 20 | 192.168.20.0/24 | 192.168.20.10 |
| Router (G0/0.10) | VLAN 10 | 192.168.10.1 |
| Router (G0/0.20) | VLAN 20 | 192.168.20.1 |

---

## ⚙️ Step 1. Create VLANs on the Switch

```
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config)# vlan 20
Switch(config-vlan)# name FINANCE
```
Assign access ports:
```
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```
---

## ⚙️ Step 2. Configure the Trunk Between Switch and Router

```
Switch(config)# interface fa0/24
Switch(config-if)# switchport mode trunk
```
---

## ⚙️ Step 3. Configure Router Subinterfaces (ROAS)
```
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface g0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0

Router(config)# interface g0/0
Router(config-if)# no shutdown
```
---

## ⚙️ Step 4. Configure PCs

| PC | IP Address | Subnet Mask | Gateway |
|----|-------------|-------------|----------|
| PC1 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

---

## ⚙️ Step 5. Verify Configuration

### Router
```
show ip interface brief
```
### Switch
```
show interfaces trunk
```
✅ Ensure VLANs are trunked correctly and the router’s subinterfaces are up.

---

## ⚙️ Step 6. Test Inter-VLAN Communication

- From **PC1**, ping `192.168.20.10`.  
✅ Successful ping means inter-VLAN routing works.

---

## 💡 Reflection

- Why is `encapsulation dot1Q` required?  
- What role does the router play in inter-VLAN communication?  
- How many physical interfaces are used for multiple VLANs?

---

## 🧭 Summary

| Concept | Description |
|----------|--------------|
| VLAN | Logical network segments |
| Router-on-a-Stick | One interface routes multiple VLANs |
| Encapsulation | VLAN tagging using IEEE 802.1Q |
| Verification | Use `show` and `ping` commands |

---
