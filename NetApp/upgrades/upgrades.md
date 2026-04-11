# 🛠️ NetApp ONTAP: Infrastructure Maintenance & Lifecycle SOP 🚀

Hardware degrades, bugs are discovered, and security vulnerabilities emerge. The "care and feeding" of your NetApp physical arrays through routine maintenance is what separates a stable enterprise environment from a fragile one. 

This detailed Standard Operating Procedure (SOP) covers the four pillars of NetApp physical and lifecycle maintenance: OS Upgrades, Hardware Break/Fix, Firmware patching, and Switch lifecycle.

> 🏷️ **Rule:** Replace placeholders like `<NODE>`, `<DISK>`, and `<VERSION>` with your specific hardware and software details. **Always execute these tasks during approved maintenance windows.**

## 📑 Table of Contents
1. [🔄 SOP 1: Non-Disruptive Upgrades (NDU) - ONTAP OS](#sop-1)
2. [⚙️ SOP 2: Hardware Replacement (Disks, PSUs, SFPs)](#sop-2)
3. [🧬 SOP 3: Firmware Updates (Disks, Shelves, SP/BMC)](#sop-3)
4. [🖧 SOP 4: Interconnect Switch Management & Upgrades](#sop-4)

---

<a id="sop-1"></a>
## 🔄 SOP 1: Non-Disruptive Upgrades (NDU) - ONTAP OS
A Non-Disruptive Upgrade (NDU) allows you to update the ONTAP OS across the entire cluster without taking applications offline. The cluster achieves this by automatically migrating data LIFs (IP addresses) and relying on SAN multipathing to keep data accessible while it reboots one node at a time.

### Phase 1: Planning & Pre-Checks (1-2 Weeks Prior)
**Responsible Team:** Storage Team & Server/Virtualization Team

1. **Generate an Upgrade Advisor Plan:** Log into NetApp Active IQ, select your cluster, and generate an Upgrade Advisor report. Read the release notes for known bugs.
2. **Run Config Advisor:** Run the Active IQ Config Advisor tool against the cluster. Resolve any cabling warnings, failed disks, or high-availability (HA) interconnect errors.
3. **Verify SAN Multipathing (CRITICAL):** If using iSCSI or Fibre Channel, verify with the VMware/Windows admins that Multipath I/O (MPIO) is healthy. If a host only has a single path to a LUN, it will experience an outage.

### Phase 2: Uploading the Image & Validation
1. **Download the Software:** Get the ONTAP `.tgz` image from the NetApp Support Site and host it on an internal HTTP/FTP server.
2. **Download to the Cluster:**
```bash
cluster image package get -url http://<YOUR_WEBSERVER>/<ONTAP_IMAGE.tgz>
```
3. **Run the Automated Pre-Update Check:**
```bash
cluster image validate -version <TARGET_VERSION>
```
*(Resolve any errors or warnings outputted by this command before proceeding).*

### Phase 3: Executing the Automated NDU
1. **Set Privilege Level:**
```bash
set -privilege advanced
```
2. **Trigger the Upgrade:**
```bash
cluster image update -version <TARGET_VERSION>
```
3. **Monitor the Progress:** Open a second SSH session to watch the process.
```bash
cluster image show-update-progress
```

### Phase 4: Post-Upgrade Checks
1. **Verify Versions:** Ensure all nodes are on the new version.
```bash
version -node *
```
2. **Revert Network LIFs:** Send Data LIFs back to their home ports.
```bash
network interface revert -vserver * -lif *
```

---

<a id="sop-2"></a>
## ⚙️ SOP 2: Hardware Replacement (Disks, PSUs, SFPs)
Physical components fail. NetApp AutoSupport usually opens a ticket and dispatches parts automatically, but you must safely swap them in the data center.

### Scenario A: Failed Disk Replacement
1. **Identify the Broken Disk:**
```bash
storage disk show -broken
```
2. **Turn on the Disk Fault LED:**
```bash
storage disk set-led -disk <DISK_NAME> -action on -duration 10
```
3. **Physical Swap:** Pull out the disk with the amber light. Wait 30 seconds. Insert the new disk fully.
4. **Assign Ownership:** Modern ONTAP usually auto-assigns the new disk, but verify and assign manually if needed.
```bash
storage disk show -unassigned
storage disk assign -disk <NEW_DISK_NAME> -node <NODE_NAME>
```

### Scenario B: Failed Power Supply (PSU) or Fan
1. **Verify the Failure:**
```bash
system node environment sensors show -node <NODE> -state !normal
```
2. **Physical Swap:** PSUs and Fans are hot-swappable. Unplug the power cable, pull the old PSU out, slide the new one in, and plug it back in.
3. **Verify Recovery:** Run the sensor command again; the state should return to `normal`.

---

<a id="sop-3"></a>
## 🧬 SOP 3: Firmware Updates (Disks, Shelves, SP/BMC)
Firmware updates prevent premature hardware failures and fix low-level hardware bugs. They are almost always non-disruptive.

### 3.1 Disk & Qualification Package (DQP) Updates
The DQP tells ONTAP how to interact with newly released drive models. Disk firmware updates fix drive-specific bugs.
1. **Update the DQP:** Download the `.zip` from NetApp, host it on your web server.
```bash
storage firmware download -node * -package-url http://<YOUR_WEBSERVER>/qual_devices.zip
```
2. **Update Disk Firmware:** Download the `all.zip` disk firmware package.
```bash
storage firmware download -node * -package-url http://<YOUR_WEBSERVER>/all.zip
```
*(ONTAP automatically applies disk firmware in the background sequentially to avoid taking down an aggregate).*

### 3.2 Disk Shelf Firmware
Updates the I/O modules on the external drive shelves.
1. **Download and Apply:**
```bash
storage firmware download -node * -package-url http://<YOUR_WEBSERVER>/<SHELF_FW.zip>
```
2. **Verify Installation:** Look for the shelf module firmware versions.
```bash
sysconfig -a
```

### 3.3 Service Processor (SP / BMC) Firmware
The SP/BMC allows you to out-of-band SSH into a node even if ONTAP has completely crashed. Keep it updated.
1. **Update the SP Image:**
```bash
system node service-processor image update -node <NODE> -package http://<YOUR_WEBSERVER>/<SP_IMAGE.zip>
```
2. **Monitor the Update:**
```bash
system node service-processor image update-progress show
```

---

<a id="sop-4"></a>
## 🖧 SOP 4: Interconnect Switch Management & Upgrades
Cluster and Management switches (Cisco Nexus or Broadcom) require patching just like the NetApp nodes. Upgrading them requires careful orchestration.

**CRITICAL WARNING:** NEVER upgrade both cluster switches at the same time. If the cluster network goes down entirely, all nodes will immediately panic and halt to prevent data corruption.

### Phase 1: Pre-Checks & Backups
1. **Verify Cluster Network Health:** Both switches must be fully operational.
```bash
system cluster-switch show
network port show -ipspace Cluster
```
2. **Backup Switch Config:** Use an automated NCM tool, or verify ONTAP's native backup:
```bash
set -privilege advanced
system cluster-switch config-backup show
```

### Phase 2: Upgrading Switch 2 (The Secondary)
1. **Console In:** Connect to the console or SSH of Switch 2.
2. **Transfer Image:** Copy the new firmware to the switch's bootflash.
3. **Execute Upgrade:** (Cisco Nexus Example - In-Service Software Upgrade)
```cisco
install all nxos bootflash:<NEW_IMAGE.bin>
```
4. **Wait & Validate:** The switch will reboot. Wait 5-10 minutes.
5. **NetApp Validation:** Verify ONTAP sees the switch as healthy and all links are back UP before touching Switch 1.
```bash
system cluster-switch show
```

### Phase 3: Upgrading Switch 1 (The Primary)
1. Ensure Switch 2 has been fully operational and routing traffic for at least 15 minutes.
2. Repeat the exact same firmware transfer and installation process on Switch 1.
3. Perform a final cluster ping test to ensure MTU 9000 and cluster traffic is passing cleanly across both upgraded switches.
```bash
cluster ping-cluster -node *
```
