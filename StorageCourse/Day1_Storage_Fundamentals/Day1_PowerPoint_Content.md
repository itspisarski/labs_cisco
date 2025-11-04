# Day 1: Storage Fundamentals & Architecture
**PowerPoint Presentation Content & Teacher Notes**

---

## **SLIDE 1: Course Welcome & Agenda**
### Content:
- **Title:** Enterprise Storage Systems Course
- **Day 1 Agenda:**
  - Storage Evolution & Business Drivers (1 hour)
  - DAS Architecture Deep Dive (1.5 hours)
  - NAS Architecture & Protocols (2 hours)
  - Break (30 min)
  - SAN Architecture & Technologies (2 hours) 
  - Block vs File vs Object Storage (1 hour)

### Teacher Notes:
- Welcome participants, introduce yourself and your storage background
- Explain the hands-on nature of the course
- Set expectations for interaction and questions
- Mention that we'll progress from basic concepts to advanced Pure Storage implementations

---

## **SLIDE 2: Why Storage Matters**
### Content:
- **Data Growth Statistics:**
  - 90% of world's data created in last 2 years
  - Expected 175 zettabytes by 2025
  - IoT generating 73.1 ZB by 2025
- **Business Impact:**
  - Storage = 25-40% of IT infrastructure costs
  - Downtime costs: $5,600 per minute average
  - Performance directly affects user experience

### Teacher Notes:
- Use real-world examples from your experience
- Ask audience about their current storage challenges
- Emphasize that storage is not just about capacity anymore
- Mention that modern applications demand both performance and reliability

---

## **SLIDE 3: Storage Evolution Timeline**
### Content:
- **1950s-1980s:** Tape and Drum Storage
- **1980s-1990s:** Hard Disk Drives (HDD)
- **1990s-2000s:** RAID and Storage Networks
- **2000s-2010s:** Flash Storage Emergence
- **2010s-Present:** All-Flash Arrays, NVMe, Cloud Storage
- **Future:** Storage Class Memory, DNA Storage

### Teacher Notes:
- Briefly touch on each era - don't spend too much time here
- Focus on the acceleration of innovation in the last decade
- Connect evolution to business needs (speed, reliability, cost)
- Ask if anyone has worked with older storage technologies

---

## **SLIDE 4: Traditional Storage Challenges**
### Content:
- **Performance Bottlenecks:**
  - Spinning disk latency (5-15ms)
  - IOPS limitations
  - Unpredictable performance
- **Management Complexity:**
  - Multiple vendors and protocols
  - Manual provisioning
  - Capacity planning difficulties
- **Reliability Issues:**
  - Single points of failure
  - Complex disaster recovery
  - Long backup windows

### Teacher Notes:
- Get audience to share their pain points with traditional storage
- Explain how these challenges led to innovation
- Set up the context for why flash storage became necessary
- Use specific examples of how these issues impact business operations

---

## **SLIDE 5: DAS - Direct Attached Storage**
### Content:
- **Definition:** Storage directly connected to a single server
- **Types:**
  - Internal drives (SATA, SAS, NVMe)
  - External enclosures (USB, eSATA, SAS)
- **Advantages:**
  - High performance (no network overhead)
  - Simple configuration
  - Low cost for single server
- **Disadvantages:**
  - No sharing between servers
  - Limited scalability
  - Single point of failure

### Teacher Notes:
- Draw a simple diagram on whiteboard showing server with attached storage
- Explain that DAS is still relevant for specific use cases
- Give examples: database servers, high-performance computing
- Discuss when DAS makes sense vs. shared storage

---

## **SLIDE 6: DAS Implementation Example**
### Content:
- **Use Case:** Database Server with High IOPS Requirements
- **Configuration:**
  - Server: Dell PowerEdge R750
  - Storage: 8x NVMe SSDs in RAID 10
  - Capacity: 16TB usable
  - Performance: 800,000 IOPS
- **Benefits:** Predictable low latency, dedicated resources
- **Limitations:** Cannot share with other applications

### Teacher Notes:
- Walk through a real-world example
- Explain RAID levels briefly (detailed coverage comes later)
- Discuss why this configuration might be chosen
- Ask audience about their DAS experiences

---

## **SLIDE 7: NAS - Network Attached Storage**
### Content:
- **Definition:** File-level storage accessible over network
- **Key Characteristics:**
  - File sharing protocols (NFS, SMB/CIFS)
  - Network connectivity (Ethernet)
  - Multiple simultaneous clients
- **Components:**
  - NAS head (controller)
  - Storage pools
  - Network interfaces
  - File system

### Teacher Notes:
- Draw network diagram showing NAS serving multiple clients
- Explain the difference between block and file level access
- Discuss how NAS appears to clients (mapped drives, mount points)
- Mention that NAS includes both hardware and software components

---

