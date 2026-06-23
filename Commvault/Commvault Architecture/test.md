You got it! Let’s get out of the theory and into the actual driver's seat.

When you are teaching absolute beginners how to build a CommCell from the ground up, you need to walk them through three distinct phases: **The Installer**, **The Command Center (Modern UI)**, and the **CommCell Console (Advanced UI)**.

Here is your exact, step-by-step installation and configuration playbook based on official Commvault deployment procedures.

---

## 🛠️ Phase 1: Running the Commvault Installer

Whether you are running `Setup.exe` on Windows or `./cvpkgadd` on Linux, the first thing your students need to know is *which boxes to check* during the installation.

When prompted to select packages, instruct your students to expand the **Server** category. To build a fully functional CommServe, they must select the following exact components:

* ☑️ **CommServe:** The core brain and database engine.
* ☑️ **Web Server:** The backend service required to host the web interfaces.
* ☑️ **Command Center:** The modern HTML5 administrative interface.
* ☑️ **Web Console:** The end-user interface (useful for self-service file restores).
* ☑️ **Workflow Engine:** Required for automating tasks and running Commvault-provided scripts.
* ☑️ **Metrics Server (Optional but highly recommended):** Used for advanced reporting and health checks.

**The Installation Flow:**

1. **Installation Path:** Choose a dedicated drive with fast I/O (e.g., `D:\Program Files\Commvault`).
2. **Database Path:** The installer will ask where to put the SQL/PostgreSQL database and disaster recovery (DR) backups. **Best Practice:** Put the database on an SSD, and ensure the DR path is separate from the installation drive.
3. **Summary & Install:** The installer will deploy the database, configure IIS/Tomcat, and start all the processes we discussed earlier.

---

## 🌐 Phase 2: Day 1 Configuration in the Command Center

Once the installer finishes, the system is alive, but it doesn't know *what* to do. You will log into the modern web UI to run the initial setup wizard.

**URL:** `https://<your-commserve-hostname>/commandcenter`

When you log in for the very first time with the default admin credentials, Commvault forces a **Core Setup Wizard**. Teach your students these exact steps:

### Step 1: Upload the License

* **Action:** Click **Upload License** and provide the `.xml` file provided by Commvault.
* **Why it matters:** The CommCell will not function or allow backups without a valid license tied to the CommServe's unique IP/Hostname.

### Step 2: Configure Storage

The CommServe needs to know where the data is going.

* **Action:** Go to **Storage** > **Disk** (or Cloud).
* **Input:** * Name the storage (e.g., `Primary_Disk_Library`).
* Select the **MediaAgent** that will control it.
* Provide the physical path (e.g., `E:\BackupData\` or a Linux mount point).
* *Note: This automatically creates the Deduplication Database (DDB) path as well.*



### Step 3: Create a Server Backup Plan

A Plan is the golden rulebook. Beginners must understand that Plans replace the old "Storage Policies and Schedules" method.

* **Action:** Go to **Manage** > **Plans** > **Create Server Backup Plan**.
* **Input:**
* **Storage:** Select the `Primary_Disk_Library` you just created.
* **RPO (Recovery Point Objective):** Set the schedule (e.g., Run an incremental backup every day at 9:00 PM).
* **Retention:** Tell Commvault how long to keep the data (e.g., Keep for 30 days).



### Step 4: Add a Client and Run a Backup

* **Action:** Go to **Protect** > **File Servers** > **Add Server**.
* **Input:** Push the "File System" software package to a target machine.
* **Final Step:** Associate that new server with the **Plan** you created in Step 3. The backup will now run automatically tonight!

---

## ⚙️ Phase 3: Essential Config in the CommCell Console (Java GUI)

While the Command Center is where administrators spend 95% of their time, the legacy **CommCell Console** is still required for deep, system-level configurations.

Instruct your students to open the CommCell Console, go to the **Home** ribbon, and click **Control Panel**. They must configure these two critical items:

### 1. CommServe Disaster Recovery (DR) Path

This is the most critical configuration in the entire environment. If the CommServe dies, this is how you survive.

* **Navigation:** Control Panel > **DR Backup**.
* **Action:** Go to the **Export Settings** tab.
* **Configuration:** Change the DR backup destination from a local drive to a **Network Share** or a **Cloud Bucket**.
* **Why:** If the physical CommServe burns down, a local DR backup burns with it. Exporting the database to an isolated network share ensures you can rebuild the CommServe tomorrow.

### 2. Email Server Configuration (SMTP)

You want Commvault to email you if something breaks, right?

* **Navigation:** Control Panel > **Email and Web Server Configuration**.
* **Action:** Enter the company's SMTP server details (e.g., `smtp.office365.com`), port (usually 25 or 587), and authentication credentials.
* **Test:** Click **Test Mail** to ensure the CommServe can successfully send alerts.

---

This flow takes your students directly from a blank server to a fully licensed, storage-aware, and DR-protected Commvault environment.

Which specific storage target (Disk, SAN/NAS, or Cloud like AWS/Azure) will your students be setting up first, so we can tailor the next module?
