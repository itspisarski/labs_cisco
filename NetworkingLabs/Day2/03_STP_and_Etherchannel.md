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

## ⚙️ Step 1. Verify Default STP Configuration
```
show spanning-tree
```
Observe:
- Root Bridge
- Port roles (Root, Designated)
- Port states (Forwarding, Blocking)

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
