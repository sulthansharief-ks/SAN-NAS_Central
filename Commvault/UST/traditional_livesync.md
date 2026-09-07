# Lecture Notes: Commvault CommServe Live Sync Implementation

## 1. Introduction and Terminology
*   **Feature Name Change:** What was previously known as **CV Failover** (prior to v11 Service Pack 19) is now officially termed **CommServe Live Sync**.
*   **Prerequisites:** It is highly recommended to understand the theoretical architecture of CommServe Failover before attempting a practical deployment.
*   **Core Concept:** The setup utilizes an active-passive node failover mechanism coupled with DNS manipulation to seamlessly redirect clients to a Standby/DR CommServe if the Primary CommServe goes offline.

## 2. Infrastructure & DNS Setup (Lab Environment)
Before installing any Commvault software, the underlying DNS architecture must be staged correctly using a "Floating Name".

*   **Primary CommServe Server:**
    *   **FQDN:** `commserv.cv.lab`
    *   **IP Address:** Pointed to the Primary Site IP (e.g., `10.0.0.x`).
*   **Standby/DR CommServe Server:**
    *   **FQDN:** `drcv.cv.lab`
    *   **IP Address:** Pointed to the DR Site IP.
*   **The Floating CommServe Name (Crucial):**
    *   **FQDN:** `cvfloat.cv.lab`
    *   **Role:** This is a dedicated DNS A-Record (floating name) created in your Active Directory/DNS environment.
    *   **Initial State:** During normal operations, `cvfloat.cv.lab` points directly to the IP address of the **Primary CommServe**.

## 3. Deployment Phase 1: Installing Instance 1 (The CommServe)
When installing the base Commvault package (Instance 1) on both nodes, the naming convention is critical to trick the environment into treating both servers as a single entity.

*   **On the Primary CommServe:**
    *   **Client Name:** `cvfloat` (You MUST use the floating name, not the physical server name).
    *   **Host Name:** `commserv.cv.lab` (The actual physical FQDN of the primary server).
