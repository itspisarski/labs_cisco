# Cisco Networking Labs

This repository contains a comprehensive 5-day networking training program that progresses from traditional Cisco networking fundamentals to cloud-based AWS networking concepts. Each day builds upon the previous concepts, creating a complete learning journey from on-premises networking to cloud architecture.

## 📚 Course Overview

| Day | Focus Area | Technology Stack | Lab Count |
|-----|------------|------------------|-----------|
| **Day 1** | Network Fundamentals | Cisco Packet Tracer, IOS, Arista EOS | 9 Labs |
| **Day 2** | Advanced Switching | VLANs, Trunking, STP, EtherChannel | 3 Labs |
| **Day 3** | AWS Cloud Networking | VPC, Security Groups, NACLs | 3 Labs |
| **Day 4** | DNS & Load Balancing | Route 53, ALB, VPC Peering | 3 Labs |
| **Day 5** | High Availability & Monitoring | Auto Scaling, VPN, CloudWatch | 3 Labs |

## 🗂️ Repository Structure

```
labs_cisco/
├── NetworkingLabs/
│   ├── Day1/                       # Network Fundamentals (Cisco)
│   │   ├── 00_Cisco_Packet_Tracer_Install.md
│   │   ├── 01_Networking_Fundamentals_with_Cisco.md
│   │   ├── 02_VLSM_Address_Design.md
│   │   ├── 03_IPv6_Configuration.md
│   │   ├── 05_Cisco_IOS_Basics.md
│   │   ├── 06_IOS_Troubleshooting.md
│   │   ├── 07_Arista_EOS_Basics.md
│   │   ├── 08_EOS_Automation.md
│   │   ├── 10_Network_Diagnostic_Tools.md
│   │   └── README_Networking_Labs.md
│   ├── Day2/                       # Advanced Switching Concepts
│   │   ├── 01_VLANS_and_Trunking.md
│   │   ├── 02_Inter_VLAN_Routing.md
│   │   └── 03_STP_and_Etherchannel.md
│   ├── Day3/                       # AWS Cloud Networking Basics
│   │   ├── 01_Building_a_VPC_Architecture_in_aws.md
│   │   ├── 02_AWS_Security_Groups.md
│   │   └── 03_AWS_Network_ACLs.md
│   ├── Day4/                       # DNS and Load Balancing
│   │   ├── 01_DNS_Configuration_Route53.md
│   │   ├── 02_Load_Balanced_Web_Tier.md
│   │   └── 03_VPC_Peering_Hybrid_Connectivity.md
│   ├── Day5/                       # High Availability & Troubleshooting
│   │   ├── 01_Multi_AZ_High_Availability_Web_Application.md
│   │   ├── 02_VPN_Failover_and_CloudWatch_Monitoring.md
│   │   └── 03_Troubleshooting_Broken_VPC.md
│   └── ressources/                 # Supporting Materials
│       └── images.placeholder
└── README.md                       # This file
```

## 📖 Day-by-Day Breakdown

### **Day 1: Network Fundamentals** 🌐
**Location:** `NetworkingLabs/Day1/`  
**Focus:** Traditional networking concepts using Cisco technologies

- **Lab 00:** Cisco Packet Tracer Installation & Setup
- **Lab 01:** Networking Fundamentals with Cisco (OSI Model, TCP/IP, ARP)
- **Lab 02:** VLSM Address Design & Subnetting
- **Lab 03:** IPv6 Configuration & Addressing
- **Lab 05:** Cisco IOS Basics & Command Line
- **Lab 06:** IOS Troubleshooting Techniques
- **Lab 07:** Arista EOS Basics & Comparison
- **Lab 08:** EOS Automation & Scripting
- **Lab 10:** Network Diagnostic Tools & Utilities

**Key Technologies:** Cisco Packet Tracer, IOS, Arista EOS, IPv4/IPv6

### **Day 2: Advanced Switching** 🔀
**Location:** `NetworkingLabs/Day2/`  
**Focus:** Layer 2 switching technologies and protocols

- **Lab 01:** VLANs and Trunking Configuration
- **Lab 02:** Inter-VLAN Routing Implementation
- **Lab 03:** STP (Spanning Tree Protocol) & EtherChannel

**Key Technologies:** VLANs, Trunking, STP, EtherChannel, Layer 3 Switching

### **Day 3: AWS Cloud Networking Basics** ☁️
**Location:** `NetworkingLabs/Day3/`  
**Focus:** Introduction to AWS networking services

- **Lab 01:** Building a VPC Architecture in AWS
- **Lab 06:** Understanding and Configuring AWS Security Groups
- **Lab 07:** AWS Network ACLs (NACLs) and Subnet-Level Security

