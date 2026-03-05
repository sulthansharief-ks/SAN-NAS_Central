# 🏗️ NetApp ONTAP: The Definitive Aggregate Guide 🚀

This guide covers the entire lifecycle of a NetApp Aggregate: from identifying spare disks and calculating RAID groups to creation, daily maintenance, and safe decommissioning.
## The Aggregate provides the physical storage blocks, and WAFL manages the logical organization of those blocks.

<!-- ![Aggregate Architecture](/media/Gemini_Generated_Image_xzc9ybxzc9ybxzc9.png) -->

```mermaid
graph LR
    %% --- Dark Mode & "Animation" Theme Definitions ---
    classDef base fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    classDef glowBlue fill:none,stroke:#00b8ff,stroke-width:3px,shadow:0 0 15px #00b8ff,color:#00b8ff
    classDef glowOrange fill:none,stroke:#ff6a00,stroke-width:3px,shadow:0 0 15px #ff6a00,color:#ff6a00
    classDef wafl fill:#001e26,stroke:#00b8ff,stroke-width:2px,color:#00b8ff,stroke-dasharray: 5 5
    classDef container fill:#161b22,stroke:#8b949e,stroke-width:2px,color:#fff
    classDef disk fill:#21262d,stroke:#8b949e,stroke-width:2px,color:#fff
    classDef parity fill:#21262d,stroke:#ff6a00,stroke-width:2px,color:#ff6a00

    %% --- Main Nodes & Structure ---
    WAFL(WAFL Filesystem Access)

    %% Main aggregate flows Left-to-Right
    subgraph AGGR_TOP [AGGREGATE e.g., aggr1_ssd]
        direction LR
        
        %% RAID Groups stack disks Top-to-Bottom internally
        subgraph RG0 [RAID GROUP 0 RAID-DP]
            direction TB
            D1[(DISK 1<br/>Data)]
            D2[(DISK 2<br/>Data)]
            P3[(DISK 3<br/>Parity)]
            DP4[(DISK 4<br/>D-Parity)]
        end
        
        subgraph RG1 [RAID GROUP 1 RAID-DP]
            direction TB
            D5[(DISK 5<br/>Data)]
            D6[(DISK 6<br/>Data)]
            P7[(DISK 7<br/>Parity)]
            DP8[(DISK 8<br/>D-Parity)]
        end
    end

    subgraph PSP [Physical Storage Pool]
        direction TB
        S1[SSD Shelf 1]
        S2[SSD Shelf 2]
    end

    %% --- Connections ---
    WAFL ==> AGGR_TOP
    AGGR_TOP <==> RG0
    AGGR_TOP <==> RG1
    RG0 <==> PSP
    RG1 <==> PSP

    %% --- Side Labels ---
    L_Side1[RAID Level: RAID-DP] --- RG0
    RG1 --- L_Side2[RAID Level: RAID-DP]

    %% --- Apply Styles ---
    class WAFL wafl
    class AGGR_TOP glowBlue
    class RG0,RG1 glowOrange
    class PSP container
    class D1,D2,D5,D6 disk
    class P3,DP4,P7,DP8 parity
    class S1,S2,L_Side1,L_Side2 base

    %% --- Link Styles (Neon Glow) ---
    linkStyle 0 stroke:#00b8ff,stroke-width:4px,stroke-dasharray: 10 5
    linkStyle 1,2 stroke:#00b8ff,stroke-width:3px
    linkStyle 3,4 stroke:#ff6a00,stroke-width:3px
    linkStyle 5,6 stroke:#ff6a00,stroke-width:1px,stroke-dasharray: 2 2 %% Dashed lines for labels
```
---

