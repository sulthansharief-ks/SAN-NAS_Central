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


### **🚀 Standard Operating Procedure: Mapping Existing LDEVs to Host Groups**
**System:** Hitachi VSP G-Series / F-Series / G1000 / G1500  
**Management Tool:** Hitachi Storage Navigator (Device Manager - Storage Navigator)  

---

## **1. Purpose 🎯**
This SOP provides the exact steps to establish a connection (LUN Path) between an existing **Logical Device (LDEV)** and a **Host Group**. This process—often called **LUN Mapping**—makes storage volumes visible to specific servers.

---

## **2. Prerequisites 📋**
* **LDEV Status:** The target LDEV must already be created and formatted.
* **Host Group Status:** The Host Group must already exist, with at least one Host WWN registered.
* **Login Credentials:** Authorized access to the SVP management interface.

---

## **3. Procedure 🛠️**

### **Step 1: Locate the Target Host Group 🖥️**
* Open your browser and log in to **Hitachi Device Manager - Storage Navigator**.
* In the left-hand explorer panel, expand the **Storage Systems** tree and select your storage array.
* Navigate to **Ports / Host Groups / iSCSI Targets**.
* In the list, find the specific **Host Group** you want to map storage to.

### **Step 2: Initiate the "Add LUN Paths" Wizard 🔗**
* Select the checkbox next to your target **Host Group**.
* Click the **Add LUN Paths** button (usually located above the list or under the "More Actions" menu).
* A new wizard window will appear labeled **Add LUN Paths**.

### **Step 3: Select the LDEVs to Map 💾**
* The wizard will display a list of **Available LDEVs** (volumes not currently mapped to this group).
* **Filter/Search:** Use the filter tool to find your LDEV by its **LDEV ID** or **LDEV Name**.
* Select the desired LDEV(s) and click the **Add** button to move them to the "Selected LDEVs" table.
* Click **Next**.

### **Step 4: Configure LUN IDs 🔢**
* In the **LUN ID** column, the system will automatically suggest the next available LUN number (e.g., LUN 0, LUN 1).
* **Manual Override:** If your organization requires a specific LUN ID, you can manually edit it here.
* Click **Next** to proceed to the confirmation screen.

### **Step 5: Review and Commit ✅**
* Review the summary table showing the **Port ID**, **Host Group Name**, and the assigned **LDEV IDs** and **LUN IDs**.
* Click **Finish**.
* A confirmation dialog will appear. Click **Apply** to submit the task to the storage controllers.

---

## **4. Verification & Host Discovery 🔍**
* **Monitor Task:** Go to the **Tasks & Alerts** window to ensure the status changes to **Completed**.
* **Host-Side Action:** * Log in to the target server.
    * **VMware:** Perform a "Rescan Storage" on the HBA.
    * **Windows:** Open "Disk Management" and select "Action > Rescan Disks".
    * **Linux:** Run `rescan-scsi-bus.sh` or scan the specific host class in `/sys/class/fc_host/`.
* The new volume should now appear as an uninitialized disk.

---


# 🚀 Standard Operating Procedure: Storage LUN Allocation & Provisioning
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
* **Capacity Configuration:** * Set the **Provisioning Type** to Dynamic Provisioning.  
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

## **5. Troubleshooting Guide: "Task Failed" 🛠️**

| Symptom / Error | Probable Cause | Recommended Action |
| :--- | :--- | :--- |
| **Insufficient Capacity** | Pool lacks free physical space. | Add Pool-VOLs or delete unused LDEVs. |
| **ID Conflict** | CU/DEV ID already in use. | Use "View LDEV IDs" to find an available ID. |
| **WWN Conflict** | WWN assigned to another group on the same port. | Remove WWN from the old group first. |
| **Timeout/Hanging** | Communication lag between SVP and Controller. | Wait 5 mins; refresh UI; check Maintenance Utility. |
| **Mode Mismatch** | Incorrect Host Mode (e.g., missed Mode 21). | Verify OS-specific mode and required options (54/63). |

---