**Key Technologies:** AWS VPC, EC2, Security Groups, NACLs, Internet Gateways

### **Day 4: DNS and Load Balancing** ⚖️
**Location:** `NetworkingLabs/Day4/`  
**Focus:** Advanced AWS networking services

- **Lab 08:** Configuring DNS with Amazon Route 53
- **Lab 09:** Building a Load-Balanced Web Tier
- **Lab 10:** VPC Peering and Hybrid Connectivity

**Key Technologies:** Route 53, Application Load Balancer, VPC Peering, Target Groups

### **Day 5: High Availability & Troubleshooting** 🔧
**Location:** `NetworkingLabs/Day5/`  
**Focus:** Enterprise-grade resilience and monitoring

- **Lab 11:** Multi-AZ High Availability Web Application
- **Lab 12:** VPN Failover and CloudWatch Monitoring
- **Lab 13:** Troubleshooting a Broken VPC

**Key Technologies:** Auto Scaling Groups, Site-to-Site VPN, CloudWatch, Launch Templates

## 🎯 Learning Objectives

By completing this lab series, students will:

### **Traditional Networking Skills**
- ✅ Understand OSI and TCP/IP models
- ✅ Configure Cisco IOS and Arista EOS devices
- ✅ Implement VLANs, trunking, and inter-VLAN routing
- ✅ Design and implement VLSM addressing schemes
- ✅ Troubleshoot Layer 2 and Layer 3 connectivity issues

### **Cloud Networking Skills**
- ✅ Design and implement AWS VPC architectures
- ✅ Configure security groups and network ACLs
- ✅ Implement DNS services using Route 53
- ✅ Deploy load-balanced, multi-AZ applications
- ✅ Establish hybrid connectivity with VPN
- ✅ Monitor and troubleshoot cloud network issues

## 🛠️ Prerequisites

### **Software Requirements**
- **Cisco Packet Tracer** (Latest version)
- **AWS Account** (Free tier sufficient)
- **SSH Client** (PuTTY, Terminal, or similar)
- **Web Browser** (Chrome, Firefox, Safari)

### **Knowledge Prerequisites**
- Basic understanding of IP networking
- Familiarity with command-line interfaces
- Basic AWS account navigation (helpful but not required)

## 🚀 Getting Started

1. **Clone this repository:**
   ```bash
   git clone https://github.com/itspisarski/labs_cisco.git
   cd labs_cisco
   ```

2. **Navigate to the labs folder:**
   ```bash
   cd NetworkingLabs
   ```

3. **Start with Day 1:**
   ```bash
   cd Day1
   # Open 00_Cisco_Packet_Tracer_Install.md to begin
   ```

4. **Work through each day sequentially:**
   - Complete all labs in Day1 before moving to Day2
   - Follow step-by-step instructions in each lab
   - Complete reflection questions and verification steps

5. **Progress to AWS (Day 3+):**
   - Set up AWS account before starting Day 3
   - Follow lab instructions step-by-step
   - Use verification checklists to confirm completion

## 📋 Lab Format

Each lab includes:
- **🎯 Objective:** Clear learning goals
- **🧩 Topology:** Visual network diagram
- **⚙️ Step-by-step Instructions:** Detailed configuration steps
- **🔍 Verification:** Testing and validation procedures
- **💡 Reflection Questions:** Concept reinforcement
- **🧠 Key Takeaways:** Summary of important concepts

## 📁 Quick Navigation

| Day | Direct Links |
|-----|--------------|
| **Day 1** | [Network Fundamentals](./NetworkingLabs/Day1/) |
| **Day 2** | [Advanced Switching](./NetworkingLabs/Day2/) |
| **Day 3** | [AWS Cloud Networking](./NetworkingLabs/Day3/) |
| **Day 4** | [DNS & Load Balancing](./NetworkingLabs/Day4/) |
| **Day 5** | [High Availability & Troubleshooting](./NetworkingLabs/Day5/) |

## 🤝 Contributing

This is a training repository. For improvements or corrections:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request with clear descriptions

## 📞 Support

For questions or issues:
- Review lab prerequisites
- Check AWS documentation for cloud labs
- Consult Cisco documentation for traditional networking labs

## 📊 Progress Tracking

- [ ] Day 1: Network Fundamentals (9 labs)
- [ ] Day 2: Advanced Switching (3 labs)
- [ ] Day 3: AWS Cloud Networking (3 labs)
- [ ] Day 4: DNS & Load Balancing (3 labs)
- [ ] Day 5: High Availability & Troubleshooting (3 labs)

---

**Happy Learning! 🌟**

*From traditional networking to cloud architecture - master both worlds with hands-on labs.*