# Lab 01 – OSI and Encapsulation Visualization

**Objective:** Understand how data is encapsulated through OSI and TCP/IP layers.

### 🧩 Topology
PC1 — Switch — PC2

| Device | Interface | IP Address |
|---------|------------|-------------|
| PC1 | NIC | 192.168.1.1/24 |
| PC2 | NIC | 192.168.1.2/24 |


## OSI and Encapsulation Visualization 

### ⚙️ Steps

1. Open Cisco Packet Tracer and add two PCs and one switch (switch 2960).
3. Connect PCs using Copper Straight-Through cables to the switch. You can use any ethernet interface.
5. Assign IPs to both PCs.
6. Click on one of the PC
7. At the top, choose the config tab
8. Select the FastEthernet0 interface
9. Set the ip adress for each PC and the subnet (255.255.255.0 wil be used by default)
10. Ping from PC1 to PC2.
11. To ping PC2 from PC1, click on PC1
12. Move to Desktop
13. CHoose Command Prompt
14. Switch to **Simulation Mode** and observe ICMP encapsulation.

## – ARP and MAC Addressing

**Objective:** Observe ARP request and MAC resolution within a subnet.

### 🧩 Topology
PC1 — Switch — PC2

| Device | IP Address |
|---------|-------------|
| PC1 | 192.168.10.1/24 |
| PC2 | 192.168.10.2/24 |

### ⚙️ Steps
1. Connect two PCs to a switch.
2. Assign IPs to both PCs.
3. On PC1,  run `arp -a` to see empty cache.
-> Expected output : No ARP Entries Found
5. Ping PC2 and re-run `arp -a`.
6. Observe ARP broadcast and reply in Simulation mode.
-> Internet Address      Physical Address      Type
   192.168.10.2          00e0.f9dd.084c        dynamic

## Subnetting and IP Planning 

### 🧩 Topology

    R1     
  /    \\     
R2  _   R3
1. Divide `192.168.50.0/24` into four `/26` subnets.
2. Create 3 routers in triangle topology using the router 2911.
3. Assign subnets between routers. 
4. Configure interfaces using CLI.
5. Verify with `show ip interface brief`.
6. Enable static routes between routers.
