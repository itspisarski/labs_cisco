# Lab 03 – Spanning Tree Protocol and EtherChannel

**Objective:**  
Understand and verify Spanning Tree Protocol (STP) operation and configure EtherChannel (LACP) for redundancy and load balancing in Packet Tracer.

---

## 🧩 Topology

```

    [SW1]
    /   \
 [SW2]  [SW3]
```

All switches are interconnected using redundant links.

---

## ⚙️ Step 1. Build the Topology and Verify Default STP Configuration

### Topology Setup
1. **Add 3 switches** (2960 series) to Packet Tracer
2. **Create redundant connections:**
   - SW1 to SW2: FastEthernet0/1 ↔ FastEthernet0/1
   - SW1 to SW3: FastEthernet0/2 ↔ FastEthernet0/2  
   - SW2 to SW3: FastEthernet0/3 ↔ FastEthernet0/3
3. **Add PCs** for testing:
   - PC1 connected to SW2 FastEthernet0/10
   - PC2 connected to SW3 FastEthernet0/10

### Initial STP Verification
1. **Check STP status on all switches:**
   ```
   Switch# show spanning-tree
   ```

2. **Analyze the output for SW1:**
   ```
   Switch# show spanning-tree vlan 1
   ```
   - **Expected Information:**
     - Bridge ID (Priority + MAC address)
     - Root Bridge identification
     - Port roles and states

3. **Example expected output:**
   ```
   VLAN0001
     Spanning tree enabled protocol ieee
     Root ID    Priority    32769
                Address     000C.CF8A.4A80
                This bridge is the root
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
   
     Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     000C.CF8A.4A80
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec
   
   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Fa0/1              Desg FWD 19        128.1    P2p
   Fa0/2              Desg FWD 19        128.2    P2p
   ```

4. **Check which ports are blocking:**
   ```
   Switch# show spanning-tree interface fa0/3
   ```
   - **Look for:** Port state (Forwarding/Blocking)
   - **Understand:** Why certain ports block to prevent loops

### STP Analysis Checklist
| Switch | Check | Command | Expected Result |
|--------|-------|---------|----------------|
| SW1 | Root Bridge Status | `show spanning-tree` | Should show "This bridge is the root" |
| SW2 | Port Roles | `show spanning-tree vlan 1` | Mix of Root/Designated ports |
| SW3 | Blocking Ports | `show spanning-tree` | At least one port should be blocking |
| All | Convergence | `show spanning-tree summary` | All VLANs should be forwarding |

---

## ⚙️ Step 2. Configure the Root Bridge
```
SW1(config)# spanning-tree vlan 1 priority 4096
```
✅ SW1 becomes the **Root Bridge** (lower priority = higher priority for election).

---

## ⚙️ Step 3. Enable PortFast and BPDU Guard

Apply to all **access ports** (connected to PCs):
```
SW2(config)# interface range fa0/1 - 5
SW2(config-if-range)# spanning-tree portfast
SW2(config-if-range)# spanning-tree bpduguard enable
```
> 💡 PortFast reduces convergence time, BPDU Guard prevents loops.

---

## ⚙️ Step 4. Configure EtherChannel Between SW2 and SW3
```
SW2(config)# interface range fa0/21 - 22
SW2(config-if-range)# channel-group 1 mode active

SW3(config)# interface range fa0/21 - 22
SW3(config-if-range)# channel-group 1 mode passive
```
This enables **LACP (Link Aggregation Control Protocol)** negotiation.

---

## ⚙️ Step 5. Verify EtherChannel
```
show etherchannel summary
show interfaces port-channel 1
```
✅ You should see `Port-channel1` as **up**, with Fa0/21–22 as members.

---

## ⚙️ Step 6. Verify STP After EtherChannel
```
show spanning-tree
```
> The EtherChannel will appear as **one logical link** to STP, reducing loops.

---

## ⚙️ Step 7. Observe Failure Recovery (Optional)

1. Disable one EtherChannel link:
```
SW2(config)# interface fa0/21
SW2(config-if)# shutdown
```
2. Observe that traffic still flows — redundancy achieved.

---

## 💡 Reflection

- Why is PortFast not used on trunk or uplink ports?  
- How does LACP help redundancy?  
- What happens if STP and EtherChannel are both disabled?

---

## 🧭 Summary

| Concept | Description |
|----------|--------------|
| STP | Prevents switching loops |
| Root Bridge | Central reference in STP topology |
| PortFast / BPDU Guard | Speed up access port activation and protect from loops |
| EtherChannel | Combines multiple links for redundancy and load balancing |

---
