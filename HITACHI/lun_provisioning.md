### **🚀 Standard Operating Procedure: Storage LUN Allocation & Provisioning**
**System:** Hitachi VSP G-Series / G1000 / G1500  
**Management Tool:** Hitachi Storage Navigator / Device Manager  

---

## **1. Purpose 🎯**
This SOP outlines the standardized process for creating Host Groups and provisioning Logical Devices (LDEVs) to ensure consistent and reliable storage allocation across the environment.

---

## **2. Prerequisites 📋**
* Access to the **SVP (Service Processor)** or management network.
* Authorized credentials for **Hitachi Device Manager**.
* Target Host HBA **World Wide Names (WWNs)**.
* Identification of the target **Storage Pool** for allocation.

---

## **3. Procedure 🛠️**

### **Step 1: System Access & Navigation 🌐**
* Open a supported web browser and navigate to the **SVP IP address**.
* Log in to the **Hitachi Device Manager/Storage Navigator** interface.
* Verify the storage system serial number and health status on the homepage dashboard.

### **Step 2: Host Group Configuration (LUN Masking) 🖥️**
* Navigate to the **Ports / Host Groups / iSCSI Targets** section in the left-hand explorer panel.
* Click **Create Host Group** to launch the setup wizard.
* **Identification:** Enter the standardized **Host Group Name** and select the appropriate **Resource Group**.
* **Host Mode:** Select the **Host Mode** corresponding to the server OS (e.g., Select **'21'** for VMware Extension).
* **Mode Options:** Click **Host Mode Options** and enable required settings (e.g., Enable **'54'** and **'63'** for VAAI compliance).
* **WWN Assignment:** * Locate and select the server's WWNs from the available list.  
    * If WWNs are not discovered, use the **Add New Host** function to manually input the 16-digit hexadecimal addresses.
* **Port Mapping:** Select the specific physical ports (e.g., **CL3-E** and **CL4-E**) assigned for the host's redundancy paths.
* Click **Finish** and then **Apply** to execute the host group creation.

### **Step 3: LDEV (LUN) Creation & Mapping 💾**
* Navigate to the **Pools** section in the sidebar to view available storage pools.
* Select the target storage pool and click the **Virtual Volumes** tab.
* Click **Create LDEV** to open the provisioning wizard.
* **Capacity Configuration:** * Set the **Provisioning Type** (typically Dynamic Provisioning).  
    * Define the **LDEV Capacity** (TB, GB, or MB) and the **Number of LDEVs** required.
* **ID & Naming:** Define the **LDEV Name** prefix and assign the **CU ID** and **DEV ID**.
* **Controller Assignment:** Set the **MP Unit ID** (Processor unit); using **'Auto'** is recommended for active-active controllers.
* **Allocation Policy:** Choose between thin provisioning or enable **Full Allocation** for thick provisioning.
* **Path Assignment:** Click **Next** to move to the LUN Path screen, select the previously created **Host Groups**, and click **Add**.
* Review the final summary, click **Finish**, and then **Apply** to confirm.

---

## **4. Verification & Finalization ✅**
* Navigate to the **Tasks** window to monitor the job status.
* Confirm the task status is marked as **Completed**.
* Perform a **Storage Rescan** on the host server to verify the new LUNs are visible and ready for use.

---

**Would you like me to create a troubleshooting guide for common "Task Failed" errors in Storage Navigator?**