## **SLIDE 8: NAS Protocols Deep Dive**
### Content:
- **NFS (Network File System):**
  - UNIX/Linux native
  - Versions: NFSv3, NFSv4, NFSv4.1
  - Stateless protocol
- **SMB/CIFS (Server Message Block):**
  - Windows native
  - Versions: SMB1, SMB2, SMB3
  - Stateful protocol
- **AFP (Apple Filing Protocol):**
  - macOS native (deprecated)

### Teacher Notes:
- Explain practical differences between protocols
- Discuss when to use each protocol
- Mention security considerations (SMB3 encryption, Kerberos)
- Give examples of cross-platform environments

---

## **SLIDE 9: NAS Architecture Components**
### Content:
- **NAS Head:**
  - CPU and memory for file operations
  - Multiple network ports (1GbE, 10GbE, 25GbE)
  - Storage controllers
- **Storage Backend:**
  - Disk shelves or flash arrays
  - RAID configurations
  - Hot spare drives
- **File System:**
  - Manages files and directories
  - Handles permissions and quotas
  - Provides data services (snapshots, replication)

### Teacher Notes:
- Show detailed architecture diagram
- Explain how NAS head processes I/O requests
- Discuss scale-out vs. scale-up NAS architectures
- Mention importance of file system choice (ZFS, NTFS, ext4)

---

## **SLIDE 10: NAS Use Cases & Benefits**
### Content:
- **Primary Use Cases:**
  - File shares and home directories
  - Content repositories
  - Backup targets
  - Development environments
- **Benefits:**
  - Easy to manage and provision
  - Cross-platform file sharing
  - Built-in data services
  - Good price/performance for file workloads

### Teacher Notes:
- Give specific examples from different industries
- Discuss why NAS is popular in small to medium businesses
- Explain the economics of shared storage vs. individual storage
- Ask audience about their file sharing requirements

---

## **SLIDE 11: SAN - Storage Area Network**
### Content:
- **Definition:** Block-level storage network connecting servers to storage
- **Key Characteristics:**
  - Dedicated storage network
  - Block-level access
  - High performance and availability
  - Centralized management
- **Components:**
  - Storage arrays
  - SAN switches
  - Host Bus Adapters (HBAs)
  - Storage fabric

### Teacher Notes:
- Draw comprehensive SAN diagram showing all components
- Emphasize that SAN provides block-level storage (like local disks)
- Explain the concept of a dedicated storage fabric
- Discuss why SANs were developed (performance, availability, manageability)

---

## **SLIDE 12: SAN Technologies**
### Content:
- **Fibre Channel (FC):**
  - Speeds: 8, 16, 32, 64, 128 Gbps
  - Low latency, high reliability
  - Dedicated network infrastructure
- **iSCSI (Internet SCSI):**
  - SCSI over Ethernet
  - Lower cost than FC
  - Uses existing network infrastructure
- **FCoE (Fibre Channel over Ethernet):**
  - FC frames over Ethernet
  - Converged network infrastructure

### Teacher Notes:
- Compare and contrast each technology
- Discuss when to choose each option
- Explain the trade-offs: performance vs. cost vs. complexity
- Show actual FC and iSCSI connections if equipment available

---

## **SLIDE 13: Fibre Channel Deep Dive**
### Content:
- **Architecture:**
  - Point-to-point, switched fabric, or arbitrated loop
  - WWN (World Wide Name) addressing
  - Zoning for security and performance
- **Components:**
  - FC switches (directors, edge switches)
  - HBAs in servers
  - Storage controllers
- **Advantages:**
  - Deterministic performance
  - Low CPU overhead
  - High availability features

### Teacher Notes:
- Explain FC addressing and zoning concepts
- Discuss FC switch architectures (core-edge, mesh)
- Show how FC provides isolation and security
- Mention FC management tools and best practices

---

## **SLIDE 14: iSCSI Implementation**
### Content:
- **Components:**
  - iSCSI initiator (client)
  - iSCSI target (storage)
  - Ethernet network
- **Addressing:**
  - IQN (iSCSI Qualified Name)
  - IP-based addressing
- **Performance Considerations:**
  - Network bandwidth and latency
  - TCP/IP overhead
  - Jumbo frames

### Teacher Notes:
- Show iSCSI configuration examples
- Explain naming conventions and discovery methods
- Discuss network design for iSCSI (VLANs, dedicated networks)
- Compare iSCSI performance to FC in real-world scenarios

---

## **SLIDE 15: Block vs File vs Object Storage**
### Content:
- **Block Storage:**
  - Raw storage blocks
  - Operating system creates file system
  - Examples: SAN LUNs, local disks
- **File Storage:**
  - File system provided by storage
  - Directory structure and metadata
  - Examples: NAS shares, cloud file systems
