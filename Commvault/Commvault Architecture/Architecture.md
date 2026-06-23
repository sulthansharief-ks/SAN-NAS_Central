
# 🚀 Commvault Enterprise Backup: Complete Architecture Guide

Welcome to Day 1 of Commvault! This master guide breaks down the physical, logical, and network architecture of a CommCell environment. Whether your environment runs on Windows or Linux, this guide explains the services, binaries, and network traffic required to keep your data safe.

---

## 📑 Table of Contents
1. [What is Commvault?](#what-is-commvault)
2. [Core Architecture: The CommCell Environment](#core-architecture-the-commcell-environment)
3. [The CommServe (Command & Control)](#the-commserve)
4. [The MediaAgent (Data Mover)](#the-mediaagent)
5. [The Client & Agents (The Source)](#the-client-agents)
6. [Logical Concepts: How Data is Managed](#logical-concepts)
7. [Network Communication & Port Architecture](#network-communication)
8. [The Backup Communication Flow (Step-by-Step)](#backup-flow)
9. [The Linux Administrator's Toolkit (CLI)](#linux-cli)

---

## <a id="what-is-commvault"></a>1. What is Commvault?
Commvault is an enterprise-level data protection and information management software platform. It is designed to back up, recover, archive, and manage data across on-premises, cloud, and hybrid environments from a single unified interface.

## <a id="core-architecture-the-commcell-environment"></a>2. Core Architecture: The CommCell Environment
The foundation of Commvault is the **CommCell**. Think of it as a centralized management domain containing all the physical and logical components that protect your data. A CommCell requires three physical tiers: a CommServe, at least one MediaAgent, and Clients.

---

## <a id="the-commserve"></a>🧠 3. The CommServe (Command & Control)
Think of the CommServe as the **Chief Executive Officer** of your backup environment. It does *not* touch or store the actual backup data. Instead, it coordinates all activities, maintains the schedules, and holds the master record of everything that happens.

**📦 Key Components Installed:**
* **Database (🗄️):** Hosts the `CommServ` database. This is the holy grail of your environment—it contains all configurations, job histories, and metadata.
    * *Windows:* Runs on **Microsoft SQL Server**.
    * *Linux:* In modern deployments, this runs on **PostgreSQL** (or connects to a designated DB server).
* **Web Server / Command Center (🌐):** Serves the modern, web-based administrative UI (Command Center).
    * *Windows:* Runs via **IIS** and MongoDB.
    * *Linux:* Runs via Linux-native web services (like **Tomcat**).
* **Workflow Engine (⚙️):** Automates complex administrative tasks.

**🛠️ Core Services / Processes Running:**
| Process Purpose | Windows Service / Binary | Linux Daemon | Description |
| :--- | :--- | :--- | :--- |
| **Master Launcher** | *(Uses Windows Services OS)* | `cvlaunchd` 🚀 | **Unique to UNIX/Linux.** The master launcher daemon. Its sole job is to spawn, monitor, and keep all other Commvault processes alive. |
| **Communications** | `GxCVD` / `cvd.exe` 📡 | `cvd` 📡 | The foundational network daemon. It listens for incoming connections and wakes up/passes instructions to other processes as needed. |
| **Event Manager** | `GxEvMgrS` / `evmgrs.exe` 🧠 | `evmgrs` 🧠 | The core intelligence. It checks schedules, creates Job IDs in the Job Controller, and tracks the status of all operations. |
| **Job Manager** | `GxJobMgrS` / `jobmgr.exe` 📋 | `jobmgr` 📋 | Handles the execution and state tracking of active jobs (Running, Pending, Waiting). |
| **Firewall/Network** | `GxFWD` / `cvfwd.exe` 🛡️ | `cvfwd` 🛡️ | Manages encrypted network tunneling and routing between components. |

---

## <a id="the-mediaagent"></a>🏋️‍♂️ 4. The MediaAgent (Data Mover)
If the CommServe is the CEO, the MediaAgent (MA) is the **Fleet of Delivery Trucks**. The MA does the heavy lifting: receiving data from clients, deduplicating it, compressing it, and writing it to storage (Disk, Tape, or Cloud).

**📦 Key Components Installed:**
* **MediaAgent Software (🚚):** The core binaries responsible for data movement.
* **Deduplication Database / DDB (✂️):** Hosted on incredibly fast local storage (SSDs on Windows, or NVMe/SSD mount points like `/mnt/ddb/` on Linux). It tracks data block hashes to ensure the system never writes the same block of data twice.
* **Index Cache (🗂️):** A local scratchpad directory where the MA temporarily stores metadata (what files were backed up and where they live) before uploading it to the CommServe and storage.

**🛠️ Core Services / Processes Running:**
| Process Purpose | Windows Service / Binary | Linux Daemon | Description |
| :--- | :--- | :--- | :--- |
| **Master Launcher** | *(Uses Windows Services OS)* | `cvlaunchd` 🚀 | Keeps the MediaAgent processes running. |
| **Communications** | `GxCVD` / `cvd.exe` 🌊 | `cvd` 🌊 | Handles the massive streams of backup data pouring in from the clients. |
| **Firewall/Network** | `GxFWD` / `cvfwd.exe` 🚇 | `cvfwd` 🚇 | Maintains the secure network tunnel back to the CommServe and clients. |
| **Deduplication** | `SIDB.exe` / `SIDB2.exe` 🔍 | `SIDB2` 🔍 | Single Instance Database engine. Performs the heavy CPU work of deduplication hash lookups against the DDB. |
| **Data/Mount Mover** | `cvNetworkMover` 🏎️ | `CVMountd` 💾 | *Windows:* Pipeline processes spawned dynamically during a backup. *Linux:* Media Mount Manager interacting directly with attached Linux OS storage targets. |

---

## <a id="the-client-agents"></a>🏢 5. The Client & Agents (The Source)
The client is the actual server, virtual machine, database, or container node you are trying to protect.

**📦 Key Components Installed:**
* **Base Package (🧱):** The foundational software/libraries allowing the machine to talk to the CommCell.
* **Intelligent Data Agent / iDA (🕵️‍♂️):** The specialized application hook (e.g., Windows/Linux File System Agent, SQL Server Agent, Oracle Agent, PostgreSQL Agent, or Virtual Server Agent).

**🛠️ Core Services / Processes Running:**
| Process Purpose | Windows Service / Binary | Linux Daemon | Description |
| :--- | :--- | :--- | :--- |
| **Master Launcher** | *(Uses Windows Services OS)* | `cvlaunchd` 🚀 | Spawns the client-side services. |
| **Communications** | `GxCVD` / `cvd.exe` 📤 | `cvd` 📤 | Packages the application data and ships it over the network to the MediaAgent. |
| **Firewall/Network** | `GxFWD` / `cvfwd.exe` 🛡️ | `cvfwd` 🛡️ | Maintains the communication tunnel. |
| **App-Specific** | `clbackup.exe`, `vsbkp.exe` 🔧 | `ClMgrS`, `clbackup` 🔧 | Temporary processes spawned during a job. *Linux `ClMgrS`* crawls the file system to find or recover actual files. |

---

## <a id="logical-concepts"></a>6. Logical Concepts: How Data is Managed

Once the infrastructure is built, beginners need to know how Commvault logically organizes a backup.

* **Backup Set:** A logical grouping of subclients.
* **Subclient:** The actual target of what is being backed up. The Subclient defines the specific rule (e.g., "Only back up the `C:\Data` folder" or "Only back up the `/etc/` directory").
* **Plans / Storage Policies:** The lifecycle management rulebook. It dictates **Where** the data goes (which MediaAgent and Library), **How long** it stays there (Retention Policy), and **How often** it runs (Schedule).
* **Deduplication:** A critical storage optimization feature that identifies and eliminates redundant blocks of data across backups. It drastically reduces the amount of storage space required and decreases network traffic.

---

## <a id="network-communication"></a>🚦 7. Network Communication & Port Architecture

Commvault uses a highly resilient network routing system to easily pass through firewalls (`iptables`, `firewalld`, or Windows Firewall). Here are the golden rules of Commvault network ports:

| Port | Service | Protocol | Purpose & Flow 🛣️ |
| :--- | :--- | :--- | :--- |
| **8400** | `cvd` / `cvd.exe` | TCP | **Data Traffic (The Highway) 🏎️:** The default port for heavy data transfer. For maximum speed, this should be open bidirectionally between Clients and MediaAgents. |
| **8403** | `cvfwd` / `cvfwd.exe` | TCP | **Tunnel / Control Traffic (The Subway) 🚇:** **The most critical port.** It handles all command traffic (CommServe ↔ Client). If port 8400 is blocked by a firewall, Commvault seamlessly encapsulates the data and tunnels it through 8403 instead! |
| **8401** | `evmgrs` / `evmgrs.exe`| TCP | **Event Manager 👔:** Used primarily by administrative consoles (like the legacy Java GUI) to administer the CommServe. |

**☁️ External & Third-Party Ports:**
* **Command Center / Web Console:** TCP `443` (HTTPS).
* **VMware VSA:** TCP `443` (vCenter API) & TCP `902` (ESXi NFC traffic for reading VM disks via HotAdd/NBD transport).
* **Cloud Storage:** TCP `443` (Outbound from MediaAgent to AWS S3, Azure Blob, or Metallic Cloud Storage Service).

---

## <a id="backup-flow"></a>🎬 8. The Backup Communication Flow (Step-by-Step)

How does a backup actually happen? Let's walk through the exact sequence:

1.  **Wake Up (⏰):** The CommServe (`evmgrs` / `evmgrs.exe`) realizes it's 10:00 PM and a backup is scheduled. It assigns a Job ID in the Job Controller.
2.  **The Knock on the Door (🚪):** The CommServe talks to the Client over port **8403** and says, *"Wake up the File System Agent, it's time to work."*
3.  **Resource Check (🚦):** The CommServe checks in with the MediaAgent: *"Is your Deduplication Database online? Do we have free space on the Disk Library?"* 4.  **The Data Pipeline (🌊):** * The Client reads the files from its local disk.
    * The Client (`cvd` / `cvd.exe`) opens a connection to the MediaAgent (`cvd` / `cvd.exe` on port **8400**) and starts blasting the data over. *(If 8400 fails, it sneaks the data through the 8403 tunnel).*
5.  **Shrink & Store (📦):** The MediaAgent receives the data stream, checks it against `SIDB2` / `SIDB.exe` to strip out duplicates, compresses it, and writes the unique blocks to the storage library.
6.  **Mission Accomplished (✅):** The MediaAgent updates its local Index Cache. Both the MediaAgent and Client send a final "thumbs up" back to the CommServe, which turns the job **Green** in the Job Controller!

---

## <a id="linux-cli"></a>💻 9. The Linux Administrator's Toolkit (CLI)

Because Linux environments lack a graphical Windows Task Manager or `services.msc`, beginners need to know how to control Commvault from the Linux terminal. 

The master control utility is typically located at: `/opt/commvault/Base/commvault` *(Note: your path may vary slightly based on installation).*

**Essential Commands to Teach Your Beginners:**

* **Check the status of all Commvault daemons:**
    `commvault -all status`
* **Stop all Commvault services cleanly:**
    `commvault -all stop`
* **Start all Commvault services:**
    `commvault -all start`
* **Restart all services (Useful after OS patching):**
    `commvault -all restart`
* **Verify processes at the OS level:**
    If you want to prove to beginners that the processes are running under the hood, use standard Linux utilities:
    `ps -ef | grep -i cvlaunchd`
    `ps -ef | grep -i cvd`
