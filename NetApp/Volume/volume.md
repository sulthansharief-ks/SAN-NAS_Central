# 📦 NetApp ONTAP (9.x) — Volume Operations (Scratch ➜ Advanced) 🚀
> ✅ **Rule:** This cheat-sheet sticks to **volume-scoped commands only** (i.e., commands that start with `volume ...`).
> 🏷️ Replace placeholders like `<SVM> <VOL> <AGGR> <SIZE>` etc. with your environment's details.

## 📑 Table of Contents
1. [🧰 0) Quick CLI Helpers](#quick-cli)
2. [🆕 1) Create a Volume (from scratch)](#create-volume)
3. [👀 2) Show / Inventory / Inspect](#show-inventory)
4. [🟢🔴 3) State Operations (online/offline/restrict)](#state-ops)
5. [🧷 4) Mount / Unmount (Junction Path for NAS volumes)](#mount-unmount)
6. [✍️ 5) Modify / Rename / Comment](#modify-rename)
7. [📏 6) Resize (Grow/Shrink) + Autosize](#resize)
8. [🗜️ 7) Efficiency (Dedup/Compression/Compaction)](#efficiency)
9. [📸 8) Snapshots (volume snapshot operations)](#snapshots)
10. [🧬 9) FlexClone (Volume Clones)](#flexclone)
11. [🚚 10) Volume Move (between aggregates)](#volume-move)
12. [🔐 11) Volume Encryption (if supported)](#encryption)
13. [☁️ 12) Volume Tiering (FabricPool volume settings)](#tiering)
14. [🌲 13) Qtrees (volume qtree)](#qtrees)
15. [🧾 14) Quotas (volume quota)](#quotas)
16. [🗑️ 15) Delete + Recovery Queue (safer deletes)](#delete-recovery)
17. [🧠 16) “Power” One-Liners (volume-only)](#power-oneliners)

---

<a id="quick-cli"></a>
## 🧰 0) Quick CLI Helpers (Exceptions to the rule)
> *Commands to help you navigate the ONTAP CLI faster and unlock advanced capabilities.*

- `man volume` (View the manual for volume commands)
- `volume ?` (List all volume subcommands)
- `volume create ?` (List all flags for volume creation)
- `set -privilege advanced`  ⚙️ (Unlock hidden/advanced commands)
- `set -privilege admin`     ✅ (Return to safe admin mode)

---

<a id="create-volume"></a>
## 🆕 1) Create a Volume (from scratch)
> *Allocates a logical container on a physical aggregate to store your data.*

### 1.1 Basic create (Unmounted / SAN typically)
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -state online`

### 1.2 NAS-style create (Mounted with junction path)
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -state online -junction-path /<VOL>`

### 1.3 Common create options (Thin provisioning, Security, Policies)
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -space-guarantee none` *(Thin Provisioned)*
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -snapshot-policy default`
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -security-style unix`
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -unix-permissions 0770`

---

<a id="show-inventory"></a>
## 👀 2) Show / Inventory / Inspect
> *Commands to view the health, capacity, and configuration of your volumes.*

### 2.1 List volumes
- `volume show`
- `volume show -vserver <SVM>`
- `volume show -vserver <SVM> -volume <VOL>`

### 2.2 Show useful fields (Custom views)
- `volume show -vserver <SVM> -volume <VOL> -fields state,size,aggregate,used,available,percent-used`
- `volume show -vserver <SVM> -fields vserver,volume,aggregate,state,size,used,available,percent-used,junction-path,security-style,space-guarantee`

### 2.3 Deep detail (everything)
- `volume show -vserver <SVM> -volume <VOL> -instance`

### 2.4 Space breakdown (What is eating the space?)
- `volume show-space -vserver <SVM> -volume <VOL>`
- `volume show-footprint -vserver <SVM> -volume <VOL>`

---

<a id="state-ops"></a>
## 🟢🔴 3) State Operations (online/offline/restrict)
> *Controls the operational availability of a volume to users and applications.*

- `volume online  -vserver <SVM> -volume <VOL>`
- `volume offline -vserver <SVM> -volume <VOL>` *(Required before deletion)*
- `volume restrict -vserver <SVM> -volume <VOL>` ⚠️ *(Used rarely, mostly for legacy SnapMirror init)*

---

<a id="mount-unmount"></a>
## 🧷 4) Mount / Unmount (Junction Path for NAS volumes)
> *Attaches a NAS volume to the SVM's namespace so clients can route to it and access files.*

### 4.1 Show junction path
- `volume show -vserver <SVM> -volume <VOL> -fields junction-path`

### 4.2 Mount / change junction
- `volume mount   -vserver <SVM> -volume <VOL> -junction-path /<VOL>`
- `volume mount   -vserver <SVM> -volume <VOL> -junction-path /data/<VOL>`

### 4.3 Unmount (Takes the share offline for users)
- `volume unmount -vserver <SVM> -volume <VOL>`

---

<a id="modify-rename"></a>
## ✍️ 5) Modify / Rename / Comment
> *Alters the properties, naming, or security style of an existing volume.*

### 5.1 Rename volume
- `volume rename -vserver <SVM> -volume <OLD_VOL> -newname <NEW_VOL>`

### 5.2 Common modifies
- `volume modify -vserver <SVM> -volume <VOL> -comment "Owned by AppTeam"`
- `volume modify -vserver <SVM> -volume <VOL> -unix-permissions 0770`
- `volume modify -vserver <SVM> -volume <VOL> -security-style unix`
- `volume modify -vserver <SVM> -volume <VOL> -snapshot-policy default`
- `volume modify -vserver <SVM> -volume <VOL> -percent-snapshot-space 5` *(Changes snapshot reserve %)*

### 5.3 Space guarantee (thin vs thick)
- `volume show   -vserver <SVM> -volume <VOL> -fields space-guarantee`
- `volume modify -vserver <SVM> -volume <VOL> -space-guarantee none`       🪶 *(Thin Provisioned)*
- `volume modify -vserver <SVM> -volume <VOL> -space-guarantee volume`     🧱 *(Thick Provisioned)*

---

<a id="resize"></a>
## 📏 6) Resize (Grow/Shrink) + Autosize
> *Expands or shrinks a volume's capacity, either manually or via automated thresholds.*

### 6.1 Manual Resize
- `volume show -vserver <SVM> -volume <VOL> -fields size,used,available,percent-used`
- `volume size -vserver <SVM> -volume <VOL> -new-size 1TB` *(Sets absolute size to 1TB)*
- `volume size -vserver <SVM> -volume <VOL> -new-size +100GB` *(Adds 100GB to current size)*
- `volume size -vserver <SVM> -volume <VOL> -new-size -50GB` *(Shrinks by 50GB)*

### 6.2 Autosize (autogrow / grow_shrink)
Show autosize status:
- `volume show -vserver <SVM> -volume <VOL> -fields autosize-mode,autosize-maximum-size,autosize-grow-threshold-percent`

Enable grow only:
- `volume autosize -vserver <SVM> -volume <VOL> -mode grow -maximum-size 2TB -grow-threshold-percent 85`

Enable grow + shrink:
- `volume autosize -vserver <SVM> -volume <VOL> -mode grow_shrink -maximum-size 2TB -grow-threshold-percent 85 -shrink-threshold-percent 60`

Disable autosize:
- `volume autosize -vserver <SVM> -volume <VOL> -mode off`

---

<a id="efficiency"></a>
## 🗜️ 7) Efficiency (Dedup/Compression/Compaction)
> *Saves physical disk space by removing duplicate data blocks and compressing files.*

Show efficiency status and savings:
- `volume efficiency show -vserver <SVM> -volume <VOL>`
- `volume efficiency show -vserver <SVM> -volume <VOL> -instance`

Enable/Disable on a volume:
- `volume efficiency on  -vserver <SVM> -volume <VOL>`
- `volume efficiency off -vserver <SVM> -volume <VOL>`

Run/Stop a manual scan:
- `volume efficiency start -vserver <SVM> -volume <VOL> -scan-old-data true`
- `volume efficiency stop  -vserver <SVM> -volume <VOL>`

Efficiency Policy Management:
- `volume efficiency policy show -vserver <SVM>`
- `volume efficiency policy create -vserver <SVM> -policy <POLICY_NAME> -schedule daily`
- `volume efficiency modify -vserver <SVM> -volume <VOL> -policy <POLICY_NAME>`

---

<a id="snapshots"></a>
## 📸 8) Snapshots (volume snapshot operations)
> *Creates instant, read-only, point-in-time copies of a volume for backup and rapid recovery.*

### 8.1 Show / Create / Rename / Delete
- `volume snapshot show   -vserver <SVM> -volume <VOL>`
- `volume snapshot create -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME>`
- `volume snapshot rename -vserver <SVM> -volume <VOL> -snapshot <OLD_SNAP> -new-name <NEW_SNAP>`
- `volume snapshot delete -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME>`

### 8.2 Restore Operations (Single File vs. Whole Volume)
**Scenario A: Restore a Single File (Safe & Recommended)**
Use this to grab a single corrupted/deleted file out of a snapshot without affecting the rest of the volume. 
*(Note: Path must be relative to the volume root)*
- `volume snapshot restore-file -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME> -path /<relative/path/to/file.txt>`

**Scenario B: Revert the Entire Volume (⚠️ HIGHLY DESTRUCTIVE)**
Reverts the *entire* volume back to the exact state it was in at the time of the snapshot. **All data written after the snapshot will be permanently lost**, and all newer snapshots will be deleted.
- `volume snapshot restore -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME>`

### 8.3 Snapshot Reserve (Hidden snapshot space)
- `volume show   -vserver <SVM> -volume <VOL> -fields percent-snapshot-space`
- `volume modify -vserver <SVM> -volume <VOL> -percent-snapshot-space 5` *(Sets 5% of vol for snapshots)*

### 8.4 Snapshot Policy (Volume-scoped attachment)
- `volume snapshot policy show -vserver <SVM>`
- `volume snapshot policy create -vserver <SVM> -policy <POLICY> -enabled true`
- `volume snapshot policy add-schedule     -vserver <SVM> -policy <POLICY> -schedule hourly -count 24`
- `volume snapshot policy remove-schedule -vserver <SVM> -policy <POLICY> -schedule hourly`
- `volume modify -vserver <SVM> -volume <VOL> -snapshot-policy <POLICY>`

---

<a id="flexclone"></a>
## 🧬 9) FlexClone (Volume Clones)
> *Creates instant, writable, zero-capacity copies of a volume for testing, development, or recovery.*

### 9.1 Create clone (Instant, zero-copy clone from a snapshot)
- `volume snapshot show -vserver <SVM> -volume <VOL>`
- `volume clone create -vserver <SVM> -flexclone <CLONE_VOL> -type RW -parent-volume <VOL> -parent-snapshot <SNAP_NAME>`

*(Optional: Mount the clone to access it)*:
- `volume mount -vserver <SVM> -volume <CLONE_VOL> -junction-path /<CLONE_VOL>`

Show clones:
- `volume clone show -vserver <SVM>`

### 9.2 Split clone (Make it independent from parent - consumes space)
- `volume clone split start -vserver <SVM> -flexclone <CLONE_VOL>`
- `volume clone split show  -vserver <SVM> -flexclone <CLONE_VOL>`
- `volume clone split stop  -vserver <SVM> -flexclone <CLONE_VOL>`

---

<a id="volume-move"></a>
## 🚚 10) Volume Move (between aggregates)
> *Non-disruptively migrates a live volume from one physical aggregate to another for load balancing or hardware upgrades.*

Show move status:
- `volume move show`
- `volume move show -vserver <SVM> -volume <VOL>`

Start move:
- `volume move start -vserver <SVM> -volume <VOL> -destination-aggregate <DEST_AGGR>`

Control an active move:
- `volume move pause  -vserver <SVM> -volume <VOL>`
- `volume move resume -vserver <SVM> -volume <VOL>`
- `volume move abort  -vserver <SVM> -volume <VOL>`

---

<a id="encryption"></a>
## 🔐 11) Volume Encryption (if supported)
> *Secures data at rest using software or hardware-based encryption keys (NVE/NAE) to prevent unauthorized drive access.*

Create a new encrypted volume:
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -encrypt true`

Show encryption flag:
- `volume show -vserver <SVM> -volume <VOL> -fields is-encrypted,encryption-state`

Convert existing unencrypted volume to encrypted (In-place):
- `volume encryption conversion start -vserver <SVM> -volume <VOL>`
- `volume encryption conversion show`

Rekey (Rotate encryption keys):
- `volume encryption rekey start -vserver <SVM> -volume <VOL>`
- `volume encryption rekey show`

---

<a id="tiering"></a>
## ☁️ 12) Volume Tiering (FabricPool volume settings)
> *Automatically moves cold (inactive) data to cheaper object storage (cloud) to free up high-performance SSD space.*

Show tiering fields:
- `volume show -vserver <SVM> -volume <VOL> -fields tiering-policy,tiering-minimum-cooling-days,cloud-retrieval-policy`

Set tiering policy:
- `volume modify -vserver <SVM> -volume <VOL> -tiering-policy none` *(Keep all data on hot tier)*
- `volume modify -vserver <SVM> -volume <VOL> -tiering-policy snapshot-only` *(Tier only snapshots)*
- `volume modify -vserver <SVM> -volume <VOL> -tiering-policy auto` *(Tier all cold data)*

Set cooling days (How long data must be cold before tiering):
- `volume modify -vserver <SVM> -volume <VOL> -tiering-minimum-cooling-days 31`

---

<a id="qtrees"></a>
## 🌲 13) Qtrees (volume qtree)
> *Creates logical sub-partitions within a volume, allowing you to apply discrete security styles or quota limits.*

Create qtree:
- `volume qtree create -vserver <SVM> -volume <VOL> -qtree <QTREE> -security-style unix`

Show:
- `volume qtree show -vserver <SVM> -volume <VOL>`

Delete:
- `volume qtree delete -vserver <SVM> -volume <VOL> -qtree <QTREE>`

Rename:
- `volume qtree rename -vserver <SVM> -volume <VOL> -qtree <QTREE> -newname <NEW_QTREE>`

---

<a id="quotas"></a>
## 🧾 14) Quotas (volume quota)
> *Restricts or tracks the amount of disk space and file counts that users, groups, or qtrees can consume.*
> ✅ This section **creates a new quota policy** (not using `default`), adds rules to it, assigns it to the volume, then compiles quotas.

### 14.1 Create a NEW quota policy
- `volume quota policy create -vserver <SVM> -policy-name <QP_NAME>`

(Optional) verify policies:
- `volume quota policy show -vserver <SVM>`

### 14.2 Add quota rules into the NEW policy
Show quota rules (all / for policy):
- `volume quota policy rule show -vserver <SVM>`
- `volume quota policy rule show -vserver <SVM> -policy-name <QP_NAME>`

**User quota (cap a user at 100GB):**
- `volume quota policy rule create -vserver <SVM> -policy-name <QP_NAME> -volume <VOL> -type user -target <USER_OR_ID> -qtree "" -disk-limit 100GB`

**Qtree (tree) quota (cap a qtree at 500GB):**
- `volume quota policy rule create -vserver <SVM> -policy-name <QP_NAME> -volume <VOL> -type tree -target <QTREE> -disk-limit 500GB`

### 14.3 Assign the NEW quota policy to the volume
Show current quota-policy assigned to the volume:
- `volume show -vserver <SVM> -volume <VOL> -fields quota-policy`

Assign your new policy:
- `volume modify -vserver <SVM> -volume <VOL> -quota-policy <QP_NAME>`

### 14.4 Compile quotas: enable / resize / report / disable
Show quota status:
- `volume quota show -vserver <SVM> -volume <VOL>`

Enable quotas (first time compilation) — *Use -foreground to wait for completion*:
- `volume quota on      -vserver <SVM> -volume <VOL> -foreground`

If quotas were already on and you changed rules/policy:
- `volume quota resize -vserver <SVM> -volume <VOL> -foreground`

Report effective quotas:
- `volume quota report -vserver <SVM> -volume <VOL>`

Disable quotas:
- `volume quota off    -vserver <SVM> -volume <VOL> -foreground`

### 14.5 Qtree-Specific Quota Workflow (The "Tree Quota")
> ⚠️ **Note:** You do not assign policies directly to Qtrees. You assign the policy to the **Volume**, then add a **Tree Rule** for the specific Qtree.

1. **Identify the Volume's active policy:**
   - `volume show -vserver <SVM> -volume <VOL> -fields quota-policy`
2. **Add a "Tree Rule" to that policy:**
   - `volume quota policy rule create -vserver <SVM> -policy-name <CURRENT_POLICY_NAME> -volume <VOL> -type tree -target <QTREE_NAME> -disk-limit 50GB`
3. **Activate the change (Resize):**
   - `volume quota resize -vserver <SVM> -volume <VOL> -foreground`
4. **Verify:**
   - `volume quota report -vserver <SVM> -volume <VOL> -qtree <QTREE_NAME>`

---

<a id="delete-recovery"></a>
## 🗑️ 15) Delete + Recovery Queue (safer deletes)
> *Safely destroys volumes and manages the temporary recycle bin to prevent accidental data loss.*

### 15.1 Standard delete flow
- `volume unmount -vserver <SVM> -volume <VOL>`  *(Mandatory if junction path exists)*
- `volume offline -vserver <SVM> -volume <VOL>`  *(Mandatory before delete)*
- `volume delete  -vserver <SVM> -volume <VOL>`

### 15.2 Recovery queue (ONTAP 9.4+ Volume Retention)
Show deleted volumes held in the recovery queue:
- `volume recovery-queue show`

Restore (Recover) a deleted volume before it expires:
- `volume recovery-queue recover -vserver <SVM> -volume <VOL>`

Purge permanently ☠️ (Skip retention period):
- `volume recovery-queue purge -vserver <SVM> -volume <VOL>`

---

<a id="power-oneliners"></a>
## 🧠 16) “Power” One-Liners (volume-only)
> *Advanced, chained commands to quickly pull comprehensive reports for daily administration.*

- `volume show -vserver <SVM> -fields volume,aggregate,state,size,used,available,percent-used,junction-path,space-guarantee,is-encrypted,quota-policy`
- `volume show-space -vserver <SVM> -volume <VOL>`
- `volume show-footprint -vserver <SVM> -volume <VOL>`
- `volume snapshot show -vserver <SVM> -volume <VOL>`
- `volume efficiency show -vserver <SVM> -volume <VOL> -instance`
- `volume move show -vserver <SVM> -volume <VOL>`
