# 🏗️ SOP: Expanding a Brocade SAN Fabric (Adding a New Switch)

Merging a new Brocade switch into an existing production fabric is a critical operation. If done incorrectly, it can cause fabric segmentation or, worst-case, overwrite the production zoning database. 

This document is written to enterprise and Broadcom/OEM best-practice standards. The golden rule is: **Always strip the zoning database from the new switch before connecting it to the production fabric.**

## 📑 Table of Contents
* [⚠️ Critical Prerequisites](#-critical-prerequisites)
* [🛠️ Phase 1: Isolate and Prepare the New Switch](#-phase-1-isolate-and-prepare-the-new-switch)
* [🧹 Phase 2: Purge the Zoning Database (Crucial)](#-phase-2-purge-the-zoning-database-crucial)
* [🔌 Phase 3: Physical Connection and Merge](#-phase-3-physical-connection-and-merge)
* [✅ Phase 4: Post-Merge Verification](#-phase-4-post-merge-verification)
* [🧠 Appendix A: Fabric Timers & Principal Priority Explained](#-appendix-a-fabric-timers--principal-priority-explained)

---

### ⚠️ Critical Prerequisites
Before touching any cables, verify the following:
1. **💾 Firmware Compatibility:** Ensure the new switch's Fabric OS (FOS) version is compatible with the existing fabric. (Refer to the Broadcom FOS Target Path and Compatibility Matrix).
2. **🆔 Unique Domain ID:** You must assign a Domain ID to the new switch that is **not** currently in use by any switch in the `fabricshow` output of the existing fabric.
3. **⏱️ Fabric Timers:** The `E_D_TOV` and `R_A_TOV` timers must be identical across all switches.
4. **📜 Licenses:** Ensure the new switch has the ISL Trunking or Extended Fabric licenses if your architecture requires them.

---

### 🛠️ Phase 1: Isolate and Prepare the New Switch
Perform these steps on the **NEW** switch while it is completely disconnected from the production SAN.

**1. 🛑 Disable the new switch:**
This prevents any accidental traffic or merging while you change core parameters.
    
    switchdisable

**2. 🔢 Set a Unique Domain ID:**
Run the configure command to change the Fabric parameters.
    
    configure

* Select `y` for `Fabric parameters`.
* Enter your newly planned, unique **Domain ID**.
* Leave all other parameters (like timers) at their defaults unless your existing fabric explicitly uses non-standard timers.

**3. 👑 Verify Fabric Principal Priority:**
Ensure the new switch does not try to take over as the Principal Switch. Keep its priority at the default (255) or higher than your existing core switches.
    
    fabricprincipal --show

*(If you need to change it, use `fabricprincipal --priority 255`).*

---

### 🧹 Phase 2: Purge the Zoning Database (Crucial)
To ensure the new switch gracefully downloads the zoning database from your existing Principal switch, you must completely wipe its local database.

**1. 🗑️ Clear the active and defined configurations:**
    
    cfgdisable
    cfgclear

**2. 💾 Save the empty database:**
    
    cfgsave

*(⚠️ Note: Type `y` to confirm. The output of `cfgshow` should now be completely empty).*

**3. ⚖️ Match the Default Zone Policy:**
Ensure the default zoning behavior matches your production fabric (usually set to `All Access` or `No Access`).
    
    defzone --show

*(To change it to the best-practice "No Access", use `defzone --noaccess` followed by `cfgsave`).*

---

### 🔌 Phase 3: Physical Connection and Merge
Now that the new switch is a "blank slate" with a unique Domain ID, it is safe to merge.

**1. 🟢 Re-enable the new switch:**
    
    switchenable

**2. 🔗 Connect the ISL Cables:**
Physically connect the fiber cables from the ISL (E_Port) designated ports on the existing fabric to the ISL ports on the new switch. 

**3. 👀 Verify Port Initialization:**
Watch the port state transition to an **E_Port** (Expansion Port).
    
    portshow <ISL_port_number>

*(You should see the port state as `Online` and the port type as `E-Port`).*

---

### ✅ Phase 4: Post-Merge Verification
Verify the fabrics have merged into a single logical entity and the zoning database successfully replicated.

**1. 🕸️ Check the Fabric Topology:**
Run this on either the new switch or the existing switch. You should see both switches listed, confirming the new Domain ID is present.
    
    fabricshow

**2. 🚦 Verify ISL Status:**
Ensure the links are active and, if licensed, properly trunked.
    
    islshow
    trunkshow

**3. 🪞 Confirm Zoning Replication:**
Run `cfgshow` on the **NEW** switch. It should now display the exact same active configuration and aliases as your production fabric.
    
    cfgshow

**4. 🚨 Check for Segmentation or Errors:**
Review the event logs to ensure there are no zone conflict or fabric segmentation errors.
    
    errdump

If the `fabricshow` output displays all switches and `cfgshow` displays your production database, the merge was successful. The new switch is now ready for end-device (F_Port) connections.

---

### 🧠 Appendix A: Fabric Timers & Principal Priority Explained

#### ⏱️ What are the `E_D_TOV` and `R_A_TOV` Timers?
In a Fibre Channel SAN, these timers dictate the fundamental "patience" of the switch. If switches in the same fabric have different timers, one switch might give up on a data frame while the other is still waiting. This disagreement causes the switches to isolate from each other (Fabric Segmentation).

* **`E_D_TOV` (Error Detect Timeout Value):** The maximum time a switch will wait for an expected response before it assumes a frame was lost in transit. (Default is usually **2000 milliseconds**).
* **`R_A_TOV` (Resource Allocation Timeout Value):** The time a switch waits before freeing up internal resources (like routing tables or buffer credits) after an error is detected or a link drops. This ensures old frames are flushed from the fabric before new paths are calculated. (Default is usually **10000 milliseconds**).

**How to Check the Timers:**
You can check these non-disruptively without entering the configuration menu by using the `configshow` command and filtering the output.
    
    configshow | grep TOV

*(You will see outputs like `fabric.ops.e_d_tov: 2000` and `fabric.ops.r_a_tov: 10000`).*

#### 👑 Fabric Principal Priority Explained
In every Brocade fabric, one switch is elected as the **Principal Switch** (the "boss"). The Principal Switch is responsible for assigning Domain IDs to all other switches and maintaining the master routing topology. 

Switches elect the Principal based on their **Priority Setting**. 
* **The Rule:** The switch with the **lowest number wins** (1 is the highest possible priority, 255 is the lowest). 
* **The Tie-Breaker:** If all switches are set to the default (255), the switch with the lowest WWN (World Wide Name) MAC address automatically wins.

**Example Scenario:**
Imagine a SAN fabric with two heavy-duty core switches and several smaller edge switches:
* **Core Switch A (Priority 1):** You manually configure this to `1`. It is the absolute boss.
* **Core Switch B (Priority 2):** You configure this to `2`. If Switch A reboots, Switch B immediately takes over as the Principal.
* **Existing Edge Switches (Priority 255):** These just accept orders and Domain IDs from the Core.

**What happens when you add a NEW switch?**
If you unbox a new switch and accidentally set its priority to `1`, or if its default priority happens to be higher than your existing Principal, the new switch will initiate a hostile takeover. This causes a **Fabric Rebuild (Build Fabric / BF event)**. The entire SAN pauses, re-calculates all routes, and re-assigns Domain IDs. **This drops active I/O between your servers and storage arrays.**

By ensuring the new switch is set to the default `255`, it quietly joins the fabric, asks the existing Principal for a Domain ID, and merges without disrupting your production traffic. 

**How to Check Priority:**
    
    fabricprincipal --show

*(Output will tell you if the switch is the principal, and what its current priority is, e.g., `Priority: 255`).*
