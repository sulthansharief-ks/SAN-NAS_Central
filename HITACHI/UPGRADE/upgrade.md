# 🚀 Hitachi VSP Firmware (Microcode) Upgrade SOP: GUM & Component Phasing

This Standard Operating Procedure (SOP) details the phased approach for upgrading Hitachi VSP firmware, specifically isolating the **GUM (Gateway Unified Management)** layer before proceeding with the main controller and drive microcode. It also includes the critical exception handling for arrays presenting HDD warnings. ⚠️

---

## **Table of Contents 📑**
* [Phase 1: Comprehensive Pre-Upgrade Checks 📋](#phase-1-comprehensive-pre-upgrade-checks)
* [Phase 2: Firmware Upload & Preparation 📤](#phase-2-firmware-upload-preparation)
* [Phase 3: The Upgrade Execution (Standard Path) 🔀](#phase-3-the-upgrade-execution-standard-path)
* [Phase 3B: The Upgrade Execution (HDD Warning Exception Path) ⚠️](#phase-3b-the-upgrade-execution-hdd-warning-exception-path)
* [Phase 4: Post-Upgrade Validation ✅](#phase-4-post-upgrade-validation)

---

## <a id="phase-1-comprehensive-pre-upgrade-checks"></a>📋 Phase 1: Comprehensive Pre-Upgrade Checks
*Never initiate a microcode upgrade without verifying the array is perfectly healthy and hosts are fully redundant.*

1.  **Verify Hardware Health:** 🩺
    * **How:** Log into **Maintenance Utility**. Navigate to **Hardware** > **System GUI**.
    * **Look for:** All components (Controllers, FEDs, BEDs, Cache, Power) must show a 🟢 **Normal** status.
2.  **Check for Active SIMs (Alerts):** 🚨
    * **How:** Navigate to **Alerts / Events** > **SIMs**.
    * **Look for:** Ensure there are no unacknowledged or active Critical/Serious hardware SIMs. If there are, resolve them before proceeding.
3.  **Verify Host Multipathing (MPIO):** 🛣️
    * **How:** Instruct server admins to run the appropriate command for their OS (Linux) or check the MPIO control panel (Windows) / vCenter Storage Paths (ESXi).
      > multipath -ll
    * **Look for:** All LUNs must show multiple active/optimized paths to *both* Controller 1 and Controller 2.
4.  **Continuous Ping Monitoring:** 📡
    * **How:** Open a command prompt or terminal on your management jump-server.
    * **Action:** Start a continuous ping to both Controller 1 and Controller 2 management IPs. Leave this running for the entire upgrade.
      > ping -t [IP_Address]

---

## <a id="phase-2-firmware-upload-preparation"></a>📤 Phase 2: Firmware Upload & Preparation

1.  **Enter Maintenance Utility:** 🧰
    * Log into the array's **Maintenance Utility**.
2.  **Navigate to Update Menu:** 📂
    * Go to **Administration** > **Firmware** > **Update Firmware**.
    * *(Note: Depending on your exact interface version, you may need to click 'Later' if prompted to schedule, to ensure manual control).*
3.  **Upload the File:** ⬆️
    * Select the downloaded firmware .iso or .zip file from your local machine.
    * Click **Apply** or **Upload** to stage the file on the array.

---

## <a id="phase-3-the-upgrade-execution-standard-path"></a>🔀 Phase 3: The Upgrade Execution (Standard Path)
*Follow this path if the Pre-Checks confirmed there are NO warnings on any HDDs.*

### Step 1: GUM Upgrade First 🧠
*Upgrading the management layer first ensures stable communication for the rest of the upgrade.*
1.  **Select Components:** On the firmware upgrade selection screen, **UNCHECK** everything except **GUM**.
2.  **Execute:** Click to proceed.
3.  **Monitor:** Watch your continuous pings. You will see the controllers drop pings one by one as they reboot the GUM services. 
    * *Do not refresh the browser aggressively during this phase.*
4.  **Verify:** Wait for the interface to report "GUM Upgrade Complete" and verify both controller IPs are responding to ping again.

### Step 2: Remaining Upgrades ⚙️
1.  **Select Components:** Return to the firmware update screen. 
2.  **Check the Rest:** This time, **UNCHECK GUM** (since it is already done) and check all remaining components (Controllers, Drive Firmware, etc.).
3.  **Execute:** Proceed with the upgrade. The array will handle the rolling reboots of the controllers.
4.  **Completion:** Wait for the "Upgrade Complete" confirmation.

---

## <a id="phase-3b-the-upgrade-execution-hdd-warning-exception-path"></a>⚠️ Phase 3B: The Upgrade Execution (HDD Warning Exception Path)
*Follow this path ONLY if you noticed a warning 🟡 on the HDD components during Phase 1.*

1.  **Isolate & Upgrade GUM:** 🧠
    * On the component selection screen, check **ONLY GUM**. Execute and wait for completion (monitor via ping as described above).
2.  **Upgrade Main Components (Excluding HDD):** 🔀
    * Once GUM is complete, return to the upgrade screen.
    * **UNCHECK** GUM.
    * **UNCHECK** HDD.
    * Check all other remaining components. Execute the upgrade.
3.  **Wait for Stabilization:** ⚖️
    * After this phase completes, allow the array a few minutes to stabilize. 
    * Check the **System GUI**. Ensure the system has returned to a normal state (aside from the pre-existing HDD warning).
4.  **Upgrade HDD Firmware Last:** 💽
    * Return to the upgrade screen one final time.
    * Check **ONLY HDD**.
    * Execute the upgrade. 

---

## <a id="phase-4-post-upgrade-validation"></a>✅ Phase 4: Post-Upgrade Validation
*Ensuring the array survived the rolling reboots and is fully functional.*

1.  **Verify Firmware Version:** 🏷️
    * **How:** In **Maintenance Utility**, look at the main dashboard or **Firmware** tab.
    * **Look for:** Confirm the 'Current Version' matches the target firmware version you uploaded.
2.  **Final Health Check:** 🩺
    * **How:** Navigate back to **Hardware** > **System GUI**.
    * **Look for:** All components must be 🟢 **Normal**. If the HDD warning from Phase 3B was a firmware bug, it should now be cleared.
3.  **Check for New SIMs:** 📋
    * **How:** Go to **Alerts / Events** > **SIMs**.
    * **Look for:** Acknowledge the informational SIMs generated by the reboots (e.g., path drops). Ensure there are no *new* critical hardware failures.
4.  **Confirm Host Recovery:** 🛣️
    * **How:** Have the server administrators run their multipath checks again.
    * **Look for:** All paths must be active. No dead, failed, or ghost paths should remain.
