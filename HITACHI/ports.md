# 🏷️ Standard Operating Procedure: Understanding VSP G800 Port Numbering
**System:** Hitachi VSP G800  
**Context:** Physical and Logical Port Identification  



---

## **1. Purpose 🎯**
To design highly available SAN topologies and troubleshoot pathing issues on a Hitachi Virtual Storage Platform (VSP) G800, a Level 3 Storage Administrator must be able to instantly decode Hitachi's port naming convention. 

Unlike other vendors that use a `Controller:Slot:Port` format (e.g., `0:0:1`), Hitachi uses an alphanumeric "Cluster" designation.

---

## **2. The Baseline Convention: `CL[#]-[Letter]` 🔠**
Every physical front-end port on the VSP G800 (Fibre Channel or iSCSI) follows this strict format:

### **Part 1: `CL` (Cluster / Controller)**
* **Definition:** `CL` stands for **Cluster**, which is Hitachi's terminology for the storage controller node. 
* **VSP G800 Architecture:** The G800 is an Active/Active dual-controller system. Therefore, you will see two primary prefixes:
    * **`CL1`** = Controller 1 (Top canister / Cluster 1)
    * **`CL2`** = Controller 2 (Bottom canister / Cluster 2)

### **Part 2: `[Letter]` (Physical Port Identifier)**
* **Definition:** The letter represents the specific physical port on a Channel Board (CHB) installed in that controller.
* **Sequencing:** The VSP G800 can scale up to 80 front-end ports using host port expansion chassis. The letters are assigned sequentially as you move across the installed modules from left to right.
* **Example:** If you install a 4-port 16Gb/s Fibre Channel board into the first slot of Controller 1, those four ports will be labeled:
    * `CL1-A`
    * `CL1-B`
    * `CL1-C`
    * `CL1-D`
* The next module installed in Controller 1 will pick up the sequence at `CL1-E`, `CL1-F`, etc.

---

## **3. The Logical Extension: Host Groups (`CL[#]-[Letter]-[ID]`) 🔗**
When you look at port allocations in Storage Navigator or output from RAIDCOM CLI (e.g., `raidcom get ldev`), you will often see a third segment appended to the name.

### **Format: `CL1-A-0`**
* **`CL1-A`**: The physical port.
* **`-0`**: The **Host Group ID** assigned to that specific WWPN mapping.
* *Why this matters:* The VSP supports up to 2,048 Host Groups per storage system. A single physical port like `CL1-A` can host hundreds of logical Host Groups (e.g., `CL1-A-0`, `CL1-A-1`, `CL1-A-2`). When troubleshooting a specific host, you must identify both the physical port and the logical Host Group ID it is bound to.

---

## **4. Multipathing & Port Selection Best Practices 🛣️**
When provisioning a new server on the G800, you must spread the LUN paths across both clusters to ensure survival if a controller panics or requires an offline microcode update.

* **Rule of Thumb:** Never map a host strictly to `CL1` ports.
* **Standard Dual-Fabric Zoning:**
    * **Fabric A:** Zone Host HBA 1 to **`CL1-A`** (Controller 1)
    * **Fabric B:** Zone Host HBA 2 to **`CL2-A`** (Controller 2)
* **High-Throughput (4-Path) Zoning:**
    * Fabric A: `CL1-A` and `CL2-B`
    * Fabric B: `CL2-A` and `CL1-B`
    * *Note:* Mixing CL1 and CL2 across both fabrics ensures that even if you lose an entire SAN switch *and* a storage controller simultaneously, the host maintains a surviving path.

---
**Prepared by:** Storage Administration Team 🧑‍💼  
**Last Updated:** March 2026 📅

---


