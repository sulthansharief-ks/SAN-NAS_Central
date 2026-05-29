# 🏛️ Hitachi NAS (HNAS) Architecture & Terminology Guide

Understanding Hitachi Network Attached Storage (HNAS) requires thinking in layers. HNAS acts as a high-performance gateway between block storage (the backend SAN) and file storage (the frontend network). 

Here is the detailed architectural breakdown of HNAS terminologies, structured from the physical backend disks up to the logical network shares.


```mermaid
graph TD
    %% Styling Definitions
    classDef network fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b
    classDef virtual fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c
    classDef logical fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#1b5e20
    classDef physical fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#e65100
    classDef backend fill:#eceff1,stroke:#37474f,stroke-width:2px,color:#37474f

    subgraph Network_Layer [1. Network Layer]
        Users([End Users / Clients]):::network
        GNS([Global Namespace - GNS]):::network
    end

    subgraph Server_Layer [2. Virtual Server Layer]
        EVS1[Enterprise Virtual Server - Node 1]:::virtual
        EVS2[Enterprise Virtual Server - Node 2]:::virtual
    end

    subgraph FS_Layer [3. File System Layer]
        FS1[(File System 1)]:::logical
        FS2[(File System 2)]:::logical
        ViVol1[/Virtual Volume A/]:::logical
        ViVol2[/Virtual Volume B/]:::logical
    end

    subgraph Pool_Layer [4. Storage Pool Layer]
        Span{Span - Aggregated Storage Pool}:::physical
        SD1[System Drive 1]:::physical
        SD2[System Drive 2]:::physical
        SD3[System Drive 3]:::physical
    end

    subgraph SAN_Layer [5. SAN Backend Layer]
        LUN1[(VSP Block LUN 1)]:::backend
        LUN2[(VSP Block LUN 2)]:::backend
        LUN3[(VSP Block LUN 3)]:::backend
    end

    %% Routing and Connections
    Users ==>|SMB / NFS Requests| GNS
    Users -.->|Direct Mount to IP| EVS1
    
    GNS -->|Routes to| EVS1
    GNS -->|Routes to| EVS2

    EVS1 ==>|Owns & Mounts| FS1
    EVS2 ==>|Owns & Mounts| FS2

    FS1 -->|Contains| ViVol1
    FS1 -->|Contains| ViVol2

    FS1 ==>|Carved out of| Span
    FS2 ==>|Carved out of| Span

    Span -->|Stripes Chunks Across| SD1
    Span -->|Stripes Chunks Across| SD2
    Span -->|Stripes Chunks Across| SD3

    SD1 ===|FC Mapping| LUN1
    SD2 ===|FC Mapping| LUN2
    SD3 ===|FC Mapping| LUN3
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