- **Object Storage:**
  - Data stored as objects with metadata
  - Accessed via REST APIs
  - Examples: Amazon S3, Azure Blob

### Teacher Notes:
- Use clear diagrams to show differences
- Explain when to use each type
- Give practical examples of each storage type
- Discuss how applications interact with each type

---

## **SLIDE 16: Block Storage Characteristics**
### Content:
- **Features:**
  - Fixed-size blocks (512B, 4KB, etc.)
  - Direct attachment to operating system
  - High performance for databases
  - Supports any file system
- **Use Cases:**
  - Database storage
  - Virtual machine storage
  - High-performance applications
- **Limitations:**
  - Requires file system management
  - Limited sharing capabilities

### Teacher Notes:
- Explain block size implications for performance
- Show how operating systems see block storage
- Discuss partitioning and file system options
- Give examples of block storage performance optimization

---

## **SLIDE 17: File Storage Characteristics**
### Content:
- **Features:**
  - Hierarchical directory structure
  - File-level permissions and metadata
  - Network accessibility
  - Built-in sharing protocols
- **Use Cases:**
  - User home directories
  - Shared application data
  - Content management
  - Backup and archival
- **Advantages:**
  - Easy to manage and share
  - Cross-platform compatibility
  - Built-in data services

### Teacher Notes:
- Show file system hierarchy examples
- Explain file permissions and access control
- Discuss file system scalability limits
- Give examples of file storage optimization

---

## **SLIDE 18: Object Storage Characteristics**
### Content:
- **Features:**
  - Flat namespace (no directories)
  - Rich metadata
  - REST API access
  - Virtually unlimited scalability
- **Use Cases:**
  - Cloud applications
  - Content distribution
  - Data archival and backup
  - Big data analytics
- **Advantages:**
  - Massive scalability
  - Built-in redundancy
  - Application integration

### Teacher Notes:
- Explain how object storage differs from traditional storage
- Show REST API examples
- Discuss object storage use cases in cloud applications
- Mention major object storage providers and protocols

---

## **SLIDE 19: Storage Selection Criteria**
### Content:
- **Performance Requirements:**
  - IOPS and throughput needs
  - Latency sensitivity
  - Concurrent user count
- **Capacity and Growth:**
  - Current and future capacity
  - Growth rate and patterns
- **Availability and Reliability:**
  - Uptime requirements
  - Disaster recovery needs
  - Data protection requirements

### Teacher Notes:
- Create a decision matrix on whiteboard
- Walk through real-world selection scenarios
- Discuss how requirements drive technology choices
- Ask audience to share their selection criteria

---

## **SLIDE 20: Day 1 Lab Exercise Preview**
### Content:
- **Lab 1.1:** Configure iSCSI Storage
- **Lab 1.2:** Set up NFS File Sharing
- **Lab 1.3:** Compare Block vs File Performance
- **Duration:** 2 hours hands-on
- **Tools:** VMware environment, Linux and Windows VMs

### Teacher Notes:
- Explain lab objectives and expected outcomes
- Review lab environment and access instructions
- Set expectations for lab completion time
- Mention that labs reinforce concepts covered in lectures

---

## **SLIDE 21: Day 1 Wrap-up & Day 2 Preview**
### Content:
- **Today's Key Concepts:**
  - Storage architecture types (DAS, NAS, SAN)
  - Storage access methods (block, file, object)
  - Protocol considerations
- **Tomorrow's Focus:**
  - Flash storage technology deep dive
  - Performance characteristics and optimization
  - NVMe and storage class memory

### Teacher Notes:
- Summarize key learning points
- Answer any remaining questions
- Preview exciting flash storage topics for tomorrow
- Assign reading materials if available

---

## **Additional Teaching Materials:**

### **Hands-on Lab Exercises:**

#### **Lab 1.1: Configure iSCSI Storage (45 minutes)**
1. Set up Linux iSCSI target
2. Configure Windows iSCSI initiator
3. Create and mount iSCSI LUN
4. Test block-level access

#### **Lab 1.2: Set up NFS File Sharing (45 minutes)**
1. Configure NFS server on Linux
2. Export file systems with different permissions
3. Mount NFS shares on multiple clients
4. Test concurrent file access

#### **Lab 1.3: Performance Comparison (30 minutes)**
1. Benchmark iSCSI block storage performance
2. Benchmark NFS file storage performance
3. Compare results and analyze differences
4. Document findings and conclusions

### **Assessment Quiz (20 questions):**
1. Multiple choice questions on storage architectures
2. Scenario-based questions on storage selection
3. Technical questions on protocols and performance

### **Reference Materials:**
- Storage networking industry standards
- Vendor documentation links
- Performance benchmarking tools
- Troubleshooting guides

---

**Total Day 1 Duration: 8 hours**
- Lectures: 5 hours
- Labs: 2 hours  
- Breaks and discussions: 1 hour