# Lab 01 – VLANs and Trunking

**Objective:**  
In this lab, you will configure VLANs, assign access ports, and enable trunking between switches in Cisco Packet Tracer. You will also verify VLAN communication and understand how VLANs isolate broadcast domains.

---

## 🧩 Topology
```
[PC1]----[SW1]----[SW2]----[PC2]
```

| Device | Interface | VLAN | IP Address |
|---------|------------|------|-------------|
| PC1 | NIC | VLAN 10 | 192.168.10.10/24 |
| PC2 | NIC | VLAN 20 | 192.168.20.10/24 |

---

## ⚙️ Step 1. Build the Topology in Packet Tracer

1. Add **2 switches** (2960) and **2 PCs**.  
2. Connect PC1 and PC2 to their respective switches using **Copper Straight-Through** cables.  
3. Connect SW1 to SW2 using **FastEthernet0/24**.  
4. Ensure all interfaces show **green links** before proceeding.

---

## ⚙️ Step 2. Create VLANs on Both Switches
```
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config)# vlan 20
Switch(config-vlan)# name FINANCE
```


> 💡 VLANs 10 and 20 will segment the network into separate broadcast domains.

---

## ⚙️ Step 3. Assign Access Ports

### On SW1
```
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```

### On SW2
```
Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```

---

## ⚙️ Step 4. Configure the Trunk Link Between Switches
```
Switch(config)# interface fa0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
```

> 💡 This ensures VLAN 10 and VLAN 20 traffic can travel across switches.

---

## ⚙️ Step 5. Configure IPs on PCs

| PC | IP Address | Subnet Mask | Gateway |
|----|-------------|-------------|----------|
| PC1 | 192.168.10.10 | 255.255.255.0 | N/A |
| PC2 | 192.168.20.10 | 255.255.255.0 | N/A |

---

## ⚙️ Step 6. Verify Configuration

### VLANs
```
show vlan brief
```
### Trunks
```
show interfaces trunk
```

✅ VLAN 10 and 20 should appear as **active**, and **FastEthernet0/24** should be a trunk port.

---

## ⚙️ Step 7. Test Connectivity

1. Ping from **PC1 → PC2** (should **fail**; different VLANs).  
2. Add a new PC to VLAN 10 on SW2 and ping PC1 (should **succeed**).  

---

## 💡 Reflection

- What happens if a VLAN is missing on one switch?  
- Why does traffic between VLANs fail without routing?  
- What is the purpose of trunking in multi-switch environments?

---

## 🧭 Summary

| Concept | Description |
|----------|--------------|
| VLANs | Separate networks at Layer 2 |
| Access Ports | Connect hosts to specific VLANs |
| Trunk Links | Carry multiple VLANs between switches |
| Verification | `show vlan brief`, `show interfaces trunk` |

---
