# Standard Operating Procedure: Veritas NetBackup Administration

<a id="table-of-contents"></a>
## 📖 Table of Contents
* [1. Service Management 🛠️](#1-service-management)
    * [1.1 Linux Servers (Master/Media/Client) 🐧](#11-linux-servers)
    * [1.2 Windows Servers (Master/Media/Client) 🪟](#12-windows-servers)
* [2. Configuring File System Backups 📁](#2-configuring-file-system-backups)
    * [2.1 Windows File System Backup 🪟](#21-windows-file-system-backup)
    * [2.2 Linux File System Backup 🐧](#22-linux-file-system-backup)
* [3. Configuring Database Backups 🗄️](#3-configuring-database-backups)
    * [3.1 Microsoft SQL Server Backup 📊](#31-microsoft-sql-server-backup)
    * [3.2 Oracle RMAN Backup 🛢️](#32-oracle-rman-backup)
* [4. Client Package Installation 📦](#4-client-package-installation)
    * [4.1 Windows Client Installation 🪟](#41-windows-client-installation)
    * [4.2 Linux Client Installation 🐧](#42-linux-client-installation)
* [5. Connectivity and Communication Tests 🌐](#5-connectivity-and-communication-tests)
    * [5.1 Master Server -> Client Connectivity Tests 📡](#51-master-to-client)
    * [5.2 Client -> Master Server Connectivity Tests 🔙](#52-client-to-master)
    * [5.3 Application-Level Connectivity Tests 🧩](#53-application-level)
    * [5.4 Clearing the Host Cache 🧹](#54-clearing-host-cache)
* [6. Essential NetBackup Command Line Utilities 💻](#6-essential-commands)
    * [6.1 `bpconfig` ⚙️](#61-bpconfig)
    * [6.2 `bptestbpcd` 🔌](#62-bptestbpcd)
    * [6.3 `bpps` 📋](#63-bpps)
    * [6.4 `bplist` 📝](#64-bplist)
    * [6.5 `bperror` ⚠️](#65-bperror)
    * [6.6 `bprestore` ⏪](#66-bprestore)
    * [6.7 `tpconfig` 📼](#67-tpconfig)
    * [6.8 `vmoprcmd` 🎛️](#68-vmoprcmd)

---

<a id="1-service-management"></a>
## 1. Service Management 🛠️
Restarting and managing NetBackup services is the first step in troubleshooting and maintenance. Always ensure no active backup or restore jobs are running for the affected server before restarting services.

<a id="11-linux-servers"></a>
### 1.1 Linux Servers (Master/Media/Client) 🐧
NetBackup services on Linux are typically managed via native NetBackup scripts.

* **Stop all NetBackup Services:**
    1. Log in as `root`.
    2. Run: `/usr/openv/netbackup/bin/goodies/netbackup stop`
    3. *Alternative (Force kill if hung):* `/usr/openv/netbackup/bin/bp.kill_all`
* **Start all NetBackup Services:**
    1. Log in as `root`.
    2. Run: `/usr/openv/netbackup/bin/goodies/netbackup start`
    3. *Alternative:* `/usr/openv/netbackup/bin/bp.start_all`
* **Verify Service Status:**
    1. Run: `/usr/openv/netbackup/bin/bpps -x`

<a id="12-windows-servers"></a>
### 1.2 Windows Servers (Master/Media/Client) 🪟
Windows services can be managed via the Services MMC or the command line interface.

* **Stop all NetBackup Services (Command Line):**
    1. Open Command Prompt as Administrator.
    2. Navigate to: `<Install_Path>\Veritas\NetBackup\bin\`
    3. Run: `bpdown -v -f` (The `-f` flag forces the shutdown without prompting).
* **Start all NetBackup Services (Command Line):**
    1. Open Command Prompt as Administrator.
    2. Navigate to: `<Install_Path>\Veritas\NetBackup\bin\`
    3. Run: `bpup -v -f`
* **Verify Service Status:**
    1. Open `services.msc`.
    2. Check services starting with "NetBackup" (e.g., NetBackup Client Service, NetBackup Request Daemon). Ensure their status is "Running".

[⬆️ Back to Top](#table-of-contents)

---

<a id="2-configuring-file-system-backups"></a>
## 2. Configuring File System Backups 📁

<a id="21-windows-file-system-backup"></a>
### 2.1 Windows File System Backup 🪟
* **Step 1: Create the Policy**
    1. Open the NetBackup Administration Console.
    2. Right-click **Policies** -> **New Policy**. Name it according to your naming convention (e.g., `PROD_WIN_FS_01`).
    3. In the **Attributes** tab, set Policy Type to **MS-Windows**.
    4. Select the appropriate Policy Storage Unit or Storage Lifecycle Policy (SLP).
* **Step 2: Define Schedules**
    1. Go to the **Schedules** tab -> **New**.
    2. Create a **Full** backup schedule (e.g., Weekly on Weekends) and a **Cumulative/Differential Inc** schedule (e.g., Daily on Weekdays).
    3. Set the retention periods for each schedule.
* **Step 3: Add Clients**
    1. Go to the **Clients** tab -> **New**.
    2. Type the exact hostname or FQDN of the Windows server. Set the OS/Hardware to Windows/x64.
* **Step 4: Backup Selections**
    1. Go to the **Backup Selections** tab.
    2. Add `ALL_LOCAL_DRIVES` for a full system backup, or specific paths like `D:\Data\` if you only need certain folders.

<a id="22-linux-file-system-backup"></a>
### 2.2 Linux File System Backup 🐧
* **Step 1: Create the Policy**
    1. Create a New Policy. Name it appropriately (e.g., `PROD_LINUX_FS_01`).
    2. In the **Attributes** tab, set Policy Type to **Standard**.
    3. Assign the target Storage Unit/SLP.
* **Step 2: Define Schedules**
    1. Create Full and Incremental schedules mirroring your organization's RPO/RTO requirements.
* **Step 3: Add Clients**
    1. Add the Linux client FQDN in the **Clients** tab.
* **Step 4: Backup Selections**
    1. Specify paths like `/` (root) or specific directories like `/var/www/`. 
    2. *Note:* Ensure you check the "Cross mount points" attribute in the Attributes tab if you need to back up nested mounted file systems.

[⬆️ Back to Top](#table-of-contents)

---

<a id="3-configuring-database-backups"></a>
## 3. Configuring Database Backups 🗄️

<a id="31-microsoft-sql-server-backup"></a>
### 3.1 Microsoft SQL Server Backup 📊
SQL backups require the NetBackup MS SQL Client application on the target database machine to generate the necessary scripts.

* **Step 1: Generate the Batch File (.bch)**
    1. Log into the target SQL Server.
    2. Open the **NetBackup MS SQL Client** application.
    3. Go to **File** -> **Backup SQL Server objects**.
    4. Select the databases to back up, choose the backup type (Full, Diff, Log), and save the batch file (e.g., `C:\NetBackup_Scripts\SQL_Full.bch`).
* **Step 2: Create the NetBackup Policy**
    1. In the Administration Console, create a new policy.
    2. Set Policy Type to **MS-SQL-Server**.
    3. Assign your Storage Unit. 
    4. Create an **Application Backup** schedule (defines retention) and an **Automatic Backup** schedule (defines when the script actually runs).
* **Step 3: Add Client and Selection**
    1. Add the SQL Server in the **Clients** tab.
    2. In the **Backup Selections** tab, enter the exact file path to the batch file created in Step 1 (e.g., `C:\NetBackup_Scripts\SQL_Full.bch`).

<a id="32-oracle-rman-backup"></a>
### 3.2 Oracle RMAN Backup 🛢️
Oracle backups use the NetBackup for Oracle agent, which integrates directly with Oracle RMAN by linking the NetBackup libraries to Oracle.

* **Step 1: Link Oracle to NetBackup**
    1. Log into the Linux Oracle server as the `oracle` user.
    2. Run the linking script: `/usr/openv/netbackup/bin/oracle_link`
* **Step 2: Prepare RMAN Scripts**
    1. Create an RMAN shell script (e.g., `/u01/app/oracle/scripts/rman_full.sh`) that calls `rman target /` and executes the backup commands.
    2. Ensure the script allocates channels of type `SBT_TAPE` and specifies the NetBackup library via parms: `parms 'ENV=(NB_ORA_POLICY=YourPolicyName,NB_ORA_SERV=MasterServerName)'`.
* **Step 3: Create the NetBackup Policy**
    1. Create a new policy and set Policy Type to **Oracle**.
    2. Configure Storage, and set up both an **Application Backup** and **Automatic Backup** schedule.
* **Step 4: Add Client and Selection**
    1. Add the Oracle server to the **Clients** tab.
    2. In the **Backup Selections** tab, specify the absolute path to the RMAN shell script you created in Step 2.

[⬆️ Back to Top](#table-of-contents)

---

<a id="4-client-package-installation"></a>
## 4. Client Package Installation 📦
Installing the NetBackup client is required on any target machine you wish to back up using a standard file system or application agent policy.

<a id="41-windows-client-installation"></a>
### 4.1 Windows Client Installation 🪟
**Prerequisites:** Ensure you have Administrator privileges and that the required firewall ports (1556, 13724) are open between the client and the Master/Media servers.

* **Step 1: Launch the Installer**
    1. Copy the NetBackup Windows installation media to the target server.
    2. Extract the package and run `Browser.exe` (or `setup.exe` in the `x64` directory).
    3. Select **Installation** -> **NetBackup Client Software Installation**.
* **Step 2: Configuration Prompts**
    1. Accept the End User License Agreement (EULA).
    2. Choose **Typical** installation (installs to default `C:\Program Files\Veritas\`).
    3. **Important:** When prompted for server details:
        * **Master Server Name:** Enter the exact FQDN of your NetBackup Master Server.
        * **Client Name:** Enter the exact FQDN or hostname of the server you are installing this on. (Ensure this matches how DNS resolves it).
* **Step 3: Service Account**
    1. By default, the NetBackup Client Service runs as the `Local System` account. For standard file system backups, this is sufficient. (SQL or Exchange backups may require a dedicated service account with specific database privileges).
* **Step 4: Verification**
    1. Click **Install** and wait for completion.
    2. Open `services.msc` and verify the **NetBackup Client Service** is running.

<a id="42-linux-client-installation"></a>
### 4.2 Linux Client Installation 🐧
**Prerequisites:** Root privileges are required. Ensure sufficient space in `/usr/openv/` (or create a symlink if installing to another partition).

* **Step 1: Extract and Execute**
    1. Transfer the NetBackup Linux client tar file to the server (e.g., to `/tmp/`).
    2. Extract the tarball: `tar -xvf NetBackup_Client_Linux.tar.gz`
    3. Navigate to the extracted directory and run the install script: `./install`
* **Step 2: Answer Prompts**
    1. Do you wish to continue? `y`
    2. Do you want to install the NetBackup client software for this client? `y`
    3. Enter the name of the NetBackup Master Server: `[Enter Master Server FQDN]`
    4. Enter the name of the NetBackup Client: `[Enter this server's FQDN]`
* **Step 3: Verification**
    1. Once the script finishes, verify the processes are running: `/usr/openv/netbackup/bin/bpps -x`
    2. You should see `bpcd` and `vnetd` listening.

[⬆️ Back to Top](#table-of-contents)

---

<a id="5-connectivity-and-communication-tests"></a>
## 5. Connectivity and Communication Tests 🌐
NetBackup relies heavily on forward and reverse DNS resolution and specific port communication (primarily PBX port 1556 and VNETD port 13724). Use the following tests to verify bi-directional communication between Master, Media, and Client servers, as well as application-specific connectivity.

<a id="51-master-to-client"></a>
### 5.1 Master Server -> Client Connectivity Tests (Windows & UNIX/Linux) 📡
Run these commands from the **Master Server** to verify it can reach and resolve the target client.

* **1. Network & Port Check:**
    * *Windows (PowerShell):* `Test-NetConnection -ComputerName <Client_FQDN> -Port 1556`
    * *Linux:* `nc -zv <Client_FQDN> 1556` or `telnet <Client_FQDN> 1556`
* **2. Hostname Resolution:** Verify the IP address the master resolves for the client.
    * Command: `bpclntcmd -hn <Client_FQDN>`
* **3. IP Resolution:** Verify the hostname the master resolves for the client's IP.
    * Command: `bpclntcmd -ip <Client_IP>`
* **4. Ping the Client Daemon:** Verifies the master can communicate directly with the `bpcd` (NetBackup Client Daemon) process running on the client.
    * Command: `bping -c <Client_FQDN>`

<a id="52-client-to-master"></a>
### 5.2 Client -> Master Server Connectivity Tests (Windows & UNIX/Linux) 🔙
Run these commands from the **Client Server** to verify it can reach, resolve, and be recognized by the Master server.

* **1. Network & Port Check:**
    * *Windows (PowerShell):* `Test-NetConnection -ComputerName <Master_FQDN> -Port 1556`
    * *Linux:* `nc -zv <Master_FQDN> 1556` or `telnet <Master_FQDN> 1556`
* **2. Hostname Resolution:** Verify the IP address the client resolves for the master.
    * Command: `bpclntcmd -hn <Master_FQDN>`
* **3. Check Master Server Recognition:** This is the most critical client-side test. It asks the master server, "Who do you think I am?"
    * Command: `bpclntcmd -pn`
    * *Expected Result:* The master server should respond confirming the client's name and IP address (e.g., `expecting response from server MASTER01... client_fqdn client_fqdn IP_Address port_number`).

<a id="53-application-level"></a>
### 5.3 Application-Level Connectivity Tests (SQL & Oracle RMAN) 🧩
Even if OS-level connectivity is fine, the specific database agents must be able to communicate with both the database and NetBackup.

* **Microsoft SQL Server (Run on the SQL Client):**
    * **Test 1: NetBackup MS SQL GUI Test:** Open the *NetBackup MS SQL Client* application. Go to **File** -> **Backup SQL Server objects**. If you can successfully expand the instance and view the databases, the NetBackup service account has the correct permissions to talk to SQL.
    * **Test 2: ODBC Connection:** Ensure the `NetBackup Client Service` is running under an account with `sysadmin` privileges in SQL. You can verify native SQL connectivity using: `sqlcmd -S <ServerName>\<InstanceName> -E`
* **Oracle RMAN (Run on the Oracle Linux Client):**
    * **Test 1: The `sbttest` Utility:** This tests the physical link between Oracle and the NetBackup media manager API without actually running a backup.
        * Log in as the `oracle` user.
        * Command: `sbttest test_file_name -libname /usr/openv/netbackup/bin/libobk.so`
        * *Expected Result:* Should return `The sbt function pointers are loaded from...` and report a successful test.
    * **Test 2: RMAN Channel Allocation Test:** Verifies RMAN can successfully allocate a tape channel via NetBackup.
        * Log in as the `oracle` user and open RMAN: `rman target /`
        * Run: 
          ```sql
          run {
            allocate channel t1 type 'SBT_TAPE';
            release channel t1;
          }
          ```
        * *Expected Result:* RMAN should report `allocated channel: t1` and then `released channel: t1` without any `ORA-27211` or `ORA-19511` media management errors.

<a id="54-clearing-host-cache"></a>
### 5.4 Clearing the Host Cache 🧹
If you recently updated DNS or a host file, NetBackup might be caching old IP addresses. Clear the cache on the affected machines (Master or Client) to force a new lookup.
* Command (Windows/Linux): `bpclntcmd -clear_host_cache`

[⬆️ Back to Top](#table-of-contents)

---

<a id="6-essential-commands"></a>
## 6. Essential NetBackup Command Line Utilities (Quick Reference) 💻
In addition to the commands mentioned above, these are the most frequently used commands by storage and backup administrators across Windows and UNIX/Linux platforms for daily tasks and troubleshooting. 

*Command Locations:*
* **UNIX/Linux:** `/usr/openv/netbackup/bin/admincmd/` (most admin commands) or `/usr/openv/netbackup/logs/bplist/` (logging commands)
* **Windows:** `<install_path>\NetBackup\bin\admincmd\`

<a id="61-bpconfig"></a>
### 6.1 `bpconfig` ⚙️
Used to configure or display global configuration attributes for NetBackup.
* **Example (Display attributes):** `bpconfig -U`
* **Output:** Shows details like Admin Mail Address, Job Retry Delay, Max Simultaneous Jobs, Keep Error/Debug Logs, etc.

<a id="62-bptestbpcd"></a>
### 6.2 `bptestbpcd` 🔌
Tests connections to clients. Unlike ping or nslookup, it tests for connection to a NetBackup Client using client connection options at the NetBackup communication level.
* **Example:** `bptestbpcd -host <client_name> -connect_options 0 0 2 0 0 2`
* **Output:** Reports information about established sockets between the NetBackup server and the `bpcd` daemon.

<a id="63-bpps"></a>
### 6.3 `bpps` 📋
Lists all process statistics for the NetBackup processes running on your system.
* **Syntax:** `bpps [-a | -x]` 
    * `-a`: Includes Media Manager processes.
    * `-x`: Includes Media Manager processes and extra shared processes (e.g., `pbx_exchange`).
* **Example:** `bpps -x`

<a id="64-bplist"></a>
### 6.4 `bplist` 📝
Displays information on the files and directories that were backed up or archived on the NetBackup server. Use options like `-A`, `-B`, `-C` to filter the desired output.
* **Example:** `bplist -l -R /home/user`
* **Output:** Detailed list of backed up files, directories, and their permissions/timestamps.

<a id="65-bperror"></a>
### 6.5 `bperror` ⚠️
Critical for troubleshooting. Used by administrators to display NetBackup status, troubleshooting information, or specific entries from the NetBackup error catalog.
* **Example (Report problems in the last 24 hours):** `bperror -U -problems`
* **Output:** Shows timestamps, server name, and error text (e.g., "no storage units configured", "scheduler exiting").

<a id="66-bprestore"></a>
### 6.6 `bprestore` ⏪
Used to manually restore a backed-up or archived file or a list of files from the NetBackup Server. Can also restore files depending on a specific time period.
* **Example (Restore using a list):** `bprestore -f backup_list`

<a id="67-tpconfig"></a>
### 6.7 `tpconfig` 📼
Runs the tape configuration utility. Used to configure and manage robots, drives, drive arrays, drive paths, and hosts for use with NetBackup.
* **Example (Add a robot):** `tpconfig -add -robot 9 -robtype tld -cntlhost perch`

<a id="68-vmoprcmd"></a>
### 6.8 `vmoprcmd` 🎛️
Performs operator functions on drives. Useful for checking currently active tapes in the NetBackup server and managing drive statuses (setting drives up/down).
* **Example (Display drive status for all drives):** `vmoprcmd -d ds`

[⬆️ Back to Top](#table-of-contents)

---

