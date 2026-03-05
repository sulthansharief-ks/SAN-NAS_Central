# 🩺 Standard Operating Procedure: Hitachi Storage Daily Health Check
**System:** Hitachi VSP G-Series / F-Series / G1000 / G1500  
**Management Tool:** Hitachi Storage Navigator / Maintenance Utility  

## 📑 Table of Contents
1. [1. Purpose 🎯](#purpose)
2. [2. Prerequisites 📋](#prerequisites)
3. [3. Procedure 🛠️](#procedure)
   * [Step 1: Dashboard Status Review 🌐](#step-1)
   * [Step 2: Component Health Inspection ⚙️](#step-2)
   * [Step 3: Pool & Capacity Monitoring 📉](#step-3)
   * [Step 4: Performance Check (Analytics) 📊](#step-4)
   * [Step 5: Backup & Replication Status 🔄](#step-5)
4. [4. Error Recovery & Escalation ✅](#error-recovery)
5. [5. Reporting 📝](#reporting)

---



<a id="purpose"></a>
## **1. Purpose 🎯**
To proactively identify hardware failures, capacity bottlenecks, and performance anomalies within the storage environment. Consistent execution of this SOP ensures maximum uptime and system integrity.

---

<a id="prerequisites"></a>
## **2. Prerequisites 📋**
* **Access**: Administrator-level credentials for **Hitachi Device Manager - Storage Navigator**.
* **Connectivity**: Network access to the **SVP (Service Processor)** IP address.
* **Documentation**: Knowledge of the baseline "Normal" state for system capacity and performance.

---

<a id="procedure"></a>
## **3. Procedure 🛠️**

<a id="step-1"></a>
### **Step 1: Dashboard Status Review 🌐**
* Log in to the management interface via the browser.
* **System Health**: Check the status icons on the top right of the dashboard. Ensure there are no **Red (Failure)** or **Yellow (Warning)** alerts.
* **Task Summary**: Review the **Tasks** window to ensure all automated system background jobs (like LDEV formatting or pool scrubbing) have reached **Completed** status.

<a id="step-2"></a>
### **Step 2: Component Health Inspection ⚙️**
* Navigate to **Administration** > **Components** (or **Hardware**).
* **Controllers**: Verify that both **MP Units** (Microprocessors) are showing a "Normal" status.
* **Drives/Cache**: Inspect the status of physical disks and cache modules. Look for "Degraded" or "Failed" statuses.
* **Ports**: Check **Ports / Host Groups** to ensure all active front-end ports are **Online** (Green).

<a id="step-3"></a>
### **Step 3: Pool & Capacity Monitoring 📉**
* Select **Pools** from the left explorer panel.
* **Usage Thresholds**: Check the **Total Capacity** vs. **Used Capacity** for each pool. 
    * *Warning:* If a Dynamic Provisioning pool reaches its **Depletion Threshold**, the system may block I/O.
* **Subscription Rate**: Ensure the subscription rate (over-provisioning) aligns with organizational safety limits.

<a id="step-4"></a>
### **Step 4: Performance Check (Analytics) 📊**
* Navigate to **Analytics** or **Performance Monitor**.
* **IOPS & Throughput**: Verify that current system activity matches expected workloads.
* **Latency Check**: Review the **Response Time** metric for LDEVs. High latency (spikes over 20ms) should be investigated for path failures or "Slow Drain" issues on the SAN fabric.

<a id="step-5"></a>
### **Step 5: Backup & Replication Status 🔄**
* Navigate to **Replication** (if applicable).
* Verify that all **ShadowImage**, **Thin Image**, or **GAD** pairs are in a "PAIR" or "MIRROR" status, indicating that data protection is current and active.

---

<a id="error-recovery"></a>
## **4. Error Recovery & Escalation ✅**
1.  **Acknowledge Alerts**: If an alert is found, click the **Alert Log** to view the 4-digit error code.
2.  **Configuration Report**: Generate a report from the **Maintenance Utility** to capture the system state for vendor analysis.
3.  **Support Contact**: If hardware is marked as "Failed," immediately open a ticket with Hitachi Vantara Support with the captured logs.

---

<a id="reporting"></a>
## **5. Reporting 📝**
Record the following daily:
* **System Health Status**: (Pass/Fail)
* **Storage Pool Utilization**: (%)
* **Critical Tasks Result**: (Success/Failure)

---
