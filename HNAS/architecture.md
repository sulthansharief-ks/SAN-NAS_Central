# 🏛️ Hitachi NAS (HNAS) Architecture & Terminology Guide

Understanding Hitachi Network Attached Storage (HNAS) requires thinking in layers. HNAS acts as a high-performance gateway between block storage (the backend SAN) and file storage (the frontend network). 

Here is the detailed architectural breakdown of HNAS terminologies, structured from the physical backend disks up to the logical network shares.


```mermaid
graph TD
    %% Styling Definitions
    classDef backend fill:#eceff1,stroke:#37474f,stroke-width:2px,color:#37474f
    classDef pool fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#e65100
    classDef logical fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#1b5e20
    classDef server fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c
    classDef network fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b

    subgraph Network_Layer [1. Client Access & Network Layer]
        Clients([SMB / NFS Clients]):::network
        GNS([Global Namespace - GNS]):::network
    end

    subgraph Server_Layer [2. Virtual Server Layer - Active/Active HNAS Nodes]
        EVS1[EVS 1: Finance_Server <br> 192.168.10.50]:::server
        EVS2[EVS 2: Eng_Server <br> 192.168.10.51]:::server
    end

    subgraph FS_Layer [3. File System & Quota Layer]
        FS1[(FS_Finance_Data)]:::logical
        FS2[(FS_HR_Data)]:::logical
        FS3[(FS_Engineering)]:::logical
        
        VV1[/ViVol: Payroll Quota/]:::logical
        VV2[/ViVol: Benefits Quota/]:::logical
    end

    subgraph Pool_Layer [4. Storage Pool Layer - Performance Tiers]
        Span1{Span 1 - Tier 1 SSD}:::pool
        Span2{Span 2 - Tier 2 SAS}:::pool
        
        SD1[System Drive 01]:::pool
        SD2[System Drive 02]:::pool
        SD3[System Drive 03]:::pool
        SD4[System Drive 04]:::pool
    end

    subgraph SAN_Layer [5. Backend VSP Block Storage]
        LUN1[(VSP LUN 0001)]:::backend
        LUN2[(VSP LUN 0002)]:::backend
        LUN3[(VSP LUN 0003)]:::backend
        LUN4[(VSP LUN 0004)]:::backend
    end

    %% Routing and Connections
    Clients ==>|UNC Path / Mount| GNS
    Clients -.->|Direct IP Mount| EVS1
    
    GNS -->|Namespace Routing| EVS1
    GNS -->|Namespace Routing| EVS2

    EVS1 ==>|Mounts & Serves| FS1
    EVS1 ==>|Mounts & Serves| FS2
    EVS2 ==>|Mounts & Serves| FS3

    FS2 -->|Applies Quota via| VV1
    FS2 -->|Applies Quota via| VV2

    FS1 -.->|Formats Capacity| Span1
    FS2 -.->|Formats Capacity| Span1
    FS3 -.->|Formats Capacity| Span2

    Span1 -->|Stripes Data| SD1
    Span1 -->|Stripes Data| SD2
    Span2 -->|Stripes Data| SD3
    Span2 -->|Stripes Data| SD4

    SD1 ===|FC Path| LUN1
    SD2 ===|FC Path| LUN2
    SD3 ===|FC Path| LUN3
    SD4 ===|FC Path| LUN4
```

---

## **1. Core Architectural Terminologies**

| Terminology | Abbreviation | Definition & Role |
| :--- | :--- | :--- |
| **System Drive** | SD | The foundational storage building block. An SD is simply a block-level LUN presented from the backend Hitachi VSP array to the HNAS gateway via Fibre Channel. |
| **Span** | - | A storage pool made by aggregating multiple System Drives. Spans stripe data across the underlying SDs for performance and provide a unified pool of raw capacity. |
| **Chunk** | - | The internal allocation unit of a Span (typically 17MB). When HNAS writes data, it allocates chunks from the Span to ensure data is evenly distributed across all SDs. |
| **File System** | FS | A logical, formatted container created *inside* a Span. HNAS uses an object-based file system (WFS) that supports thin provisioning, deduplication, and snapshots. |
| **Virtual Volume** | ViVol | A logical subdivision within a File System. ViVols act like directory quotas or sub-file systems, allowing you to apply specific snapshot policies or size limits to different departments sharing the same File System. |
| **Enterprise Virtual Server** | EVS | A logical, virtualized NAS server. An EVS acts as an independent file server with its own IP addresses, routing tables, active directory computer account, and CIFS/NFS configurations. |
| **Global Namespace** | GNS | A feature that abstracts physical server names, allowing you to present a single, unified directory tree to users, even if the folders physically reside across multiple different EVSs or File Systems. |

---

## **2. How They Are Connected (The Bottom-Up Flow)**

To understand how these components interact, follow the data path from the physical storage array up to the user's laptop.

1. **Provisioning the Backend (SAN Layer):** The storage administrator carves out block-level LUNs on the backend VSP array and maps them to the HNAS gateway's Fibre Channel ports.
2. **Creating the System Drives (SD):** HNAS scans its Fibre Channel ports, discovers these LUNs, and labels them as **System Drives**.
3. **Building the Span (Pool Layer):** You group these System Drives together to form a **Span**. This creates a large, high-performance pool of raw storage.
4. **Formatting the File System (FS Layer):** You carve out a **File System** from the Span's available capacity. You can create multiple File Systems within a single Span.
5. **Assigning to an EVS (Server Layer):** A File System cannot talk to the network on its own. It must be explicitly mounted to an **Enterprise Virtual Server (EVS)**. The EVS acts as the "owner" of that File System.
6. **Creating the Export (Network Layer):** Finally, you create an SMB/CIFS Share or an NFS Export on the File System. Because the File System is bound to the EVS, users access the share by connecting to the EVS's specific IP address.

---

## **3. The Role of the EVS in High Availability (Clustering)**

The **EVS** is the most critical concept for HNAS clustering and failover. 

HNAS gateways are typically deployed in active/active clusters (nodes). 
* An EVS is a logical entity that "floats" on top of the physical HNAS nodes.
* If physical Node 1 fails, the EVS (along with its IP addresses and all its attached File Systems) instantly migrates to physical Node 2.
* Because the users are mapped to the EVS IP address, they experience a brief pause, but the connection remains intact without needing to remap their network drives.

---

Are you currently setting up a new HNAS environment, or are you looking to troubleshoot an existing configuration?
