# 🗑️ Standard Operating Procedure: Storage LUN Unmapping & Decommissioning
**System:** Hitachi VSP G-Series / F-Series / G1000 / G1500  
**Management Tool:** Hitachi Storage Navigator (Device Manager - Storage Navigator)  

---

## **1. Purpose 🎯**
This SOP provides the standardized steps to safely remove **LUN paths** and decommission **Host Groups** when a server is being retired. This ensures that storage resources are reclaimed and the system configuration remains clean.

---

## **2. Prerequisites 📋**
* **Host-Side Preparation**: The server must be powered down or the disks must be taken offline and unmounted from the OS to prevent I/O errors.
* **Data Verification**: Ensure all required data has been migrated or backed up; once unmapped, the host loses all access immediately.
* **Resource Identification**: Maintain a list of specific **LDEV IDs** and the **Host Group Name** targeted for removal.

---

## **3. Procedure 🛠️**

### **Step 1: System Access 🌐**
* Log in to the **Hitachi Storage Navigator** via the SVP IP address.
* Ensure you are viewing the correct storage system serial number on the homepage.

### **Step 2: Remove LUN Paths (Unmapping) 🔗**
* Navigate to **Ports / Host Groups / iSCSI Targets** in the left explorer panel.
* Select the specific **Host Group** associated with the decommissioned server.
* Click the **LUNs** tab in the main display window to view all volumes currently mapped to that host.
* Select the LUNs intended for removal and click the **Remove LUN Paths** button.
* **Final Review**: Confirm the list of LUNs to ensure no production volumes are included, then click **Finish** followed by **Apply**.

### **Step 3: Delete Host Group (Optional) ✖️**
* Return to the **Host Group** list after the LUN paths are successfully removed.
* If the server is being permanently retired, select the empty **Host Group**.
* Click **Delete Host Group** to clear WWN registrations from the storage ports, freeing them for new assignments.

### **Step 4: Reclaim or Delete LDEVs ♻️**
* Navigate to the **Logical Devices** or **Pools** section.
* Locate the **LDEV IDs** that were just unmapped.
* **Re-use Option**: You may leave the LDEV intact for future mapping to a different server.
* **Deletion Option**: If the data is no longer needed, select the LDEV and click **Delete LDEV**.
* **Note**: For Dynamic Provisioning, deleting the LDEV automatically returns the capacity to the storage pool.

---

## **4. Verification ✅**
* Open the **Tasks** window to monitor job progress.
* Confirm the "Remove LUN Paths" and "Delete Host Group" tasks reach a **Completed** status.
* Verify the **Pools** summary to see that unallocated capacity has increased (if LDEVs were deleted).

---
**Prepared by:** Sulthan Sharief K S
**Last Updated:** February 2026

