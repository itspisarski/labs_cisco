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

### Router Verification
1. **Check interface status:**
   ```
   Router# show ip interface brief
   ```
   - **Expected Output:**
   ```
   Interface              IP-Address      OK? Method Status                Protocol
   GigabitEthernet0/0     unassigned      YES unset  up                    up
   GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up
   GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up
   ```

2. **Verify subinterface configuration:**
   ```
   Router# show interfaces g0/0.10
   Router# show interfaces g0/0.20
   ```
   - **Look for:** Encapsulation dot1Q and correct IP addresses

3. **Check routing table:**
   ```
   Router# show ip route connected
   ```
   - **Expected:** Connected routes for both VLAN subnets

### Switch Verification
1. **Verify VLAN database:**
   ```
   Switch# show vlan brief
   ```
   - **Expected Output:**
   ```
   VLAN Name                             Status    Ports
   ---- -------------------------------- --------- -------------------------------
   1    default                          active    Fa0/3, Fa0/4, Fa0/5, ...
   10   HR                              active    Fa0/1
   20   FINANCE                         active    Fa0/2
   ```

2. **Check trunk configuration:**
   ```
   Switch# show interfaces fa0/24 trunk
   ```
   - **Expected:** VLANs 1,10,20 should be allowed and active

3. **Verify port assignments:**
   ```
   Switch# show interfaces fa0/1 switchport
   Switch# show interfaces fa0/2 switchport
   ```
   - **Check:** Mode should be "access" and VLAN assignment correct

### Detailed Trunk Verification
```
Switch# show interfaces trunk
```
- **Expected Output:**
```
Port        Mode         Encapsulation  Status        Native vlan
Fa0/24      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/24      1-4094

Port        Vlans allowed and active in management domain
Fa0/24      1,10,20

Port        Vlans in spanning tree forwarding state
Fa0/24      1,10,20
```

### Configuration Checklist
| Component | Command | What to Verify | Status |
|-----------|---------|----------------|---------|
| Router Physical Interface | `show ip int brief` | G0/0 is up/up | ✅ |
| Router Subinterfaces | `show ip int brief` | G0/0.10 and G0/0.20 up/up | ✅ |
| VLAN Creation | `show vlan brief` | VLANs 10,20 exist and active | ✅ |
| Access Ports | `show int fa0/X switchport` | Correct VLAN assignment | ✅ |
| Trunk Port | `show int trunk` | Fa0/24 trunking VLANs 10,20 | ✅ |

✅ **All components must be properly configured before testing connectivity.**

---

## ⚙️ Step 6. Test Inter-VLAN Communication

### Test 1: Verify Gateway Connectivity
1. **Test PC1 to its gateway (Router subinterface):**
   ```
   C:\> ping 192.168.10.1
   ```
   - **Expected Result:** ✅ **Reply from 192.168.10.1** (PC1 can reach its default gateway)

2. **Test PC2 to its gateway (Router subinterface):**
   ```
   C:\> ping 192.168.20.1
   ```
   - **Expected Result:** ✅ **Reply from 192.168.20.1** (PC2 can reach its default gateway)

### Test 2: Inter-VLAN Communication
1. **From PC1 (VLAN 10), ping PC2 (VLAN 20):**
   ```
   C:\> ping 192.168.20.10
   ```
   - **Expected Result:** ✅ **Reply from 192.168.20.10** (Inter-VLAN routing successful)
   - **Traffic Path:** PC1 → SW1 → Router (VLAN 10 → VLAN 20) → SW2 → PC2

2. **From PC2 (VLAN 20), ping PC1 (VLAN 10):**
   ```
   C:\> ping 192.168.10.10
   ```
   - **Expected Result:** ✅ **Reply from 192.168.10.10** (Bidirectional routing works)

### Test 3: Trace the Routing Path
1. **From PC1, trace route to PC2:**
   ```
   C:\> tracert 192.168.20.10
   ```
   - **Expected Output:**
   ```
   1    <1 ms    <1 ms    <1 ms    192.168.10.1    (Router Gateway)
   2    <1 ms    <1 ms    <1 ms    192.168.20.10   (PC2 Destination)
   ```
   - **Analysis:** Traffic goes through the router to reach the other VLAN

### Test 4: Verify ARP Tables
1. **Check ARP table on PC1:**
   ```
   C:\> arp -a
   ```
   - **Expected Entry:** `192.168.10.1` should show the router's MAC address
   
2. **Check router's ARP table:**
   ```
   Router# show ip arp
   ```
   - **Expected Entries:** Should show both PCs' MAC addresses learned through subinterfaces

### Troubleshooting Tests (If Pings Fail)
1. **Verify router subinterfaces are UP:**
   ```
   Router# show ip interface brief
   ```
   - **Look for:** Both G0/0.10 and G0/0.20 should show "up/up"

2. **Check VLAN configuration on switch:**
   ```
   Switch# show vlan brief
   ```
   - **Verify:** VLANs 10 and 20 are active and ports are assigned correctly

3. **Verify trunk configuration:**
   ```
   Switch# show interfaces fa0/24 switchport
   ```
   - **Look for:** Mode should be "trunk" and VLANs 10,20 should be allowed

4. **Check router routing table:**
   ```
   Router# show ip route
   ```
   - **Expected:** Connected routes for 192.168.10.0/24 and 192.168.20.0/24

### Expected Results Summary
| Test | Source | Destination | Expected Result | Purpose |
|------|--------|-------------|----------------|---------|
| Gateway Test | PC1 | 192.168.10.1 | ✅ Success | Verify L3 connectivity to router |
| Gateway Test | PC2 | 192.168.20.1 | ✅ Success | Verify L3 connectivity to router |
| Inter-VLAN | PC1 | PC2 (192.168.20.10) | ✅ Success | Confirm router-on-a-stick works |
| Inter-VLAN | PC2 | PC1 (192.168.10.10) | ✅ Success | Confirm bidirectional routing |

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