*   **On the Standby/DR CommServe:**
    *   **Client Name:** `cvfloat` (This must perfectly match the Primary's client name).
    *   **Host Name:** `drcv.cv.lab` (The actual physical FQDN of the DR server).

## 4. Deployment Phase 2: Installing Instance 2 (SQL iDA & CV Failover)
After the base CommServe is installed, a second instance (Instance 2) must be installed on **both** servers. This instance runs the SQL iDA (Intelligent Data Agent) and the CV Failover component to synchronize the backend databases.

*   **On the Primary CommServe (Instance 2):**
    *   **Packages:** Select SQL iDA and CV Failover.
    *   **Client Name:** `commserv_2` (Appended with `_2` to denote the second instance).
    *   **Host Name:** `commserv.cv.lab` (Actual FQDN).
    *   **CommServe Name (To Report To):** `cvfloat` (It will report back to Instance 1 via the DNS entry).
*   **On the Standby/DR CommServe (Instance 2):**
    *   **Packages:** Select SQL iDA and CV Failover.
    *   **Client Name:** `drcv_2` (Appended with `_2`).
    *   **Host Name:** `drcv.cv.lab` (Actual FQDN).
    *   **CommServe Name (To Report To):** `cvfloat`.

## 5. Failover Execution and Troubleshooting
During a disaster scenario, clients and MediaAgents will lose communication with the Primary CommServe. The video demonstrates this exact scenario:

*   **Symptom:** If you attempt to run a backup or check readiness on a MediaAgent while the primary server is down, the job will hang indefinitely. The readiness check will eventually report: *"Media Agent is not ready. Communication failure between the Media Agent and the CommServe."*
*   **Diagnosis:** Checking the MediaAgent properties reveals it is actively trying to contact `cvfloat`. Because the primary server is down, the DNS record is pointing to an unreachable IP.
*   **The Failover Action (The Fix):**
    1. Log into your Active Directory / DNS Server.
    2. Locate the A-Record for the floating name (`cvfloat.cv.lab`).
    3. Manually update the IP address of `cvfloat` to point to the **DR CommServe's IP Address**.
*   **Verification:** After updating DNS and allowing it to propagate, running "Check Readiness" from the DR CommServe will immediately show the MediaAgent as ready. Hung backups will resume normally.

## 6. Key Takeaways & Best Practices
*   **The "Floating Name" Rule:** When deploying *any* new client or MediaAgent in the environment, **always** specify the floating name (`cvfloat`) as the CommServe host, never the physical machine name.
*   **Massive Time Savings:** By strictly adhering to the floating name rule, a DR failover requires updating only *one* single DNS record. If physical hostnames were used during client deployment, an administrator would have to manually update the CommServe hostname on every single client in the enterprise.
*   **Failback Process:** Once the primary datacenter is restored, the failback process works exactly the same way in reverse. You log into Instance 2 of the primary server, trigger the failover task to synchronize the SQL databases back to primary, and revert the DNS record to the original primary IP.






According to the official Commvault documentation, performing a CommServe Live Sync failover is actually very straightforward once the initial setup is complete! 🚀 

There are two main scenarios: **Planned Failover** (for maintenance or testing) and **Unplanned Failover** (when your primary CommServe crashes). Here are the exact steps for how to perform it using the Commvault Process Manager, along with an explanation of what every option does.

### 🛠️ Step-by-Step Failover Process

**1. Open the Process Manager**
*   **For a Planned Failover:** You can log on to *either* the Primary or the Standby CommServe host.
*   **For an Unplanned Failover:** You must log on to the Standby CommServe host (since your Primary is down/inaccessible).
*   Go to the Start Menu -> Commvault -> **Process Manager**.
*   *Important Note:* Make sure you open the Process Manager associated with the **SQL Client Instance** (Instance 2 from the setup), not the base CommServe instance. 

**2. Navigate to the Failover Assistant**
*   Click on the **Failover Assistant** tab at the top of the Process Manager window. 

**3. Configure the Failover Options**
Here is exactly what each option means based on the Commvault docs:

*   🔽 **Failover To:** This is a dropdown menu where you select the name of the passive (Standby) node that you want to take over as the new active CommServe.
*   🔽 **Failover Type:** This dropdown defines exactly *how* the failover will behave. You will see these options:
    *   **Production:** 🚨 This is the main one! It initiates a true failover. The standby node will apply the latest SQL logs, start all Commvault services, and officially become your active production CommServe.
    *   **Production Maintenance:** 🔧 Use this if you are just doing OS patching or installing a Commvault Feature Release. It pauses schedules gracefully to let you do maintenance.
    *   **Test:** 🧪 Use this to verify your disaster readiness. It mounts the database and starts services on the standby node in an isolated way *without* shutting down or impacting your live primary CommServe.

**4. Initiate the Failover**
*   Select **Production** from the Failover Type list.
*   Click the **Initiate Failover** button.
*   A confirmation prompt will pop up. Type the word `confirm` in the box and click **OK**.

**5. Monitor the Process**
*   The Process Manager window will display a sequence of tasks (stopping services on the old primary, restoring the latest SQL database changes, and starting services on the new primary). 
*   *Fun fact from the docs:* Once it finishes, Commvault will automatically trigger a DDB Resynchronization job behind the scenes to make sure your Deduplication databases are perfectly aligned! 📊

### 🌐 The Final (and Most Crucial) Step: Update DNS
Because you set this up using a "Floating CommServe Name" (as discussed in your previous setup notes), the failover doesn't magically route the network traffic until you tell it to!

Once the Process Manager says the failover is complete:
1. Log into your Windows Active Directory / DNS server.
2. Find the A-Record for your floating CommServe name (e.g., `cvfloat.cv.lab`).
3. Change the IP address from the old Primary CommServe's IP to the new **Standby CommServe's IP**.
4. Flush your DNS (`ipconfig /flushdns`) and test it with a ping. 

Once that DNS change propagates, all your MediaAgents and Clients will seamlessly reconnect to the new CommServe, and your backups will resume normally! 🎉
