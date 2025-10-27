Lab 03 : 

**Objective:**  

In this lab, you will configure IPv6 addresses on routers and PCs in Cisco Packet Tracer, verify IPv6 communication, and understand IPv6 addressing concepts such as link-local, global unicast, and EUI-64 address generation.

---

## 🧩 Topology

[PC1] --- [Router1] --- [Router2] --- [PC2]


Each router connects to a PC on one side (LAN) and to the other router on the opposite side (WAN).

---

## 🧰 Devices and Interfaces

| Device | Interface | Network | Example IPv6 Address |
|---------|------------|----------|-----------------------|
| PC1 | NIC | LAN1 | 2001:db8:1::10/64 |
| Router1 | G0/0 | LAN1 | 2001:db8:1::1/64 |
| Router1 | G0/1 | WAN | 2001:db8:12::1/64 |
| Router2 | G0/0 | LAN2 | 2001:db8:2::1/64 |
| Router2 | G0/1 | WAN | 2001:db8:12::2/64 |
| PC2 | NIC | LAN2 | 2001:db8:2::10/64 |

---

## ⚙️ Step 1. Build the Topology in Packet Tracer

1. Open **Cisco Packet Tracer**.
2. Drag and drop:
   - **2 Routers** (e.g., 2911 or 1941)
   - **2 PCs**
   - **2 Switches**
3. Connect devices:
   - PC1 → Switch1 → Router1 G0/0
   - Router1 G0/1 → Router2 G0/1 (WAN link)
   - Router2 G0/0 → Switch2 → PC2

> 💡 Use **Copper Straight-Through** cables for PC–Switch and Switch–Router links.

---

## ⚙️ Step 2. Enable IPv6 Routing

On both routers:

```
Router(config)# ipv6 unicast-routing
```
> This command allows the router to forward IPv6 packets between interfaces.

---

## ⚙️ Step 3. Configure IPv6 Addresses on Routers

### Router1 Configuration

```
Router1> enable
Router1# configure terminal

# LAN interface
Router1(config)# interface gigabitEthernet0/0
Router1(config-if)# ipv6 address 2001:db8:1::1/64
Router1(config-if)# no shutdown

# WAN interface
Router1(config)# interface gigabitEthernet0/1
Router1(config-if)# ipv6 address 2001:db8:12::1/64
Router1(config-if)# no shutdown
Router1(config-if)# exit

```

### Router2 Configuration

```
Router2> enable
Router2# configure terminal

# LAN interface
Router2(config)# interface gigabitEthernet0/0
Router2(config-if)# ipv6 address 2001:db8:2::1/64
Router2(config-if)# no shutdown

# WAN interface
Router2(config)# interface gigabitEthernet0/1
Router2(config-if)# ipv6 address 2001:db8:12::2/64
Router2(config-if)# no shutdown
Router2(config-if)# exit
```

---

## ⚙️ Step 4. Configure PCs

In Packet Tracer, click on **PC1** → **Desktop Tab** → **IP Configuration**.

- **IPv6 Address:** `2001:db8:1::10`
- **Prefix Length:** `64`
- **Default Gateway:** `2001:db8:1::1`

Then do the same for **PC2**:

- **IPv6 Address:** `2001:db8:2::10`
- **Prefix Length:** `64`
- **Default Gateway:** `2001:db8:2::1`

---

## ⚙️ Step 5. Configure IPv6 Static Routes

To allow the two LANs to communicate, configure static routes.

### On Router1
```
Router1(config)# ipv6 route 2001:db8:2::/64 2001:db8:12::2
```

### On Router2
```
Router2(config)# ipv6 route 2001:db8:1::/64 2001:db8:12::1
```

> These routes tell each router how to reach the other LAN through the WAN link.

---

## ⚙️ Step 6. Verify Configuration

### 1. Show IPv6 Interfaces
```
Router# show ipv6 interface brief

Expected output example:
Gig0/0 [up/up] 2001:db8:1::1/64 FE80::1
Gig0/1 [up/up] 2001:db8:12::1/64 FE80::2
```

### 2. Show IPv6 Routes
```
Router# show ipv6 route


Expected output:
C 2001:db8:1::/64 [Connected]
C 2001:db8:12::/64 [Connected]
S 2001:db8:2::/64 [Static]
```

---

## ⚙️ Step 7. Test Connectivity

### From Router1
```
Router1# ping 2001:db8:12::2
Router1# ping 2001:db8:2::10
```

### From PC1 (Desktop → Command Prompt)
```
ping 2001:db8:2::10
```

✅ You should receive **successful ping replies** between all devices.

If not:
- Check that all interfaces are **up**.
- Verify static routes on both routers.
- Ensure PCs have correct **IPv6 gateways**.

---

## ⚙️ Step 8. Observe EUI-64 Address Generation

1. Remove a manual address from an interface:
```
Router(config-if)# no ipv6 address 2001:db8:1::1/64
```
2. Enable automatic addressing using EUI-64:
```
Router(config-if)# ipv6 address 2001:db8:1::/64 eui-64
```
3. Verify:
```
Router# show ipv6 interface g0/0
```
You’ll see the router auto-generated the interface ID using its **MAC address**.

---

## ⚙️ Step 9. Observe Link-Local Addresses

Check link-local addresses:
```
Router# show ipv6 interface g0/0
```

All active IPv6 interfaces should automatically generate a **link-local address** beginning with `FE80::/10`.

> These addresses are **not routable** and are used only for communication within the same local link.

---

## 🔍 Verification Checklist

| Task | Command | Expected Result |
|------|----------|----------------|
| IPv6 enabled | `show running-config` | `ipv6 unicast-routing` present |
| Interfaces configured | `show ipv6 interface brief` | IPv6 addresses assigned |
| Static routes added | `show ipv6 route` | Routes visible for all LANs |
| Connectivity test | `ping` | Successful responses |
| EUI-64 tested | `show ipv6 interface` | Auto-generated address |
| Link-local verified | `show ipv6 interface` | FE80:: address visible |

---

## 💡 Reflection Questions

1. What is the difference between **global unicast** and **link-local** IPv6 addresses?  
2. How does **EUI-64** derive the interface identifier from a MAC address?  
3. Why must you enable `ipv6 unicast-routing` on routers in Packet Tracer?  
4. What happens if you omit static routes between Router1 and Router2?

---

## 📝 Student Notes

> Use this section to record your interface addresses, `ping` results, and answers to reflection questions.  
> Include screenshots of successful tests if required.

---

## 🧭 Summary

| Concept | Description |
|----------|--------------|
| IPv6 Enablement | `ipv6 unicast-routing` enables IPv6 packet forwarding |
| Address Assignment | Global unicast addresses are configured manually |
| Static Routing | Ensures connectivity between LANs |
| Link-Local | Used for local communication (FE80::/10) |
| EUI-64 | Auto-generates interface ID using MAC |
| Verification | Performed using `ping` and `show ipv6` commands |

---