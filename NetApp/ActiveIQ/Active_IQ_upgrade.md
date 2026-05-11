# 📈 NetApp Active IQ Unified Manager (AIQUM) for Windows: Master Upgrade SOP

Upgrading NetApp Active IQ Unified Manager on a Windows Server environment utilizes the NetApp InstallShield GUI wizard. While the execution is graphical, strict pre-upgrade backups and post-upgrade service verifications are mandatory to ensure your ONTAP monitoring remains uninterrupted.

This Master Standard Operating Procedure (SOP) is tailored specifically for Microsoft Windows Server deployments.

> 🏷️ **Note:** Verify your exact upgrade path using the NetApp Interoperability Matrix Tool (IMT). Skipping major versions (e.g., jumping from 9.8 directly to 9.14) often requires intermediate upgrades.

---

## 📋 Table of Contents
1. [🕵️ Phase 1: Pre-Upgrade Readiness Checks](#phase-1)
2. [💾 Phase 2: The Backup Protocol (Database & Server)](#phase-2)
3. [🪟 Phase 3: Upgrade Execution (Windows Installer)](#phase-3)
4. [✅ Phase 4: Post-Upgrade Verification (Services & GUI)](#phase-4)
5. [⏪ Phase 5: Rollback Procedure](#phase-5)
6. [📚 Official Documentation Reference](#references)

---

<a id="phase-1"></a>
## 🕵️ Phase 1: Pre-Upgrade Readiness Checks

1. **Verify Windows OS Compatibility:** Ensure your current Windows Server version (e.g., 2016, 2019, 2022) is supported by the new AIQUM version via the NetApp IMT.
2. **Review Resource Allocations:** Newer AIQUM versions require more RAM and CPU. Ensure your Windows VM meets the minimum specs (typically 12+ vCPUs and 32GB+ RAM for enterprise environments).
3. **Download the Installer:** Go to the NetApp Support Site and download the `.exe` installer (e.g., `ActiveIQUnifiedManager-<version>-windows.exe`). Save it to a local drive like `C:\Temp\` (do not execute over a mapped network drive).
4. **Silence Active Alerts:** Log into the AIQUM Web GUI, navigate to **Configuration** -> **Alert Setup**, and temporarily suspend email/SNMP alerts to prevent false positives during the reboot.
5. **Clear Browser Cache:** Clear your web browser's cache completely to prevent interface glitches after the upgrade.

---

<a id="phase-2"></a>
## 💾 Phase 2: The Backup Protocol (Database & Server)

*Because you are performing an in-place upgrade on a Windows OS, you must secure the application database and the OS itself.*

### 2.1 Generate a Native MySQL Database Backup
1. Log into the AIQUM Web GUI as the Maintenance User or Administrator.
2. Go to **Management** -> **Database Backup**.
3. Verify the Backup Settings are pointing to a directory with sufficient free space.
4. Click **Action** -> **Backup Now**.
5. Wait for the status to show `Completed`. Copy this backup file to an external server for safety.

### 2.2 Take a Windows Server VM Snapshot
1. Log into your hypervisor (e.g., VMware vCenter or Hyper-V Manager).
2. Locate the Windows Server hosting AIQUM.
3. Take a virtual machine snapshot. 
4. Uncheck "Snapshot the virtual machine's memory" if applicable.
5. Name it: `PRE-UPGRADE-AIQUM-WINDOWS`.

---

<a id="phase-3"></a>
## 🪟 Phase 3: Upgrade Execution (Windows Installer)



1. Log into the Windows Server via RDP using an account with **Local Administrator** privileges.
2. Navigate to the folder containing the downloaded `.exe` file.
3. Right-click `ActiveIQUnifiedManager-<version>-windows.exe` and select **Run as administrator**.
4. The InstallShield Wizard will launch, detect the existing installation, and display the **Upgrade** prompt.
5. Click **Next**.
6. The wizard will execute the following sequence automatically:
   * Stop the background Windows Services (MySQL, Management Server).
   * Back up the MySQL database schema.
   * Extract and overwrite the application binaries.
   * Migrate the database to the new schema.
   * Start the new Windows Services.
7. **Warning:** Do not interrupt this process. The database migration phase may appear stalled for several minutes depending on the size of your historical data.
8. Click **Finish** when the "Upgrade Completed Successfully" screen appears.

---

<a id="phase-4"></a>
## ✅ Phase 4: Post-Upgrade Verification (Services & GUI)

### 4.1 Verify Windows Services
1. Open the Windows Run dialog (`Win + R`), type `services.msc`, and press Enter.
2. Verify the following services show a status of **Running** and a Startup Type of **Automatic**:
   * `MySQL`
   * `NetApp Active IQ Acquisition Service`
   * `NetApp Active IQ Management Server Service`

### 4.2 GUI Access & Version Check
1. Open an Incognito/Private browser window.
2. Navigate to `https://<AIQUM_Windows_IP>`.
3. Log in. Check the top right corner (**Help** -> **About**) to confirm the new version is active.

### 4.3 Cluster Polling & AutoSupport
1. Navigate to **Storage** -> **Clusters**.
2. Check the **Collection Status** column. Ensure it transitions from `Discovering` to `Completed`. (This may take 15-30 minutes for the first polling cycle).
3. Navigate to **General** -> **AutoSupport** and click **Generate and Send AutoSupport** to verify outbound connectivity to NetApp.
4. Go to **Configuration** -> **Alert Setup** and re-enable your alerts.

### 4.4 Final Cleanup
1. Wait 24 to 48 hours to monitor system stability.
2. Delete the VM snapshot taken in Phase 2.2 to prevent hypervisor performance degradation.

---

<a id="phase-5"></a>
## ⏪ Phase 5: Rollback Procedure

*If the InstallShield wizard fails catastrophically or the services refuse to start, execute this rollback.*

1. Power off the Windows Virtual Machine in your hypervisor.
2. Revert the VM to the `PRE-UPGRADE-AIQUM-WINDOWS` snapshot taken in Phase 2.2.
3. Power the VM back on. 
4. Verify the AIQUM Web GUI is accessible and the three NetApp Windows Services are running. 
5. *(Alternative)* If the snapshot is corrupted, you must uninstall AIQUM via Windows Control Panel, install the *original* older version cleanly, and use the Native Database Restore function in the GUI to import the MySQL backup file from Phase 2.1.

---

<a id="references"></a>
## 📚 Official Documentation Reference

| Task | Official NetApp Documentation Reference |
| :--- | :--- |
| **Upgrade Paths / IMT** | [NetApp Interoperability Matrix Tool (IMT)](https://mysupport.netapp.com/matrix) |
| **Upgrading on Windows** | [Docs: Upgrading Active IQ Unified Manager on Windows](https://docs.netapp.com/us-en/active-iq-unified-manager/install-windows/concept_upgrading_unified_manager_on_windows.html) |
| **Database Backup** | [Docs: Managing Unified Manager Database Backups](https://docs.netapp.com/us-en/active-iq-unified-manager/health-checker/concept_managing_backup_and_restore_operations.html) |
