Provisioning a LUN on a Hitachi array is a highly structured process. Because Hitachi separates the logical volume creation from the host presentation layer, you have fine-grained control over exactly how the server sees the disk. 

Below is the exhaustive, step-by-step Standard Operating Procedure (SOP) for provisioning a 100 GB LUN to a new Linux server using Hitachi Device Manager - Storage Navigator (HDvM-SN).



---

## Pre-Requisites & Information Gathering
Before touching the array, gather this data:
1.  **Host WWNs:** The World Wide Names of the Linux server's Fibre Channel HBAs (e.g., `10:00:00:90:fa:xx:xx:xx`).
2.  **Target Ports:** Which Hitachi Front-End ports you will use (e.g., `1A`, `2A`, `3A`, `4A` for redundancy).
3.  **DP Pool:** Identify the Dynamic Provisioning (DP) Pool that has the required 100 GB of free space.

---

## Phase 1: SAN Zoning (Fabric Layer)
*Note: This is done on your Cisco MDS or Brocade switches, not the Hitachi array.*
1.  Log into your SAN fabric switches.
2.  Create zones pairing the Linux Host's HBA WWNs with the chosen Hitachi Front-End Target Ports.
3.  Add the zones to the active zoneset and activate/commit the configuration. 
*Hitachi arrays will not auto-discover a host WWN until it is actively zoned and logged into the fabric.*

---

## Phase 2: Host Group Creation & WWN Registration
*This maps the server's identity to the array.*

1.  **Navigate to Host Groups:**
    * Open Storage Navigator.
    * Go to **Ports/Host Groups/iSCSI Targets** in the left-hand Explorer pane.
    * Select the specific port you zoned the host to (e.g., Port `1A`).
2.  **Create Host Group:**
    * Click **Create Host Groups**.
    * **Host Group Name:** Enter a standard naming convention (e.g., `LNX_DB_SRV01`).
3.  **Set the Host Mode (Crucial Step):**
    * In the **Host Mode** dropdown, select `00 [Standard]`. This is the standard setting for generic Linux systems (RHEL, SLES, Ubuntu).
    * *Optional but common:* Click **Host Mode Options (HMO)**. For modern Linux environments using ALUA (Asymmetric Logical Unit Access) multipathing, you often need to enable **HMO 54** (Support for VAAI/ALUA) or check your specific OS matrix for Hitachi best practices.
4.  **Add Host WWNs:**
    * In the same window, look at the **Available Hosts** section. If your zoning is correct, the Linux server's WWNs will appear here automatically.
    * Select the WWNs and add them to the **Selected Hosts** list.
    * If they don't appear, you can manually type the WWNs by clicking **Add New Host**.
5.  **Submit:**
    * Click **Add**, then click **Finish**.
    * Review the summary screen and click **Apply** to execute the task on the array.
    * *Repeat Phase 2 for the redundant port (e.g., Port `2A`) to ensure multipath connectivity.*

---

## Phase 3: DP-VOL (LUN) Creation
*This carves the actual 100 GB chunk of storage.*

1.  **Navigate to LDEVs:**
    * Go to **Logical Devices** in the left-hand Explorer pane.
    * Click **Create LDEVs**.
2.  **Configure LDEV Parameters:**
    * **LDEV Type:** Select `DP-VOL`.
    * **Capacity:** Enter `100` and select `GB` from the dropdown.
    * **Number of LDEVs:** `1`.
    * **LDEV ID:** Leave as `Initial ID` (the array will assign the next available hex ID, e.g., `00:0A`), or manually specify one based on your site's numbering scheme.
    * **Provisioning Type:** Selected Pool should be your target DP Pool (e.g., `Pool 0`).
3.  **Format and Naming:**
    * **LDEV Name:** Assign a descriptive name (e.g., `LNX_DB_SRV01_DATA_100G`).
    * **Format Type:** Select `No Format` (Hitachi DP-VOLs do not require formatting at the array level upon creation; the host OS will format it).
4.  **Submit:**
    * Click **Add**, verify the details, click **Finish**, and then click **Apply**.

---

## Phase 4: LUN Mapping (Adding the LUN Path)
*This connects the 100 GB volume to the Linux Host Group.*

1.  **Navigate to LUN Mapping:**
    * Go back to **Logical Devices**.
    * Select the newly created 100 GB LDEV (you can search by its LDEV Name or ID).
    * Click **Add LUN Paths**.
2.  **Select Target Host Groups:**
    * A window will pop up showing all ports and Host Groups.
    * Select the Host Groups you created in Phase 2 (e.g., `LNX_DB_SRV01` on Port `1A` and Port `2A`).
3.  **Assign LUN ID:**
    * You can choose `Auto` (the array will assign the next available LUN number, usually starting at `0000`).
    * Or choose `Manual` to specify a LUN ID (e.g., `0001`). Make sure it is consistent across all paths.
4.  **Submit:**
    * Click **Add**, then **Finish**, then **Apply**. The array is now actively presenting the storage to the fabric.

---

## Phase 5: Linux Host-Side Discovery
*The storage admin often guides the Linux admin through this, or you may do it yourself if you hold both roles.*

1.  **Rescan the SCSI Bus:**
    * SSH into the Linux server.
    * Run the rescan script (part of `sg3_utils`): 
        `/usr/bin/rescan-scsi-bus.sh -a`
    * *Alternative manual method:* Loop through the hosts and issue a scan:
        `for host in /sys/class/scsi_host/host*/scan; do echo "- - -" > $host; done`
2.  **Verify Multipathing:**
    * Run `multipath -ll` to verify the new 100 GB disk is visible across multiple paths (you should see multiple active paths depending on how many ports you mapped).
    * The disk will appear as a `mapper` device (e.g., `/dev/mapper/mpatha`).
3.  **Format and Mount:**
    * Create a filesystem: `mkfs.xfs /dev/mapper/mpatha` (or `ext4`).
    * Create a mount point: `mkdir /data`
    * Mount the disk: `mount /dev/mapper/mpatha /data`
    * Add to `/etc/fstab` for persistence across reboots.

---


