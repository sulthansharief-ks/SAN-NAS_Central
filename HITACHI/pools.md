# 🏗️ Standard Operating Procedure: DP Pool Creation & Capacity Management
**System:** Hitachi VSP Series  
**Management Tool:** Hitachi Device Manager - Storage Navigator (HDvM-SN)  

---

## **Table of Contents**
* [1. Purpose 🎯](#1-purpose)
* [2. Phase 1: Creating Parity Groups (Physical RAID) 💽](#2-phase-1-creating-parity-groups)
* [3. Phase 2: Creating Pool Volumes (LDEVs) 📦](#3-phase-2-creating-pool-volumes)
* [4. Phase 3: Creating the Dynamic Provisioning (DP) Pool 🏊‍♂️](#4-phase-3-creating-dp-pool)
* [5. Post-Creation Verification ✅](#5-post-creation-verification)
* [6. Understanding DP Pool Capacity Metrics 📊](#6-understanding-capacity-metrics)
* [7. Best Practices for Pool Parameters & Limits 🌟](#7-best-practices)

---

## <a id="1-purpose"></a>**1. Purpose 🎯**
To provide storage capacity to hosts using Hitachi Dynamic Provisioning (HDP), an administrator must build the physical and logical backend hierarchy. This involves grouping raw disks into **Parity Groups**, carving those into **LDEVs (Pool Volumes)**, and finally aggregating them into a **Dynamic Provisioning (DP) Pool**. Once the pool is created, understanding how capacity is measured and consumed is critical for ongoing storage management.

---

## <a id="2-phase-1-creating-parity-groups"></a>**2. Phase 1: Creating Parity Groups (Physical RAID) 💽**
*Parity Groups group raw physical drives together and apply a RAID protection level.*

1. In Storage Navigator, navigate to **Storage Systems** > **Parity Groups**.
2. Click the **+ Create Parity Groups** button (or the plus sign).
3. Choose your creation method:
   * **Basic Option:** The system uses best practices automatically. You simply select the disk type, change the RAID level (e.g., RAID-5 or RAID-6), and specify the number of parity groups.
   * **Advanced Option:** Allows you to manually configure the exact RAID layout by selecting the specific individual disks to assign to the parity group.
4. *Note:* Parity Group creation is restricted on some enterprise arrays (e.g., VSP 5000 series, G1000, F1500) and must be performed by an authorized Hitachi Service Representative, though you can still initialize them in the software.
5. Click **Finish** and **Apply**. Creating parity groups automatically creates the underlying LDEVs that can be consumed for pool creation.

---

## <a id="3-phase-2-creating-pool-volumes"></a>**3. Phase 2: Creating Pool Volumes (LDEVs) 📦**
*Before adding capacity to a pool, the Parity Groups must be chopped into Logical Devices (LDEVs) that act as "Pool-VOLs".*

1. Navigate to **Logical Devices**.
2. Click **Create LDEVs**.
3. Select the target Parity Group.
4. **LDEV Capacity:** Set the size for the Pool-VOLs. A recommended best practice for pool volume capacity is **2.99 TB**, as this is typically the maximum optimal capacity for a single pool-VOL. 
5. Calculate and create as many LDEVs as possible from the parity group to fill its physical capacity.
6. Click **Finish** and **Apply**.
7. **⚠️ Formatting:** You must format these new LDEVs. Select them, click **Format LDEVs**, and wait for the status to change to `Normal`. Quick format cannot be performed if you are using specific accelerated compression features.

---

## <a id="4-phase-3-creating-dp-pool"></a>**4. Phase 3: Creating the Dynamic Provisioning (DP) Pool 🏊‍♂️**
*This step aggregates the formatted Pool-VOLs into a single large reservoir of thin-provisioned capacity.*

1. In the left navigation tree, select **Pools** and click **Create Pools**.
2. **Pool Type:** Select **Dynamic Provisioning** from the dropdown.
3. **System Type:** Select **Open** (unless you are attaching mainframe systems).
4. **Multi-Tier Pool:** Select **Disable** (enabling this creates an HDT/Tiering pool instead).
5. **Pool Name:** Enter an alphanumeric prefix for the pool name (up to 32 characters including the initial number).
6. Click **Select Pool VOLs**.
7. A window showing available pool volumes will appear. Select the LDEVs you created in Phase 2 and click **Add** to move them into the *Selected Pool Volumes* table.
   * *Critical Rule:* Add all LDEVs that were created from a single parity group to the same pool.
   * *Critical Rule:* The Cache Mode for all selected volumes must be identical (either all enabled or all disabled).
8. **System Area Designation:** The array will automatically place its internal "System Area" (metadata) on one of these Pool-VOLs. The array prioritizes this placement based on drive type (Priority 1: 7.2 krpm HDD, Priority 2: 10 krpm HDD, Priority 3: 15 krpm HDD, Priority 4: SSD, Priority 5: External volume).
9. Click **OK** to return to the pool creation screen.
10. **Set Thresholds:**
    * **Warning Threshold:** Set between 1% - 100%. The default value is **70%**.
    * **Depletion Threshold:** Set between 1% - 100%. The default value is **80%**, and it *must* be higher than the Warning Threshold.
11. Click **Finish**, review the summary in the Confirm window, and click **Apply**.

---

## <a id="5-post-creation-verification"></a>**5. Post-Creation Verification ✅**
* Once applied, monitor the task list. 
* Navigate back to **Pools** and verify that your new DP Pool shows a status of `Normal` or `Formatting`. 
* Once formatting completes, you are ready to carve Virtual Volumes (V-VOLs) from this pool and map them to your host servers!

---

## <a id="6-understanding-capacity-metrics"></a>**6. Understanding DP Pool Capacity Metrics 📊**
Because HDP "lies" to the host servers about how much space is available (thin provisioning), Hitachi uses specific terminologies to differentiate between the physical hard drives and the logical illusions presented to the OS.

### **A. Total Pool Capacity (The Vault)**
* **What it is:** The actual, physical usable space across all the disks (Parity Groups) assigned to your DP Pool.
* **The Rule:** This is the hard physical limit of the hardware (e.g., 8TB of usable RAID space). 

### **B. Subscribed Capacity (The Credit Limit) 💳**
* **What it is:** The sum total of the *logical sizes* of all Virtual Volumes (V-VOLs) you have created and presented to the hosts. 
* **The Math:** If you have an 8TB physical pool and present 10TB of LUNs to VMware, you have a **125% Subscription Rate**. Thin provisioning allows and encourages this over-allocation.

### **C. Allocated Capacity / Pool Used Capacity (The Actual Debt) 📉**
* **What it is:** The total amount of physical space (measured in 42MB Pages) that the Hitachi controller has actually grabbed from the pool and permanently assigned to a V-VOL because a host wrote data to it.
* **The Rule:** This metric is what triggers the **Warning** and **Depletion thresholds** configured in Step 4.10. If Allocated Capacity hits 100% of the Total Pool Capacity, the array will freeze incoming I/O to protect metadata.

### **D. Host Used Capacity (The OS Perspective) 💻**
* **What it is:** The amount of space the Host Operating System (e.g., Windows, ESXi) *thinks* it is using. 
* **The Disconnect:** If a Windows admin deletes a 500GB file, the Hitachi array does not automatically know. The 500GB remains "Allocated" on the VSP until you run a **SCSI UNMAP** command from the host OS to tell the storage array to reclaim those abandoned 42MB pages.

---

## <a id="7-best-practices"></a>**7. Best Practices for Pool Parameters & Limits 🌟**
To ensure high availability, prevent out-of-space outages, and maintain performance, always adhere to the following field engineering best practices when configuring DP Pools:

### **A. Subscription Limit Rate**
* **The Concept:** While thin provisioning is powerful, unchecked over-provisioning can lead to a disastrous storage depletion event if multiple hosts suddenly write massive amounts of data.
* **Best Practice:** Never leave the Subscription Limit set to "Limitless" unless heavily monitoring via Ops Center. 
  * **Standard Virtualization (VMware/Hyper-V):** Set the subscription limit to **150% - 200%**. This provides excellent capacity savings while creating a hard stop to prevent rogue admins from creating phantom LUNs indefinitely.
  * **Database Workloads (SQL/Oracle):** Set the subscription limit much closer to **100% - 125%**. Databases often format or fill their entire allocated capacity, making high over-subscription extremely risky.

### **B. Tuning the Warning & Depletion Thresholds**
* **Warning Threshold:** The system default is 70%. Best practice is to set it between **70% - 75%**. This alerts the storage team early enough to initiate a hardware procurement and racking cycle (which often takes 30-90 days) before the physical disks run out of space.
* **Depletion Threshold:** The system default is 80%. Best practice is to set it to **85% - 90%**. 
  * *⚠️ Danger:* If the physical pool hits the depletion threshold, the array will lock the pool and reject incoming host writes to prevent the internal metadata map from being corrupted. Never set this close to 100%.

### **C. System Area Placement**
* **The Concept:** The array automatically reserves a portion of a single Pool-VOL for the pool's "System Area" (the map of exactly which 42MB pages belong to which V-VOLs).
* **Best Practice:** The controllers always place the System Area on the highest-performing drive type present when the pool is created. Always ensure your fastest drives (e.g., NVMe SSDs or FMDs) are included in the pool *during initial creation*, guaranteeing the System Area lands on flash storage. This prevents metadata lookups from becoming a bottleneck.

### **D. LDEV (Pool-VOL) Sizing & Consistency**
* **The Concept:** The DP Pool stripes data randomly across the underlying Pool-VOLs.
* **Best Practice:** Keep all Pool-VOLs within a single DP Pool exactly the same size (e.g., maintaining the 2.99 TB standard across the board). Mixing drastically different LDEV sizes (e.g., dropping a 500GB Pool-VOL into a pool of 3TB Pool-VOLs) forces the array's wide-striping algorithm to work harder and can create uneven I/O distribution and physical hot spots on the backend SAS loops.
