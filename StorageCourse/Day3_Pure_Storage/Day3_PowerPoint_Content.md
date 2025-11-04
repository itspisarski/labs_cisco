# Day 3: Pure Storage Platforms
**PowerPoint Presentation Content & Teacher Notes**

---

## **SLIDE 1: Day 3 Agenda**
### Content:
- **Pure Storage Company Overview** (30 minutes)
- **FlashArray//X Architecture** (2.5 hours)
- **Break** (30 minutes)
- **FlashBlade//S Scale-Out Architecture** (2 hours)
- **Purity Operating Environment** (1.5 hours)
- **Hands-on Pure Storage Labs** (1 hour)

### Teacher Notes:
- Welcome participants to Pure Storage deep dive
- Emphasize practical, hands-on approach
- Mention that Pure Storage revolutionized enterprise storage
- Set expectations for detailed technical content

---

## **SLIDE 2: Pure Storage: Disrupting Enterprise Storage**
### Content:
- **Founded:** 2009 by Dietzen, Colgrove, and Miller
- **Mission:** "Make data storage simple, affordable, and reliable"
- **Key Innovation:** All-flash arrays with advanced software
- **Market Position:**
  - Leader in all-flash array market
  - $2.8B+ annual revenue (2023)
  - 11,000+ customers worldwide
- **Core Principles:**
  - Software-defined storage
  - Customer-centric design
  - Continuous innovation

### Teacher Notes:
- Explain how Pure Storage challenged traditional storage vendors
- Discuss the founding team's background (storage industry veterans)
- Show market share and growth statistics
- Emphasize cultural focus on simplicity and customer success

---

## **SLIDE 3: Pure Storage Product Portfolio**
### Content:
- **FlashArray:**
  - Block storage for databases, VMs, containers
  - Models: //X, //C, //XL
- **FlashBlade:**
  - Scale-out file and object storage
  - Models: //S, //E
- **Pure Cloud Block Store:**
  - Native cloud storage services
  - AWS, Azure, GCP integration
- **Portworx:**
  - Kubernetes-native storage platform
  - Acquired 2020 for container storage

### Teacher Notes:
- Position each product in the overall storage ecosystem
- Explain target use cases for each platform
- Show how products complement each other
- Discuss Pure's cloud and container strategy evolution

---

## **SLIDE 4: FlashArray//X Overview**
### Content:
- **Architecture Type:** Scale-up block storage array
- **Target Workloads:**
  - Mission-critical databases (Oracle, SQL Server, SAP)
  - Virtualized environments (VMware, Hyper-V)
  - Container orchestration platforms
- **Key Features:**
  - Always-on data reduction
  - DirectFlash technology
  - ActiveCluster synchronous replication
  - 99.9999% uptime guarantee

### Teacher Notes:
- Position FlashArray as Pure's flagship product
- Explain scale-up vs. scale-out architectures
- Emphasize enterprise reliability and performance
- Connect to customer use cases from your experience

---

## **SLIDE 5: FlashArray//X Hardware Architecture**
### Content:
- **Chassis Design:**
  - 3U rackmount appliance
  - Dual controllers for high availability
  - Hot-swappable components
- **Controller Specifications:**
  - Intel Xeon processors
  - Large NVRAM write buffers
  - Multiple host connectivity options
- **Storage Media:**
  - DirectFlash Modules (DFMs)
  - Raw capacities: 18TB to 150TB
  - Effective capacity: 70TB to 600TB (with data reduction)

### Teacher Notes:
- Show detailed hardware diagrams if available
- Explain dual-controller active-active design
- Discuss hot-swap capabilities and maintenance procedures
- Emphasize data reduction impact on effective capacity

---

## **SLIDE 6: DirectFlash Technology**
### Content:
- **Pure's Custom Flash Modules:**
  - Direct NAND flash access (no SSD controllers)
  - Custom firmware and flash management
  - Optimized for Pure's software stack
- **Benefits vs. Standard SSDs:**
  - Lower latency (no SSD controller overhead)
  - Better predictability and consistency
  - Tighter integration with Purity OS
  - Higher reliability and endurance
