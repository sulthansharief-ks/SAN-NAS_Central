# 📈 Standard Operating Procedure: Online LDEV Capacity Expansion
**System:** Hitachi VSP G-Series / F-Series / G1000 / G1500  
**Management Tool:** Hitachi Storage Navigator (Device Manager - Storage Navigator)  

---

## **1. Purpose 🎯**
This SOP defines the process for increasing the capacity of an existing **Virtual Volume (V-VOL)** while the host remains online. This allows for seamless storage growth without requiring the creation of new LUNs or the migration of data.

---

## **2. Prerequisites 📋**
* **Dynamic Provisioning**: The target LDEV must be a Virtual Volume (V-VOL) associated with a **Dynamic Provisioning Pool**.
* **Pool Capacity**: Ensure the target Storage Pool has sufficient physical free space to accommodate the expanded size.
* **Compatibility**: Verify the Host OS and File System support online volume expansion (e.g., Windows Disk Management, Linux LVM, or VMware VMFS).
* **No Resource Locks**: Ensure the LDEV is not part of an active replication pair (e.g., ShadowImage or GAD) unless the specific pair state allows expansion.

---

## **3. Procedure 🛠️**

![v-vol_operations](/HITACHI/media/hitachi vols.png)

### **Step 1: System Access 🌐**
* Log in to **Hitachi Storage Navigator** via the SVP IP address.
* Select the correct **Storage System** from the left-hand explorer tree.

### **Step 2: Locate the Target LDEV 💾**
* Navigate to the **Storage Systems** tab > **Logical Devices**.
* Use the **Filter** tool to search for the specific **LDEV ID** or **LDEV Name** that requires expansion.
* Verify the current capacity and the Pool ID it is associated with.

### **Step 3: Perform V-VOL Expansion 🚀**
* Select the checkbox for the target **LDEV**.
* Click **More Actions** (or right-click) and select **Expand V-VOL Capacity**.
* In the expansion wizard:
    * **Current Capacity**: Displays the existing size.
    * **Increased Capacity**: Enter the **total final size** or the amount to be added (ensure units like GB/TB are correctly selected).
* Click **Finish** to move to the confirmation screen.

### **Step 4: Confirm and Apply ✅**
* Review the **Task Name** and the list of LDEVs to be expanded.
* Click **Apply** to submit the task to the storage controllers.
* Navigate to the **Tasks** window and wait for the status to show **Completed**.

---

## **4. Host-Side Finalization 🏁**
*Once the storage task is complete, the host must be notified of the new capacity:*

* **Windows**: Open **Disk Management**, select **Action > Rescan Disks**. Right-click the existing volume and select **Extend Volume**.
* **Linux (LVM)**: Run `pvresize /dev/sdX` followed by `lvextend` and `resize2fs` (or `xfs_growfs`).
* **VMware**: Go to the **Datastore** view, select the volume, and click **Increase Datastore Capacity**.

---

## **5. Safety Warnings ⚠️**
* **Do Not Shrink**: Capacity expansion is a one-way operation. Hitachi V-VOLs cannot be shrunk once expanded.
* **Meta-Resource Restrictions**: Ensure the LDEV is not reserved by a specialized meta-resource group that prohibits configuration changes.

---
**Prepared by:** Sulthan Sharief K S  
**Last Updated:** February 2026
