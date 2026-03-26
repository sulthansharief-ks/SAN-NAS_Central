# 💽 NetApp ONTAP: The Definitive Physical Disk Operations Cheat Sheet 🛠️

Managing physical disks is the bedrock of storage administration. Whether you are assigning new shelves, proactively failing a dying drive, or securely wiping retired hardware, ONTAP provides granular control over the physical media. 

This cheat sheet covers the complete lifecycle of disk operations in an enterprise NetApp environment.

---

## 📑 Table of Contents
1. [🔍 Phase 1: Discovery & Identification](#phase-1)
2. [🏷️ Phase 2: Ownership Management](#phase-2)
3. [🚑 Phase 3: Proactive Replacement & Failing](#phase-3)
4. [🧼 Phase 4: Zeroing & Sanitization](#phase-4)
5. [🔐 Phase 5: Hardware Encryption (SED/NSE)](#phase-5)

---


---

<a id="phase-1"></a>
## 🔍 Phase 1: Discovery & Identification
*Before operating on a disk, you must locate it logically in the OS and physically in the datacenter.*

Show all disks in the cluster and their current status (Spare, Shared, Broken, or Data):
```bash
storage disk show
```

Filter specifically for disks that are available to be added to an aggregate:
```bash
storage disk show -container-type spare
```

Filter for disks that ONTAP has marked as failed:
```bash
storage disk show -container-type broken
```

**Blink the Physical LED:**
Turn on the fault LED on a specific disk for 15 minutes so the datacenter tech can safely pull it (replace `<disk_name>` with the actual ID, e.g., `1.0.15`):
```bash
storage disk set-led -disk <disk_name> -action on -duration 15
```

---

<a id="phase-2"></a>
## 🏷️ Phase 2: Ownership Management
*In ONTAP, a disk cannot be used until it is explicitly owned by a specific node (controller).*

Assign an unowned disk to a specific node:
```bash
storage disk assign -disk <disk_name> -owner <node_name>
```

Assign ALL currently unowned disks to a specific node at once:
```bash
storage disk assign -all true -node <node_name>
```

Check the cluster's automatic disk assignment policy:
```bash
storage disk option show -fields autoassign
```

**Remove Disk Ownership (Advanced):**
If you need to move a spare disk to the other controller, or if you are preparing to physically move shelves to a new cluster, you must strip the ownership. This requires advanced privilege:
```bash
set -privilege advanced
storage disk removeowner -disk <disk_name>
set -privilege admin
```

---

<a id="phase-3"></a>
## 🚑 Phase 3: Proactive Replacement & Failing
*Do not wait for a disk to hard-fail if it is throwing media errors. ONTAP allows you to seamlessly copy data off a dying disk to a spare before failing it.*

**Proactive Disk Replacement (Rapid RAID Recovery):**
Copy data from a suspect disk to a healthy spare, then automatically fail the suspect disk. This prevents the aggregate from entering a degraded state.
```bash
storage disk replace -disk <failing_disk_name> -replacement <healthy_spare_name>
```

Manually force a disk to fail immediately (puts the aggregate into degraded mode until parity rebuilds):
```bash
storage disk fail -disk <disk_name>
```

**Unfail a Disk (Advanced):**
If a disk was accidentally marked as broken (e.g., someone pulled the wrong drive temporarily), you can force ONTAP to accept it back into the spare pool:
```bash
set -privilege advanced
storage disk unfail -disk <disk_name>
set -privilege admin
```

---

<a id="phase-4"></a>
## 🧼 Phase 4: Zeroing & Sanitization
*Erasing data securely.*

When a disk is added to the spare pool, ONTAP must write zeros to every block before it can be added to an aggregate. Start a background zeroing job for all non-zeroed spares:
```bash
storage disk zerospares
```

Check the progress of active zeroing jobs:
```bash
storage disk show -state zeroing
```

**Disk Sanitization (Department of Defense Wipe):**
If you are retiring hardware and need to cryptographically shred or multi-pass wipe the physical media (Requires the `sanitization` license to be active):
```bash
storage disk sanitize start -disk <disk_name> -pattern cycle1_pattern
```

---

<a id="phase-5"></a>
## 🔐 Phase 5: Hardware Encryption (SED/NSE)
*Managing NetApp Storage Encryption (NSE) physical self-encrypting drives.*

Show the FIPS compliance and encryption status of all physical disks:
```bash
storage encryption disk show
```

Sanitize (cryptographically shred) a self-encrypting drive. This destroys the physical encryption key on the drive, instantly rendering all data unreadable, and returns the disk to an unassigned state:
```bash
storage encryption disk sanitize -disk <disk_name>
```

---
