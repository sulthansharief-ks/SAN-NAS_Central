# 📊 Standard Operating Procedure: Generating LDEV Capacity & Utilization Reports
**System:** Hitachi VSP Series  
**Management Tools:** Hitachi Device Manager - Storage Navigator (HDvM-SN) & Ops Center Administrator

---

## **Table of Contents 📑**
* [1. Purpose 🎯](#1-purpose)
* [2. Method A: The Storage Navigator Export (Raw CSV Method) 📝](#2-method-a-storage-navigator)
    * [Step 1: Access the Logical Devices View](#step-1-access-logical-devices-view)
    * [Step 2: Export the Data](#step-2-export-data)
    * [Step 3: Excel Pivot & Filtering](#step-3-excel-pivot-filtering)
* [3. Method B: Hitachi Ops Center Administrator (The Modern Method) 🚀](#3-method-b-ops-center)
    * [Step 1: Access the Storage/Volumes Dashboard](#step-1-access-volumes-dashboard)
    * [Step 2: Filter by Host or Server](#step-2-filter-by-host)
    * [Step 3: Download the Report](#step-3-download-report)
* [4. Understanding the Reporting Limitations ⚠️](#4-reporting-limitations)

---

## <a id="1-purpose"></a>**1. Purpose 🎯**
Storage administrators are frequently asked by management or app owners to provide a breakdown of exactly how much storage a specific host is consuming versus how much was allocated. 

Because of Dynamic Provisioning (HDP), a standard LUN has two distinct sizes: its **Logical Capacity** (what you subscribed/presented to the host) and its **Allocated Capacity** (the physical pages it has actually consumed from the pool). 

This SOP covers how to extract these metrics for every LUN mapped to a host, using either the traditional Storage Navigator or the modern Ops Center.

---

## <a id="2-method-a-storage-navigator"></a>**2. Method A: The Storage Navigator Export (Raw CSV Method) 📝**
*This is the universally available method on every VSP, though it requires a bit of Excel filtering afterward.*

### <a id="step-1-access-logical-devices-view"></a>**Step 1: Access the Logical Devices View**
1. Log in to **Storage Navigator**.
2. Navigate to **Storage Systems** > **Logical Devices**.
3. *Crucial Step:* By default, SN does not show all the columns you need. Click the **Column Settings** button (the small grid icon on the far right of the table header, or right-click the column header and select Column Settings).
4. Ensure the following columns are checked and visible:
   * **LDEV ID**
   * **LDEV Name** (If you use naming conventions for your hosts)
   * **Capacity** (This is your *Subscribed Capacity*)
   * **Allocated Capacity** (This is your *Pool Used Capacity*)
   * **Provisioning Type** (To filter out basic volumes; you want DP)
   * **Attribute** (To see if it is a Command Device, TSE, etc.)
   * **Number of Paths** (To ensure it is actually mapped to a host; 0 means it is unmapped).

### <a id="step-2-export-data"></a>**Step 2: Export the Data**
1. Do not try to read this all in the GUI if you have thousands of LDEVs.
2. In the bottom right corner of the screen, click **More Actions** > **Export**.
3. A prompt will appear. Select **All Rows** (or **Selected Rows** if you used the GUI filter).
4. Click **OK**. A .csv file will download to your local machine.

### <a id="step-3-excel-pivot-filtering"></a>**Step 3: Excel Pivot & Filtering 📈**
1. Open the CSV in Microsoft Excel.
2. Filter the **Number of Paths** column to exclude 0 (this removes all unused/unmapped LUNs from your report).
3. Filter the **Provisioning Type** to DP (Dynamic Provisioning).
4. You now have a clean report showing exactly what is mapped. 
   * *Pro-Tip:* Subtract the **Allocated Capacity** from the **Capacity** to calculate the exact amount of "Dead Space" (provisioned but unused capacity) per LUN.

---

## <a id="3-method-b-ops-center"></a>**3. Method B: Hitachi Ops Center Administrator (The Modern Method) 🚀**
*If your environment uses Ops Center, reporting is vastly superior and host-aware. You can pull the report directly by the Host name rather than filtering raw LDEV IDs.*

### <a id="step-1-access-volumes-dashboard"></a>**Step 1: Access the Storage/Volumes Dashboard**
1. Log in to **Hitachi Ops Center Administrator**.
2. On the left-hand navigation menu, click **Storage** > **Volumes**.
3. Ops Center has a unified view. You will immediately see columns for **Capacity** and **Allocated Capacity**.

### <a id="step-2-filter-by-host"></a>**Step 2: Filter by Host or Server 🖥️**
1. To find the LUNs for a specific server, use the global search bar at the top or click the **Filter** icon.
2. Set the filter to **Server** (or Host) and type the name of the server (e.g., PROD_ESXi_Cluster_01).
3. The view will instantly restrict to only the LUNs mapped to that specific host.

### <a id="step-3-download-report"></a>**Step 3: Download the Report**
1. Click the **Download** icon (the downward-pointing arrow at the top right of the data table).
2. The system will generate a beautifully formatted CSV or Excel file containing the exact host mapping, the subscribed LUN capacity, and the physical allocated capacity.

---

## <a id="4-reporting-limitations"></a>**4. Understanding the Reporting Limitations ⚠️**
When you generate these reports, you will be able to provide the **Subscribed Capacity** and the **Allocated Capacity**. 

However, you **cannot** easily pull the **Host Used Capacity** (what the OS guest filesystem sees as free space) directly from the storage array's basic export. 
* *Why?* The storage array is block-level. It only knows which blocks it has handed out (Allocated). It cannot natively read the NTFS, VMFS, or EXT4 filesystem sitting on top of those blocks. 
* *Solution:* If management demands the exact OS-level utilization, you must cross-reference your Storage Navigator export with a report generated by the VMware Administrator (vCenter Datastore report) or the Windows/Linux Server team. Alternatively, this level of deep integration requires **Hitachi Ops Center Analyzer** with specific host-agent integrations enabled, allowing the Analyzer Probe to query vCenter or the OS directly to bridge the gap.