- **Technical Advantages:**
  - Native NVMe interface
  - Advanced error correction
  - Wear leveling optimization

### Teacher Notes:
- Explain why Pure built custom flash modules
- Show latency comparisons vs. commodity SSDs
- Discuss vertical integration benefits
- Connect to reliability and performance advantages

---

## **SLIDE 7: FlashArray//X Data Services**
### Content:
- **Always-On Data Reduction:**
  - Inline deduplication and compression
  - Pattern removal and thin provisioning
  - Typical 5:1 to 10:1 reduction ratios
- **Snapshot Technology:**
  - Instant, space-efficient snapshots
  - Up to 512 snapshots per volume
  - Snapshot-based cloning and recovery
- **Replication Services:**
  - Asynchronous replication between arrays
  - ActiveCluster synchronous replication
  - Cloud replication to Pure Cloud Block Store

### Teacher Notes:
- Emphasize that data reduction is always enabled
- Show real-world data reduction examples
- Explain snapshot technology benefits for backup/recovery
- Discuss replication options for different RPO/RTO requirements

---

## **SLIDE 8: FlashArray//X Performance Characteristics**
### Content:
- **Performance Specifications (//X90):**
  - Up to 1.5M IOPS (4KB random read)
  - Up to 23GB/s throughput
  - Sub-500 microsecond latency
  - Consistent performance regardless of capacity utilization
- **Performance Optimization:**
  - Automatic workload balancing
  - Quality of Service (QoS) controls
  - Performance monitoring and analytics

### Teacher Notes:
- Show performance specification sheets for different models
- Explain how Pure maintains consistent performance
- Discuss QoS implementation and use cases
- Demonstrate performance monitoring tools if available

---

## **SLIDE 9: FlashArray//X Connectivity & Protocols**
### Content:
- **Host Connectivity:**
  - Fibre Channel: 32Gb, 16Gb
  - iSCSI: 25GbE, 10GbE
  - NVMe-oF: RDMA, TCP
- **Supported Protocols:**
  - Block protocols only (no file/object)
  - SCSI command set
  - Multipathing support (MPIO, ALUA)
- **Integration:**
  - VMware vSphere integration
  - Kubernetes CSI driver
  - Hypervisor plugins and management tools

### Teacher Notes:
- Show physical connectivity options on actual hardware
- Explain protocol selection criteria for different environments
- Discuss multipathing configuration and benefits
- Demonstrate integration tools and plugins

---

## **SLIDE 10: FlashBlade//S Overview**
### Content:
- **Architecture Type:** Scale-out unified storage platform
- **Target Workloads:**
  - Modern analytics and AI/ML
  - Content repositories and file shares
  - Backup and archive consolidation
  - Unstructured data growth
- **Supported Protocols:**
  - NFS v3/v4.1 (file)
  - S3 (object)
  - Native scale-out architecture

### Teacher Notes:
- Position FlashBlade as complementary to FlashArray
- Explain unified file and object storage benefits
- Show use cases for modern data pipelines
- Discuss scale-out advantages for unstructured data

---

## **SLIDE 11: FlashBlade//S Scale-Out Architecture**
### Content:
- **Blade Design:**
  - Independent compute and storage blades
  - Minimum 7 blades, scale to 150+ blades
  - Linear performance and capacity scaling
- **Network Fabric:**
  - High-speed interconnect between blades
  - Built-in load balancing and failover
  - No single points of failure
- **Distributed Services:**
  - Distributed metadata management
  - Erasure coding for protection
  - Global namespace and load balancing

### Teacher Notes:
- Draw scale-out architecture diagrams
- Explain how blades are added for scaling
- Show fault tolerance and load distribution
- Compare to traditional scale-up NAS architectures

---

## **SLIDE 12: FlashBlade//S Hardware Components**
### Content:
- **Chassis:**
  - 4U chassis housing multiple blades
  - Hot-swappable blade design
  - Redundant power and cooling
- **Blade Types:**
  - **Fabric Modules (FMs):** Network switching and routing
  - **Storage Blades:** Flash storage and compute
- **Blade Specifications:**
  - 52TB raw capacity per blade
  - Built-in compute and networking
  - NVMe flash storage

### Teacher Notes:
- Show physical blade components if available
- Explain blade insertion and removal procedures
- Discuss power and cooling requirements
- Show chassis management and monitoring

---

## **SLIDE 13: FlashBlade//S Data Services**
### Content:
- **File Services:**
  - NFSv3 and NFSv4.1 support
  - POSIX-compliant file system
  - Advanced file features (quotas, snapshots)
- **Object Services:**
  - S3-compatible object storage
  - Versioning and lifecycle management
  - Multi-protocol access (file and object to same data)
- **Data Protection:**
  - Erasure coding (EC 8+2, 16+4)
  - Snapshots and replication
  - Built-in data reduction

### Teacher Notes:
- Explain multi-protocol access benefits
- Show how erasure coding provides protection
- Discuss snapshot and replication capabilities
- Compare to traditional NAS data protection methods

---

## **SLIDE 14: Purity Operating Environment**
### Content:
- **Core Components:**
  - Purity//FA (FlashArray OS)
  - Purity//FB (FlashBlade OS) 
  - Pure1 cloud management platform
- **Key Features:**
  - Simple, intuitive management interface
  - Predictive analytics and support
  - Non-disruptive upgrades (NDU)
  - API-first architecture

### Teacher Notes:
- Show Purity GUI interface if possible
- Emphasize simplicity compared to traditional storage
- Explain predictive support capabilities
- Discuss API-first design for automation

---

## **SLIDE 15: Purity Management Interface**
### Content:
- **Web-Based GUI:**
  - Modern, responsive design
  - Role-based access control
  - Integrated monitoring and alerting
- **Dashboard Features:**
  - Real-time performance metrics
  - Capacity utilization and trending
  - Health and status monitoring
  - Quick access to common tasks
- **Workflow Automation:**
  - Volume provisioning wizards
  - Protection policy configuration
  - Automated capacity management

### Teacher Notes:
- Give live demo of Purity interface if available
- Show dashboard customization options
- Walk through common administrative tasks
- Emphasize ease of use compared to competitors

---

## **SLIDE 16: Pure1 Cloud Management Platform**
### Content:
- **Cloud-Based Management:**
  - Centralized management of all Pure arrays
  - Predictive analytics and AI-driven insights
  - Proactive support and issue resolution
- **Key Capabilities:**
  - **Meta:** Multi-array analytics and reporting
  - **Risk:** Predictive issue identification
  - **Support:** Automated case creation and resolution
  - **Plan:** Capacity planning and optimization

### Teacher Notes:
- Show Pure1 interface and capabilities
- Explain predictive analytics value proposition
- Discuss privacy and security considerations
- Show real examples of proactive support

---

## **SLIDE 17: Purity Data Reduction**
### Content:
- **Always-On Efficiency:**
  - Inline global deduplication
  - Compression (LZ4, zlib)
  - Pattern removal (zero-fill detection)
  - Thin provisioning
- **Global Deduplication:**
  - Deduplication across all volumes
  - Variable-length deduplication
  - Metadata efficiency
- **Typical Ratios:**
  - Databases: 2:1 to 5:1
  - Virtual machines: 10:1 to 20:1
  - General files: 3:1 to 8:1

### Teacher Notes:
- Explain why Pure chose always-on approach
- Show real customer data reduction examples
- Compare to competitive approaches (post-process)
- Discuss data reduction guarantee program

---

## **SLIDE 18: Pure Storage APIs & Automation**
### Content:
- **REST API:**
  - Complete functionality exposure
  - Comprehensive documentation
  - SDKs for multiple languages (Python, PowerShell, Go)
- **Integration Ecosystem:**
  - Ansible modules
  - Terraform providers
  - VMware vRealize integration
  - ServiceNow connectors
- **DevOps Integration:**
  - Infrastructure as Code support
  - CI/CD pipeline integration
  - Container orchestration plugins

### Teacher Notes:
- Show API documentation and examples
- Demonstrate automation tools and integrations
- Discuss Infrastructure as Code benefits
- Give examples of customer automation use cases

---

## **SLIDE 19: Pure Storage Reliability & Support**
### Content:
- **Reliability Features:**
  - 99.9999% uptime guarantee
  - Non-disruptive upgrades and maintenance
  - Predictive component replacement
- **Support Model:**
  - Proactive support through Pure1
  - Evergreen storage (no forklift upgrades)
  - 24/7 global support organization
- **Evergreen Architecture:**
  - Non-disruptive controller upgrades
  - Capacity expansion without downtime
  - Technology refresh without migration

### Teacher Notes:
- Explain Evergreen value proposition and customer benefits
- Show support statistics and customer satisfaction scores
- Discuss non-disruptive upgrade procedures
- Compare to traditional storage refresh cycles

---

## **SLIDE 20: Day 3 Lab Exercises**
### Content:
- **Lab 3.1:** FlashArray Management (30 minutes)
  - Connect to Purity interface
  - Create volumes and protection groups
  - Configure host connectivity
- **Lab 3.2:** Data Reduction Analysis (15 minutes)
  - Monitor data reduction ratios
  - Analyze space utilization
- **Lab 3.3:** Pure1 Platform Tour (15 minutes)
  - Explore Pure1 management features
  - Review analytics and recommendations

### Teacher Notes:
- Ensure lab environment has Pure Storage simulator or demo environment
- Guide students through key management tasks
- Help interpret data reduction and performance metrics
- Show Pure1 features and capabilities

---

## **SLIDE 21: Day 3 Summary & Day 4 Preview**
### Content:
- **Today's Key Learning:**
  - FlashArray//X scale-up block storage architecture
  - FlashBlade//S scale-out unified storage
  - Purity Operating Environment capabilities
  - Pure1 cloud management platform
- **Tomorrow's Focus:**
  - Enterprise data protection strategies
  - Backup, recovery, and business continuity
  - Snapshot technologies and replication
  - Disaster recovery planning and implementation

### Teacher Notes:
- Summarize Pure Storage architectural advantages
- Reinforce key differentiators vs. traditional storage
- Preview data protection topics for Day 4
- Answer any remaining Pure Storage questions

---

## **Additional Teaching Materials:**

### **Hands-on Lab Exercises:**

#### **Lab 3.1: FlashArray Management (30 minutes)**
**Objectives:**
- Navigate Purity//FA management interface
- Create and manage storage volumes
- Configure host connectivity and multipathing

**Steps:**
1. Log into Purity interface (demo environment)
2. Explore dashboard and monitoring views
3. Create new volumes with different sizes and protection policies
4. Configure host groups and connectivity
5. Review performance metrics and capacity utilization

#### **Lab 3.2: Data Reduction Analysis (15 minutes)**
**Objectives:**
- Understand Pure Storage data reduction technologies
- Analyze real data reduction ratios
- Monitor space efficiency over time

**Steps:**
1. Review data reduction dashboard in Purity
2. Analyze different workload data reduction ratios
3. Create snapshots and observe space efficiency
4. Compare logical vs. physical capacity usage

#### **Lab 3.3: Pure1 Platform Tour (15 minutes)**
**Objectives:**
- Explore Pure1 cloud management platform
- Review predictive analytics capabilities
- Understand proactive support features

**Steps:**
1. Access Pure1 demo environment
2. Review multi-array dashboard and analytics
3. Explore capacity planning and optimization tools
4. Examine support case management features

### **Assessment Questions:**
1. What are the key advantages of Pure's DirectFlash technology over standard SSDs?
2. How does FlashBlade's scale-out architecture differ from traditional NAS systems?
3. Explain Pure Storage's Evergreen architecture and its business benefits
4. What role does Pure1 play in the overall Pure Storage ecosystem?

### **Case Studies:**
- Enterprise database consolidation on FlashArray
- Media and entertainment workflow on FlashBlade  
- Healthcare data analytics platform implementation
- Financial services disaster recovery with ActiveCluster

---

**Total Day 3 Duration: 8 hours**
- Lectures: 6.5 hours
- Labs: 1 hour
- Breaks and discussions: 30 minutes