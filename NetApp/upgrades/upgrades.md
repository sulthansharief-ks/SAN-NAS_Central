# 🔄 NetApp ONTAP: Non-Disruptive Upgrade (NDU) SOP 🚀

A Non-Disruptive Upgrade (NDU) allows you to update the ONTAP OS across the entire cluster without taking applications offline. The cluster achieves this by automatically migrating data LIFs (IP addresses) and relying on SAN multipathing to keep data accessible while it reboots one node at a time.

> 🏷️ **Rule:** Replace placeholders like `<TARGET_VERSION>` and `<WEBSERVER_IP>` with your environment's details.

## 📑 Table of Contents
1. [🛑 Phase 1: Pre-Flight Checks & Planning](#phase-1)
2. [📥 Phase 2: Image Upload & Validation](#phase-2)
3. [🚀 Phase 3: Automated Execution](#phase-3)
4. [🕵️‍♂️ Phase 4: Post-Upgrade Validation](#phase-4)

---

<a id="phase-1"></a>
## 🛑 Phase 1: Pre-Flight Checks & Planning
*Perform these steps 1-2 weeks before the approved maintenance window.*

> 👥 **Responsible Team:** **Storage Team & Server/Virtualization Team**

**1. Generate an Upgrade Advisor Plan:**
Log into NetApp Active IQ, select your cluster, and generate an Upgrade Advisor report for your target ONTAP version. Read the release notes for known bugs affecting your specific hardware.

**2. Run Active IQ Config Advisor:**
Run the Config Advisor tool against the cluster. Resolve any cabling warnings, failed disks, or high-availability (HA) interconnect errors. **An NDU will fail if the cluster is not 100% healthy.**

**3. Verify SAN Multipathing (CRITICAL):**
If using iSCSI or Fibre Channel, you must verify with the VMware/Windows admins that Multipath I/O (MPIO) or ALUA is healthy and active. If a host only has a single path to a LUN, it *will* experience an outage when that node reboots.

**4. Suspend Heavy Background Jobs:**
Ensure there are no massive SnapMirror initializations, volume moves, or deduplication scans scheduled during the upgrade window.

---

<a id="phase-2"></a>
## 📥 Phase 2: Image Upload & Validation
*Perform these steps a few days before the upgrade.*

> 👥 **Responsible Team:** **Storage Team**

**1. Host the ONTAP Image:**
Download the target ONTAP `.tgz` image from the NetApp Support Site and place it on an internal HTTP or FTP server that the NetApp cluster can ping.

**2. Download the Image to the Cluster:**
```bash
cluster image package get -url http://<WEBSERVER_IP>/<ONTAP_IMAGE_NAME.tgz>
```

**3. Verify the Package Downloaded Successfully:**
```bash
cluster image package show-repository
```

**4. Run the Automated Pre-Update Validation:**
*This step simulates the upgrade logic and flags any issues that would block the upgrade (e.g., degraded aggregates, offline LIFs, unsupported CIFS sessions).*
```bash
cluster image validate -version <TARGET_VERSION>
```
*Wait for the validation to complete. If it returns any warnings or errors, you must resolve them before proceeding to Phase 3.*

---

<a id="phase-3"></a>
## 🚀 Phase 3: Automated Execution
*Perform these steps during your approved maintenance window.*

> 👥 **Responsible Team:** **Storage Team**

**1. Set Privilege Level:**
```bash
set -privilege advanced
```

**2. Trigger the Automated NDU:**
*The cluster will handle the failover, reboot, giveback, and waiting periods automatically.*
```bash
cluster image update -version <TARGET_VERSION>
```

**3. Monitor the Progress (Open a second SSH session):**
*Use this command to watch the cluster migrate LIFs, reboot Node 1, wait for quorum, and then proceed to Node 2.*
```bash
cluster image show-update-progress
```

> ⚠️ **Emergency Pause:** If something goes catastrophic on the application side during the upgrade, you can pause the NDU process:
> `cluster image pause-update`
> *(Note: You cannot pause a node while it is actively rebooting).*

---

<a id="phase-4"></a>
## 🕵️‍♂️ Phase 4: Post-Upgrade Validation
*Perform these steps immediately after the `show-update-progress` command reports complete.*

> 👥 **Responsible Team:** **Storage Team**

**1. Verify Cluster Version:**
Ensure all nodes in the cluster report the new target version.
```bash
version -node *
```

**2. Verify High Availability (HA):**
Ensure HA is fully re-enabled and storage failover is possible.
```bash
storage failover show
```

**3. Revert Network LIFs:**
During the upgrade, Data LIFs migrated to surviving nodes. Send them back to their home ports to restore load balancing.
```bash
network interface revert -vserver * -lif *
```

**4. Verify Network Port Health:**
Ensure all logical interfaces are `up` and on their home nodes.
```bash
network interface show -is-home false
```
*(This should return no entries if all LIFs reverted successfully).*
