# 📦 NetApp ONTAP (9.x) — Volume Operations (Scratch ➜ Advanced) 🚀
> ✅ **Rule:** This cheat-sheet sticks to **volume-scoped commands only** (i.e., commands that start with `volume ...`).
> 🏷️ Replace placeholders like `<SVM> <VOL> <AGGR> <SIZE>` etc.

---

## 🧰 0) Quick CLI Helpers (Exceptions to the rule)
- `man volume`
- `volume ?`
- `volume create ?`
- `set -privilege advanced`   ⚙️
- `set -privilege admin`      ✅

---

## 🆕 1) Create a Volume (from scratch)
### 1.1 Basic create
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -state online`

### 1.2 NAS-style create (with junction path)
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -state online -junction-path /<VOL>`

### 1.3 Common create options
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -state online -comment "App volume"`
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -space-guarantee none`
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -snapshot-policy default`
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -security-style unix`
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -unix-permissions 0770`

---

## 👀 2) Show / Inventory / Inspect
### 2.1 List volumes
- `volume show`
- `volume show -vserver <SVM>`
- `volume show -vserver <SVM> -volume <VOL>`

### 2.2 Show useful fields
- `volume show -vserver <SVM> -volume <VOL> -fields state,size,aggregate,used,available,percent-used`
- `volume show -vserver <SVM> -fields vserver,volume,aggregate,state,size,used,available,percent-used,junction-path,security-style,space-guarantee`

### 2.3 Deep detail (everything)
- `volume show -vserver <SVM> -volume <VOL> -instance`

### 2.4 Space breakdown
- `volume show-space -vserver <SVM> -volume <VOL>`
- `volume show-footprint -vserver <SVM> -volume <VOL>`

---

## 🟢🔴 3) State Operations (online/offline/restrict)
- `volume online   -vserver <SVM> -volume <VOL>`
- `volume offline  -vserver <SVM> -volume <VOL>`
- `volume restrict -vserver <SVM> -volume <VOL>`  ⚠️ (restricted state for some ops)

---

## 🧷 4) Mount / Unmount (Junction Path for NAS volumes)
### 4.1 Show junction
- `volume show -vserver <SVM> -volume <VOL> -fields junction-path`

### 4.2 Mount / change junction
- `volume mount   -vserver <SVM> -volume <VOL> -junction-path /<VOL>`
- `volume mount   -vserver <SVM> -volume <VOL> -junction-path /data/<VOL>`

### 4.3 Unmount
- `volume unmount -vserver <SVM> -volume <VOL>`

---

## ✍️ 5) Modify / Rename / Comment
### 5.1 Rename volume
- `volume rename -vserver <SVM> -volume <OLD_VOL> -newname <NEW_VOL>`

### 5.2 Common modifies
- `volume modify -vserver <SVM> -volume <VOL> -comment "Owned by AppTeam"`
- `volume modify -vserver <SVM> -volume <VOL> -unix-permissions 0770`
- `volume modify -vserver <SVM> -volume <VOL> -security-style unix`
- `volume modify -vserver <SVM> -volume <VOL> -snapshot-policy default`
- `volume modify -vserver <SVM> -volume <VOL> -snapshot-reserve 5`

### 5.3 Space guarantee (thin vs thick)
- `volume show   -vserver <SVM> -volume <VOL> -fields space-guarantee`
- `volume modify -vserver <SVM> -volume <VOL> -space-guarantee none`     🪶 thin
- `volume modify -vserver <SVM> -volume <VOL> -space-guarantee volume`   🧱 thick

---

## 📏 6) Resize (Grow/Shrink) + Autosize
### 6.1 Resize
- `volume show -vserver <SVM> -volume <VOL> -fields size,used,available,percent-used`
- `volume size -vserver <SVM> -volume <VOL> -new-size 1TB`

### 6.2 Autosize (autogrow / grow_shrink)
Show autosize:
- `volume show -vserver <SVM> -volume <VOL> -fields autosize-mode,autosize-maximum-size,autosize-grow-threshold-percent`

Enable grow:
- `volume autosize -vserver <SVM> -volume <VOL> -mode grow -maximum-size 2TB -grow-threshold-percent 85`

Enable grow + shrink:
- `volume autosize -vserver <SVM> -volume <VOL> -mode grow_shrink -maximum-size 2TB -grow-threshold-percent 85 -shrink-threshold-percent 60`

Disable:
- `volume autosize -vserver <SVM> -volume <VOL> -mode off`

---

## 🗜️ 7) Efficiency (Dedup/Compression/Compaction)
Show:
- `volume efficiency show -vserver <SVM> -volume <VOL>`
- `volume efficiency show -vserver <SVM> -volume <VOL> -instance`

Enable/Disable:
- `volume efficiency on  -vserver <SVM> -volume <VOL>`
- `volume efficiency off -vserver <SVM> -volume <VOL>`

Run/Stop:
- `volume efficiency start -vserver <SVM> -volume <VOL>`
- `volume efficiency stop  -vserver <SVM> -volume <VOL>`

Policy (if you use them):
- `volume efficiency policy show -vserver <SVM>`
- `volume efficiency policy create -vserver <SVM> -policy <POLICY_NAME> -schedule daily`
- `volume efficiency modify -vserver <SVM> -volume <VOL> -policy <POLICY_NAME>`

---

## 📸 8) Snapshots (volume snapshot operations)
### 8.1 Show / create / delete
- `volume snapshot show   -vserver <SVM> -volume <VOL>`
- `volume snapshot create -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME>`
- `volume snapshot delete -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME>`

### 8.2 Restore (revert) ⚠️ destructive
- `volume snapshot restore -vserver <SVM> -volume <VOL> -snapshot <SNAP_NAME>`

### 8.3 Snapshot reserve
- `volume show   -vserver <SVM> -volume <VOL> -fields snapshot-reserve`
- `volume modify -vserver <SVM> -volume <VOL> -snapshot-reserve 5`

### 8.4 Snapshot policy (still volume-scoped)
- `volume snapshot policy show -vserver <SVM>`
- `volume snapshot policy create -vserver <SVM> -policy <POLICY> -enabled true`
- `volume snapshot policy add-schedule    -vserver <SVM> -policy <POLICY> -schedule hourly -count 24`
- `volume snapshot policy remove-schedule -vserver <SVM> -policy <POLICY> -schedule hourly`
- `volume modify -vserver <SVM> -volume <VOL> -snapshot-policy <POLICY>`

---

## 🧬 9) FlexClone (Volume Clones)
### 9.1 Create clone (from snapshot)
- `volume snapshot show -vserver <SVM> -volume <VOL>`
- `volume clone create -vserver <SVM> -flexclone <CLONE_VOL> -type RW -parent-volume <VOL> -parent-snapshot <SNAP_NAME>`

(Optional junction on clone):
- `volume mount -vserver <SVM> -volume <CLONE_VOL> -junction-path /<CLONE_VOL>`

Show clones:
- `volume clone show -vserver <SVM>`

### 9.2 Split clone (make independent)
- `volume clone split start -vserver <SVM> -flexclone <CLONE_VOL>`
- `volume clone split show  -vserver <SVM> -flexclone <CLONE_VOL>`
- `volume clone split stop  -vserver <SVM> -flexclone <CLONE_VOL>`

---

## 🚚 10) Volume Move (between aggregates)
Show status:
- `volume move show`
- `volume move show -vserver <SVM> -volume <VOL>`

Start move:
- `volume move start -vserver <SVM> -volume <VOL> -destination-aggregate <DEST_AGGR>`

Control:
- `volume move pause  -vserver <SVM> -volume <VOL>`
- `volume move resume -vserver <SVM> -volume <VOL>`
- `volume move abort  -vserver <SVM> -volume <VOL>`

---

## 🔐 11) Volume Encryption (if supported)
Create encrypted volume:
- `volume create -vserver <SVM> -volume <VOL> -aggregate <AGGR> -size <SIZE> -encrypt true`

Show encryption flag:
- `volume show -vserver <SVM> -volume <VOL> -fields is-encrypted`

Convert existing volume to encrypted:
- `volume encryption conversion start -vserver <SVM> -volume <VOL>`
- `volume encryption conversion show`

Rekey:
- `volume encryption rekey start -vserver <SVM> -volume <VOL>`
- `volume encryption rekey show`

---

## ☁️ 12) Volume Tiering (FabricPool volume settings)
Show tiering fields:
- `volume show -vserver <SVM> -volume <VOL> -fields tiering-policy,tiering-minimum-cooling-days,cloud-retrieval-policy`

Set tiering policy:
- `volume modify -vserver <SVM> -volume <VOL> -tiering-policy none`
- `volume modify -vserver <SVM> -volume <VOL> -tiering-policy snapshot-only`
- `volume modify -vserver <SVM> -volume <VOL> -tiering-policy auto`

Set cooling days:
- `volume modify -vserver <SVM> -volume <VOL> -tiering-minimum-cooling-days 31`

---

## 🌲 13) Qtrees (volume qtree)
Create qtree:
- `volume qtree create -vserver <SVM> -volume <VOL> -qtree <QTREE> -security-style unix`

Show:
- `volume qtree show -vserver <SVM> -volume <VOL>`

Delete:
- `volume qtree delete -vserver <SVM> -volume <VOL> -qtree <QTREE>`

Rename:
- `volume qtree rename -vserver <SVM> -volume <VOL> -qtree <QTREE> -newname <NEW_QTREE>`

---

## 🧾 14) Quotas (volume quota)
> ✅ This section **creates a new quota policy** (not using `default`), adds rules to it, assigns it to the volume, then compiles quotas.

### 14.1 Create a NEW quota policy
- `volume quota policy create -vserver <SVM> -policy-name <QP_NAME>`

(Optional) verify policies:
- `volume quota policy show -vserver <SVM>`

### 14.2 Add quota rules into the NEW policy
Show quota rules (all / for policy):
- `volume quota policy rule show -vserver <SVM>`
- `volume quota policy rule show -vserver <SVM> -policy-name <QP_NAME>`

Create quota rules (examples)

**User quota (cap a user at 100GB):**
- `volume quota policy rule create -vserver <SVM> -policy-name <QP_NAME> -volume <VOL> -type user -target <USER_OR_ID> -qtree "" -disk-limit 100GB`

**Qtree (tree) quota (cap a qtree at 50GB/500GB etc.):**
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
- `volume quota on     -vserver <SVM> -volume <VOL> -foreground`

If quotas were already on and you changed rules/policy:
- `volume quota resize -vserver <SVM> -volume <VOL> -foreground`

Report effective quotas:
- `volume quota report -vserver <SVM> -volume <VOL>`

Disable quotas:
- `volume quota off    -vserver <SVM> -volume <VOL> -foreground`

---

## 🗑️ 15) Delete + Recovery Queue (safer deletes)
### 15.1 Standard delete flow
- `volume unmount -vserver <SVM> -volume <VOL>`   (if NAS mounted)
- `volume offline  -vserver <SVM> -volume <VOL>`
- `volume delete   -vserver <SVM> -volume <VOL>`

### 15.2 Recovery queue (if available)
Show deleted volumes:
- `volume recovery-queue show`

Restore (Recover):
- `volume recovery-queue recover -vserver <SVM> -volume <VOL>`

Purge permanently ☠️:
- `volume recovery-queue purge -vserver <SVM> -volume <VOL>`

---

## 🧠 16) “Power” One-Liners (volume-only)
- `volume show -vserver <SVM> -fields volume,aggregate,state,size,used,available,percent-used,junction-path,space-guarantee,is-encrypted,quota-policy`
- `volume show-space -vserver <SVM> -volume <VOL>`
- `volume show-footprint -vserver <SVM> -volume <VOL>`
- `volume snapshot show -vserver <SVM> -volume <VOL>`
- `volume efficiency show -vserver <SVM> -volume <VOL> -instance`
- `volume move show -vserver <SVM> -volume <VOL>`
