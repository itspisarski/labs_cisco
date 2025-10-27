# Lab 01 – Networking Fundamentals with Cisco Packet Tracer

**Objective:**  
In this introductory lab, you will use Cisco Packet Tracer to build basic Ethernet networks, explore the OSI and TCP/IP models, observe data encapsulation, understand ARP and MAC addressing, and practice subnetting and IP planning.

---

## 🧩 Topology 1 – Basic LAN

[PC1] --- [Switch] --- [PC2]


| Device | Interface | IP Address |
|---------|------------|-------------|
| PC1 | NIC | 192.168.1.1/24 |
| PC2 | NIC | 192.168.1.2/24 |

---

## ⚙️ Step 1. Build the Network in Packet Tracer

1. Open **Cisco Packet Tracer**.
2. Add **two PCs** and **one switch** (e.g., `2960`).
3. Connect both PCs to the switch using **Copper Straight-Through** cables.
4. Use any available Ethernet interface on each PC (usually `FastEthernet0`).

---

## ⚙️ Step 2. Configure IP Addresses on PCs

1. Click **PC1** → **Desktop Tab** → **IP Configuration**.  
   - IPv4 Address: `192.168.1.1`  
   - Subnet Mask: `255.255.255.0`
2. Do the same for **PC2**:  
   - IPv4 Address: `192.168.1.2`  
   - Subnet Mask: `255.255.255.0`

> 💡 The subnet mask `255.255.255.0` is automatically filled when you type `/24` in Packet Tracer.

---

## ⚙️ Step 3. Test Basic Connectivity

1. Click **PC1** → **Desktop Tab** → **Command Prompt**.  
2. Type:  
```ping 192.168.1.2```


3. You should receive **successful ping replies**.

If the ping fails:
- Check that both PCs are on the same subnet.
- Ensure cables are properly connected.
- Wait a few seconds for ARP resolution.

---

## ⚙️ Step 4. Observe OSI Encapsulation

1. Switch to **Simulation Mode** in Packet Tracer.
2. From **PC1**, open **Command Prompt** and send a ping to `192.168.1.2`.
3. Watch the packet flow in the simulation pane:
- Observe **ICMP encapsulation** inside **IP** and **Ethernet** frames.
- Click each packet in the event list to view the **encapsulation details** at each OSI layer.

> 🧠 Note how data moves from Layer 7 (Application) down to Layer 1 (Physical).

---

## 🧩 Topology 2 – ARP and MAC Address Resolution

[PC1] --- [Switch] --- [PC2]


| Device | IP Address |
|---------|-------------|
| PC1 | 192.168.10.1/24 |
| PC2 | 192.168.10.2/24 |

---

## ⚙️ Step 5. Build and Configure the ARP Network

1. Add two PCs and one switch.  
2. Connect them using **Copper Straight-Through** cables.  
3. Configure IP addresses as above.

---

## ⚙️ Step 6. Observe the ARP Process

1. On **PC1**, open **Command Prompt**.  
2. Run:  
``` 
arp -a 
Expected Output: *No ARP Entries Found*  
```
3. Ping PC2: 

```ping 192.168.10.2```

4. Run 
```
arp -a 
  
Example output:
Internet Address Physical Address Type
192.168.10.2 00e0.f9dd.084c dynamic
```


5. Switch to **Simulation Mode** and replay the ARP request and reply packets.

> 💡 ARP dynamically maps IP addresses to MAC addresses, allowing devices to communicate on a LAN.

---

## 🧩 Topology 3 – Subnetting and IP Planning


```
     [R1]
    /    \
 [R2] -- [R3]
```

---

## ⚙️ Step 7. Subnet and Plan the Network

1. Starting from `192.168.50.0/24`, divide the network into **four `/26` subnets**.  
   | Subnet | Network | Range | Broadcast |
   |---------|----------|--------|------------|
   | A | 192.168.50.0/26 | .1 – .62 | .63 |
   | B | 192.168.50.64/26 | .65 – .126 | .127 |
   | C | 192.168.50.128/26 | .129 – .190 | .191 |
   | D | 192.168.50.192/26 | .193 – .254 | .255 |

2. Create a **triangle topology** using **three 2911 routers**.  
3. Assign one `/26` subnet to each router-to-router link.  
4. Assign router interface IPs from each subnet (e.g., `.1` and `.2` per link).

---

## ⚙️ Step 8. Configure Router Interfaces

Example for Router1:

```
Router1> enable
Router1# configure terminal
Router1(config)# interface g0/0
Router1(config-if)# ip address 192.168.50.1 255.255.255.192
Router1(config-if)# no shutdown
Router1(config-if)# exit
```

Repeat for Router2 and Router3, adapting IPs according to your subnet plan.

---

## ⚙️ Step 9. Verify Connectivity Between Routers

1. Run the command:
```
show ip interface brief
```

Verify that all interfaces are up and assigned correctly.

2. Test connectivity:
```
ping <neighbor IP>
```

> 💡 If pings fail, ensure subnets match, interfaces are up, and cables are correct.

---

## ⚙️ Step 10. Configure Static Routes (Optional)

To allow full network reachability:
```
Router1(config)# ip route 192.168.50.128 255.255.255.192 192.168.50.2
Router2(config)# ip route 192.168.50.0 255.255.255.192 192.168.50.1
```

> 🧭 This step helps understand **Layer 3 routing** and inter-router communication.

---

## 🔍 Verification Checklist

| Task | Command | Expected Result |
|------|----------|----------------|
| PCs connected | Visual check | Green link lights |
| IPs assigned | `ipconfig` | Correct IP and subnet |
| Ping success | `ping` | Replies received |
| ARP verified | `arp -a` | Dynamic MAC entry visible |
| Router interfaces up | `show ip interface brief` | All interfaces up/up |
| Static routes | `show ip route` | Routes visible |

---

## 💡 Reflection Questions

1. Which OSI layer handles MAC addressing?  
2. What protocol resolves IP addresses to MAC addresses?  
3. How does subnetting affect broadcast domains?  
4. What’s the difference between a switch and a router in this lab?  
5. How does Packet Tracer’s Simulation Mode help visualize network communication?

---

## 📝 Student Notes

> Use this space to record your IP plan, observations, and answers to reflection questions.  
> Include screenshots or notes from Simulation Mode.

---

## 🧭 Summary

| Concept | Description |
|----------|--------------|
| Network Creation | Built basic Ethernet networks in Packet Tracer |
| OSI Encapsulation | Observed packet flow across layers |
| ARP | Learned how IPs map to MACs |
| Subnetting | Divided networks into smaller subnets |
| IP Planning | Assigned IPs to routers and PCs logically |
| Verification | Tested end-to-end connectivity with pings |

---