## 📋 Table of Contents
1. [Phase 1: Preparation & Planning](#phase-1-preparation--planning)
2. [Phase 2: Creating the Aggregate](#phase-2-creating-the-aggregate)
3. [Phase 3: Verification & Health Checks](#phase-3-verification--health-checks)
4. [Phase 4: Expansion (Adding Disks)](#phase-4-expansion-adding-disks)
5. [Phase 5: Maintenance & Operations](#phase-5-maintenance--operations)
6. [Phase 6: Safe Deletion](#phase-6-safe-deletion)

---

## 🧐 Phase 1: Preparation & Planning
*Before building, you must identify your available resources (bricks).*

### 1. Identify Spare Disks
You need "Spare" disks to build an aggregate. Partitioned disks (ADP) may show as "shared".

```bash
# Show all spare disks in the cluster
storage disk show -container-type spare

# Show spares sorted by type (SSD vs SAS) and usable size
storage disk show -container-type spare -fields type,usable-size,checksum-compatibility

# Check if you have enough spares for a specific RAID type (e.g., RAID-DP requires 2 parity disks)
storage disk show -count
```

### 2. Simulate the Creation (Dry Run)
*Highly Recommended.* This tells you exactly how much usable space you will get after RAID overhead, without actually creating anything.

```bash
# Simulate creating a 20-disk aggregate
storage aggregate create -aggregate aggr_test -diskcount 20 -simulate

# Check the output for "Total Usable Space" vs "Total Physical Space"
```

---

## 🔨 Phase 2: Creating the Aggregate
*Constructing the storage pool. Choose the method that fits your requirements.*

### Option A: Automatic Selection (The "Easy Button")
Let ONTAP choose the best disks and RAID group sizing. Best for standard deployments.

```bash
# Create an SSD aggregate with 24 disks using RAID-DP
storage aggregate create -aggregate aggr1_ssd -diskcount 24 -raidtype raid_dp

# Create an aggregate on a specific node
storage aggregate create -aggregate aggr1_node1 -node <Node_Name> -diskcount 24
```

### Option B: Manual Selection (The "Control Freak")
Use this if you need to pick specific disks (e.g., from a specific shelf or loop).

```bash
# Create using a specific list of disks
storage aggregate create -aggregate aggr2_manual -disklist 1.10.0,1.10.1,1.10.2,1.10.3 -raidtype raid_dp
```

### Option C: Mirrored Aggregate (SyncMirror)
Required for MetroCluster or high-redundancy setups. Requires two distinct pools of disks.

```bash
# Create a mirrored aggregate (requires equal spares on both 'plexes')
storage aggregate create -aggregate aggr3_mirrored -diskcount 20 -mirror true
```

---

## 👀 Phase 3: Verification & Health Checks
*Inspect the foundation immediately after building.*

```bash
# 1. Verify Status (Target: State=online, Status=normal)
storage aggregate show

# 2. Check RAID Layout
# Verify which disks are Data, Parity, or D-Parity
storage aggregate show-status

# 3. Check Space Efficiency
# Ensure you aren't already hitting space warnings (should be 0% used)
storage aggregate show-space
```

---

## 📈 Phase 4: Expansion (Adding Disks)
*Running out of space? Add more disks to the pool.*

> **⚠️ Warning:** You generally **cannot remove** disks once added. Expansion is permanent until you destroy the aggregate.

```bash
# Add 12 spare disks to an existing aggregate
# ONTAP will append them to existing RAID groups or create a new RAID group if needed.
storage aggregate add-disks -aggregate aggr1_ssd -diskcount 12

# (Optional) Manually specify which disks to add
storage aggregate add-disks -aggregate aggr1_ssd -disklist 2.10.0,2.10.1
```

---

## 🔧 Phase 5: Maintenance & Operations
*Day-to-day tasks to keep the aggregate healthy.*

### 1. Renaming
```bash
# Rename to match your naming convention
storage aggregate rename -aggregate aggr1_ssd -newname AGGR_Flash_Prod_01
```

### 2. Scrubbing (RAID Health)
RAID scrubbing reads disk sectors to find latent media errors and correct parity.

```bash
# Manually start a scrub
storage aggregate scrub start -aggregate AGGR_Flash_Prod_01

# Check the status of a running scrub
storage aggregate scrub show
```

### 3. Relocation (HA Failover Test)
Move the aggregate to the HA partner node (only for un-mirrored aggregates).

```bash
# Move aggregate to the other controller
storage aggregate relocation start -aggregate AGGR_Flash_Prod_01 -destination <Partner_Node>
```

### 4. Space Monitoring
```bash
# Show aggregates that are dangerously full (>90%)
storage aggregate show-space -percent-used >90
```

---

## 🧨 Phase 6: Safe Deletion
*Decommissioning an aggregate. This is destructive and irreversible.*

### Step 1: Evacuate Data
You cannot delete an aggregate that contains volumes.

```bash
# Check for existing volumes
volume show -aggregate AGGR_Flash_Prod_01

# Move volumes to another aggregate (if needed)
volume move start -vserver <SVM> -volume <Vol_Name> -destination-aggregate <Other_Aggr>
```

### Step 2: Destroy
```bash
# 1. Take the aggregate offline
storage aggregate offline -aggregate AGGR_Flash_Prod_01

# 2. Delete the aggregate
# This wipes the RAID config and returns disks to the 'Spare' pool.
storage aggregate delete -aggregate AGGR_Flash_Prod_01
```

---
*Sulthan Sharief K S*
