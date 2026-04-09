# 🚀 Master Standard Operating Procedure: Hitachi Ops Center Ecosystem 🌐
**System:** 🏢 Hitachi Ops Center (Administrator, Analyzer, Automator, Protector, Common Services)  
**Context:** 📚 Comprehensive guide covering deployment, component interconnection (Probes), master exhaustive task lists, daily operations, advanced system administration, log collection, upgrading, and troubleshooting.  

---

## **Table of Contents 📑**
* [1. Deployment & Component Architecture 🏗️](#1-deployment-architecture)
    * [1.1 Deployment Methods 📦](#11-deployment-methods)
    * [1.2 Core Components & Interconnections 🔗](#12-core-components)
    * [1.3 The Role of Analyzer Probes 📡](#13-analyzer-probes)
* [2. Ops Center Master Task List (Exhaustive) 📋](#2-master-task-list)
* [3. System Administration, Maintenance & Logging ⚙️](#3-system-administration)
    * [3.1 Restarting Ops Center Services 🔄](#31-restarting-services)
    * [3.2 Troubleshooting: Storage System Not Visible 🕵️‍♂️](#32-storage-system-not-visible)
    * [3.3 Onboarding a New Storage System ➕](#33-onboarding-array)
    * [3.4 User Access & RBAC Management 🔐](#34-user-access)
    * [3.5 Registering an Analyzer Probe to Ops Center 🔌](#35-registering-analyzer-probe)
    * [3.6 Log Collection (Dump Files & Audit Logs) 🗃️](#36-log-collection)
    * [3.7 License Management 🔑](#37-license-management)
    * [3.8 Upgrading Ops Center (In-Place Upgrade) ⬆️](#38-upgrading-ops-center)
* [4. Daily Operations (Provisioning, Reclaiming & Mobility) 🗄️](#4-daily-operations)
    * [4.1 Registering a New Host (Server) 🖥️](#41-registering-host)
    * [4.2 Allocating & Mapping Volumes 🎯](#42-allocating-volumes)
    * [4.3 Detaching (Unmapping) Volumes ✂️](#43-detaching-volumes)
    * [4.4 Deleting / Shredding Volumes ♻️](#44-deleting-volumes)
    * [4.5 Volume Migration & External Storage (UVM) 🚚](#45-volume-migration)

---

## <a id="1-deployment-architecture"></a>**1. Deployment & Component Architecture 🏗️**

### <a id="11-deployment-methods"></a>**1.1 Deployment Methods 📦**
Hitachi Ops Center is primarily deployed in modern enterprise environments using one of two methods:
1. **Virtual Appliance (OVA):** 💻 Pre-packaged VMware virtual machines containing the Linux OS and all necessary Ops Center modules.
2. **Docker / Kubernetes Containers:** 🐳 For massive scalability, Ops Center can be deployed as Docker containers running on a dedicated Linux host (RHEL/Oracle Linux), orchestrated via docker-compose.

### <a id="12-core-components"></a>**1.2 Core Components & Interconnections 🔗**
All modules share telemetry and are accessed via a single pane of glass.
* 🚪 **Common Services (csportal):** The gateway layer. It handles Single Sign-On (SSO), LDAP/AD integration, user roles, and the unified launchpad portal.
* 🛠️ **Administrator (Rainier):** The execution engine. Pushes REST API commands directly to the array's SVP or GUM to carve storage and map LUNs.
* 🧠 **Analyzer (HTnM):** The AI engine. Receives telemetry from the infrastructure to forecast capacity limits and identify latency bottlenecks.
* 🤖 **Automator:** The workflow orchestration engine. Uses custom templates to execute multi-step tasks across compute, network, and storage.
* 🛡️ **Protector:** The copy-data management tool. Orchestrates local snapshots (Thin Image) and remote replication (TrueCopy / GAD).

### <a id="13-analyzer-probes"></a>**1.3 The Role of Analyzer Probes 📡**
Ops Center Analyzer does not natively reach out to all devices by itself. It relies on a separate architectural component called the **Analyzer Probe**.
* 🖥️ **What it is:** A lightweight data-collector server (often a separate VM).
* 🔌 **How it connects:** The Probe uses vendor-specific APIs to connect to your endpoint devices (e.g., polling VMware vCenter via SOAP API, polling Brocade switches via REST/SMI-S, and polling VSP arrays via Export Tool). It translates all these different languages into a unified format.
* 🌊 **The Flow:** Endpoints ➡️ Analyzer Probe ➡️ Ops Center Analyzer Server. This separates the heavy lifting of data collection from the AI analysis engine.

---

## <a id="2-master-task-list"></a>**2. Ops Center Master Task List (Exhaustive) 📋**
Below is a comprehensive matrix of every major operation you can perform across the Ops Center suite.

| Module | Category | Available Tasks / Actions |
| :--- | :--- | :--- |
| **Administrator** | Storage Provisioning | Create Volumes, Expand Volumes, Delete Volumes, Shred Volumes (Secure Erase 3-Pass), Format LDEVs. |
| **Administrator** | Host Management | Add Servers, Map WWPNs/iSCSI IQNs, Assign Host Modes, Attach Volumes, Detach Volumes, Configure iSCSI Targets/CHAP. |
| **Administrator** | Parity & Pools | Initialize Parity Groups, Create DP Pools, Expand Pools, Set Pool Warning/Depletion Thresholds, Monitor System Area. |
| **Administrator** | Hardware Mgmt | Onboard Arrays, Configure Front-End Ports (Speed/Topology/Fabric), Monitor Hardware Alerts (SIMs), Power Manage Ports. |
| **Administrator** | System Maintenance | **Collect Dump Files (Logs)**, Download Audit Logs, Install/Update Licenses, Configure SNMP Traps, Configure Syslog forwarding. |
| **Administrator** | Data Mobility | Perform Non-Disruptive Migration (NDM), Volume Migration (Tiering), Virtualize External Storage (Universal Volume Manager - UVM). |
| **Administrator** | Basic Replication | Create Local Snapshots (Thin Image), Create Local Clones (ShadowImage). |
| **Analyzer** | Performance | Monitor End-to-End Latency, View Sparkline Graphs (IOPS/MBps), Run Root Cause Analysis (RCA), Analyze Noisy Neighbors. |
| **Analyzer** | Capacity | View Array/Pool Fill Rates, Forecast Capacity Depletion Date, Monitor Data Reduction (Compression/Dedupe) Ratios. |
| **Analyzer** | Alerting & Reporting | Configure Alert Profiles, Integrate Alerts to ITSM (ServiceNow), Generate custom PDF/CSV reports for specific Host utilization. |
| **Automator** | Orchestration | Execute pre-built templates (e.g., "Provision Datastore to ESXi", "Zoning Brocade Switch"), Build Custom Workflows, REST API orchestration. |
| **Protector** | Advanced Protection | Create Snapshot Policies, Mount Clones for Dev/Test, Configure Metro-Cluster (Global-Active Device) paths, TrueCopy/Universal Replicator setup. |
| **Common Services** | Security & Auth | Manage Local Users, Connect to Active Directory/LDAP, Configure Role-Based Access Control (RBAC), Manage SSL Certificates, Set Password Policies. |

---

## <a id="3-system-administration"></a>**3. System Administration, Maintenance & Logging ⚙️**

### <a id="31-restarting-services"></a>**3.1 Restarting Ops Center Services 🔄**
If the Ops Center UI becomes unresponsive, throws HTTP 500 errors, or fails to execute provisioning tasks, the underlying Linux services must be restarted.

💻 SSH into the Linux host running Ops Center using root or sudo credentials.

1. To restart the main portal (SSO and UI Launchpad):
   > systemctl restart csportal

2. To restart the Administrator module (The provisioning engine, codenamed 'Rainier'):
   > systemctl restart rainier

3. To restart the Analyzer engine:
   > /opt/jp1pc/htnm/bin/htmsrv restart -all

4. *(Optional)* If you are running a Docker-based deployment and the above fails, you may need to restart the Docker daemon itself:
   > systemctl restart docker

⏳ Wait 5-10 minutes for the Java microservices to fully initialize before attempting to log back into the UI.

### <a id="32-storage-system-not-visible"></a>**3.2 Troubleshooting: Storage System Not Visible / Disconnected 🕵️‍♂️**
If an array is missing or shows a 🔴 Red Disconnected icon in Ops Center Administrator.

**Step A: Check Credentials (Authentication Failure) ❌**
1. In Ops Center Administrator, navigate to **Storage Systems**.
2. Select the disconnected array and click **Edit Storage System**.
3. Re-enter the storage administrator username and password for the array's SVP/GUM.
4. ⚠️ *Warning:* If Ops Center polled the array with an expired password too many times, the local user account on the VSP may be locked. You must log directly into the array's SVP/GUM to unlock the account first.

**Step B: Force a Database Resync 🔄**
1. Navigate to **Storage Systems**.
2. Select the array.
3. Click the **Refresh** icon (circular arrows). This forces Ops Center to dump its cache and pull a fresh configuration map directly from the controller's Shared Memory.

### <a id="33-onboarding-array"></a>**3.3 Onboarding a New Storage System ➕**
Ops Center does not auto-discover new hardware; it must be explicitly added.

1. Log in to **Ops Center Administrator**.
2. Navigate to **Storage Systems** and click the **+ Add** button.
3. Enter the **IP Address** 🌐 of the array's Service Processor (SVP) or GUM interface.
4. Enter the storage administrator credentials 👤.
5. Click **Submit** ✅. Ops Center will connect, download the array's configuration metadata, and begin monitoring it.

### <a id="34-user-access"></a>**3.4 User Access & RBAC Management 🔐**
1. Log in to the main **Ops Center Portal** (Common Services).
2. Click the gear icon ⚙️ (Settings) in the top right > **Security**.
3. Navigate to **Users & Roles** 👥.
4. Click **Create User**, input their Active Directory details (if LDAP is configured) or create a local user.
5. Assign granular roles 🛡️ (e.g., Storage Administrator, Security Administrator, Monitor).

### <a id="35-registering-analyzer-probe"></a>**3.5 Registering an Analyzer Probe to Ops Center 📡**
**Step A: Register the Probe Server 🖥️**
1. Log in to **Ops Center Analyzer**.
2. Click the **Gear Icon** ⚙️ > **Data Collection**.
3. Click **+ Add Probe Server**. Enter the IP Address and Communication Port (default 22015/443).
4. Input the Authentication Token 🔑. Click **Test Connection**, then **Submit**.

**Step B: Add Targets to the Probe 🎯**
1. In the Data Collection menu, select the Probe you just registered.
2. Click **Add Target**.
3. Select the resource type 🗄️ (e.g., Hitachi VSP System, VMware vCenter).
4. Enter the specific endpoint IP (e.g., the VSP SVP IP) and credentials. Save and initiate collection 🚀.

### <a id="36-log-collection"></a>**3.6 Log Collection (Dump Files & Audit Logs) 🗃️**
If Hitachi Global Support Center (GSC) requests logs for a hardware failure or software bug, you extract them via Ops Center.

**A. Collecting Array Dump Files (Trace Logs) 🖨️**
1. Log in to **Ops Center Administrator**.
2. Navigate to **Storage Systems** and click on your specific array.
3. Click the **More Actions** menu (the three vertical dots in the top right) > **Download Dump Tool Data**.
4. Select the specific components requested by GSC (usually Normal Dump or Detail Dump).
5. Click **Submit** ✅. The array will package the internal microcode logs. Once complete, it will download as a .tgz or .zip to your local browser.

**B. Collecting Ops Center Server Logs (Rainier Logs) 💻**
If the Ops Center UI itself is crashing, you need the server logs.
1. SSH into the Ops Center Administrator Linux host as root.
2. Run the log collection script:
   > /opt/hitachi/opscenter/rainier/bin/getlogs
3. The script will output a compressed tarball 📦 in the `/tmp/` directory containing all tomcat, API, and database logs to send to support.

**C. Downloading Audit Logs 🕵️**
1. In Ops Center Administrator, click the **Gear Icon** ⚙️ (Settings) > **Audit Logs**.
2. Select the date range 📅.
3. Click **Export** 📥. This generates a CSV of every user login, LUN creation, and deletion event for compliance reviews.

### <a id="37-license-management"></a>**3.7 License Management 🔑**
1. Log in to **Ops Center Administrator**.
2. Navigate to **Storage Systems** and select the array.
3. Click **More Actions** > **License Keys** 🎫.
4. From here, you can view capacity limits for specific features 📊 (like Dynamic Provisioning or Local Replication).
5. To add a new license, click **Install Licenses**, browse for the .txt or .xml key file provided by Hitachi, and click **Submit** ✅.

### <a id="38-upgrading-ops-center"></a>**3.8 Upgrading Ops Center (In-Place Upgrade) ⬆️**
Ops Center is upgraded using a unified installer media (ISO) provided via the Hitachi support portal.

**Step A: Pre-Requisites & Safety 🛑**
1. Download the latest Ops Center unified ISO file to your local machine or jump server 📥.
2. ⚠️ **IMPORTANT:** Take a crash-consistent VMware Snapshot 📸 of ALL Linux VMs hosting Ops Center components before proceeding.
3. Ensure you have the root password 🔑 for the Linux hosts.

**Step B: Mount the ISO & Run Installer 💿**
1. SSH into the primary Ops Center Linux host as root.
2. Transfer the ISO to the server (e.g., via WinSCP to the `/tmp/` directory).
3. Mount the ISO to a local directory 📂:
   > mkdir -p /media/cdrom
   > mount -o loop /tmp/Hitachi_Ops_Center_vXX.iso /media/cdrom
4. Navigate to the mount directory:
   > cd /media/cdrom
5. Execute the unified setup script 🚀:
   > ./setup
6. The Text User Interface (TUI) wizard will launch 🧙‍♂️. It will automatically detect the currently installed modules.
7. Follow the prompts to confirm the upgrade path. The installer will automatically stop the necessary services, replace the Java/Tomcat binaries, upgrade the backend PostgreSQL databases, and restart the services 🔄.
8. Unmount the ISO when complete:
   > cd /
   > umount /media/cdrom

**Step C: Post-Upgrade Verification ✅**
1. Clear your web browser's cache 🧹 (CTRL + F5).
2. Log in to the Common Services portal.
3. Navigate to the **About** section ℹ️ and verify all modules reflect the new version number.
4. Navigate to Administrator and verify that all Storage Systems show a 🟢 green Normal status.

---

## <a id="4-daily-operations"></a>**4. Daily Operations (Provisioning, Reclaiming & Mobility) 🗄️**

### <a id="41-registering-host"></a>**4.1 Registering a New Host (Server) 🖥️**
1. Log in to **Ops Center Administrator**.
2. Navigate to **Storage** > **Servers**. Click **+ Add Server**.
3. Enter the **Server Name** 🏷️ (e.g., PROD_ESXi_01).
4. Select the **Host Mode** 🐧 (e.g., VMware, Windows, Linux).
5. Under **Ports/WWNs**, select the initiators from the Discovered list, or enter them manually. Click **Submit** ✅.

### <a id="42-allocating-volumes"></a>**4.2 Allocating & Mapping Volumes 🎯**
1. Navigate to **Storage** > **Volumes**. Click **+ Create Volumes**.
2. **Volume Details 📝:** Select target VSP, target DP Pool, Number of Volumes, Capacity, and Volume Name Prefix.
3. Click **Next** ➡️.
4. **Attach Volumes 🔗:** Select the **Server** you created in Step 4.1. Ops Center automatically selects the optimal Front-End Director (FED) ports. Click **Submit** ✅.

### <a id="43-detaching-volumes"></a>**4.3 Detaching (Unmapping) Volumes ✂️**
🛑 **STOP:** Ensure the server/OS admin has offlined the disk at the OS level before proceeding.

1. Navigate to **Storage** > **Servers**.
2. Click on the specific Server to view its details 🔍.
3. Scroll down to the **Attached Volumes** tab. Check the box ☑️ next to the volume(s) you wish to remove.
4. Click the **Detach** button ✂️. Confirm the action.

### <a id="44-deleting-volumes"></a>**4.4 Deleting / Shredding Volumes ♻️**
1. Navigate to **Storage** > **Volumes**.
2. Locate the volume you just detached (It should show 0 paths) 🪹.
3. Check the box next to the volume. Click the Trash Can icon 🗑️ (**Delete**).
4. **Shredding (Optional) 🌪️:** Check the box for **Shred Volume** during deletion. Ops Center will overwrite the volume with zeros or a 3-pass DoD pattern.
5. Click **Submit** ✅.

### <a id="45-volume-migration"></a>**4.5 Volume Migration & External Storage (UVM) 🚚**
Used for migrating data non-disruptively from an older array to a new one, or moving a LUN from a slow HDD pool to an NVMe pool.

**A. Volume Migration (Internal Tiering) 🛗**
1. Navigate to **Storage** > **Volumes**.
2. Select the volume you want to move 📦.
3. Click **More Actions** > **Migrate Volumes**.
4. Select the **Target Pool** 🎯 (e.g., your new NVMe pool).
5. Ops Center will utilize Hitachi Tiering logic to move the 42MB pages in the background transparently to the host 👻.

**B. External Storage Virtualization (UVM) 🪄**
1. Zone the old legacy array 📼 (e.g., an old NetApp or older VSP) to the Back-End ports of your new VSP.
2. In Ops Center Administrator, navigate to **Storage Systems** > Select your new Array > **External Storage** 🌐.
3. Click **Discover External Storage** 🔭. The VSP will scan the backend ports and find the old LUNs.
4. Select the discovered LUNs and click **Virtualize** 🔮. They now appear as normal LDEVs inside the new VSP, ready to be migrated into your DP Pools.
