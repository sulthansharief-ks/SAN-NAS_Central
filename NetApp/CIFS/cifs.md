# 📁 NetApp ONTAP: CIFS Share on a Qtree (with 100GB Quota) 🚀

This Standard Operating Procedure (SOP) covers creating a CIFS (SMB) share specifically hosted on a **Qtree** and enforcing a hard **100GB storage limit** using Tree Quotas. 

> ✅ **Rule:** This guide assumes your **SVM**, **CIFS Server**, and **Volume** are already created and online, and the volume is mounted to a junction path (e.g., `/<VOL>`).
> 🏷️ Replace placeholders like `<SVM>`, `<VOL>`, `<QTREE>`, and `<SHARE>` with your actual environment names.

## 📑 Table of Contents
1. [🧠 Prerequisites & Architecture](#prereq)
2. [🌳 Step 1: Create the Qtree](#create-qtree)
3. [🧾 Step 2: Apply the 100GB Quota](#apply-quota)
4. [🤝 Step 3: Create the CIFS Share](#create-share)
5. [🔐 Step 4: Configure Share Permissions (ACLs)](#config-acls)
6. [✅ Step 5: Verification](#verification)

---





---

<a id="prereq"></a>
## 🧠 Prerequisites & Architecture
* **Volume Junction Path:** You must know where your volume is mounted in the SVM namespace. If your volume is named `vol_data`, it is typically mounted at `/vol_data`.
* **Security Style:** Since this is for CIFS (Windows), the Qtree **must** be set to `ntfs` security style so Windows ACLs can be applied from the client side.

---

<a id="create-qtree"></a>
## 🌳 Step 1: Create the Qtree
*The Qtree acts as a sub-partition inside the volume where we can apply a hard quota.*

```bash
# Create the Qtree and force the security style to NTFS
volume qtree create -vserver <SVM> -volume <VOL> -qtree <QTREE> -security-style ntfs

# Verify the Qtree was created successfully
volume qtree show -vserver <SVM> -volume <VOL> -qtree <QTREE>
```

---

<a id="apply-quota"></a>
## 🧾 Step 2: Apply the 100GB Quota
*We need to assign a 100GB limit to the newly created Qtree.*

```bash
# 1. Check which Quota Policy is currently assigned to your volume
volume show -vserver <SVM> -volume <VOL> -fields quota-policy
# (Let's assume the policy is named 'default_policy')

# 2. Create the Tree Quota rule for 100GB
volume quota policy rule create -vserver <SVM> -policy-name <POLICY_NAME> -volume <VOL> -type tree -target <QTREE> -disk-limit 100GB

# 3. Apply the quota changes to the volume (Activate it)
# Note: If quotas are currently OFF for this volume, use 'volume quota on' instead.
volume quota resize -vserver <SVM> -volume <VOL> -foreground

# 4. Verify the quota is active and reporting the correct limit
volume quota report -vserver <SVM> -volume <VOL> -qtree <QTREE>
```

---

<a id="create-share"></a>
## 🤝 Step 3: Create the CIFS Share
*Expose the Qtree path to the network as an SMB share.*

```bash
# Create the CIFS share pointing directly to the Qtree path
# Note: The path is usually /<Volume_Junction_Path>/<Qtree_Name>
vserver cifs share create -vserver <SVM> -share-name <SHARE> -path /<VOL>/<QTREE> -comment "100GB Qtree Share"

# Verify the share properties
vserver cifs share show -vserver <SVM> -share-name <SHARE>
```

---

<a id="config-acls"></a>
## 🔐 Step 4: Configure Share Permissions (ACLs)
*By default, ONTAP shares are created with `Everyone / Full_Control`. Best practice dictates removing this and adding specific Active Directory groups.*

```bash
# 1. Grant Full Control (or Change/Read) to your specific AD Group
# Example: -user-or-group "DOMAIN\FileShareAdmins"
vserver cifs share access-control create -vserver <SVM> -share <SHARE> -user-or-group "<DOMAIN>\<AD_Group>" -permission Full_Control

# 2. Remove the default "Everyone" access
vserver cifs share access-control delete -vserver <SVM> -share <SHARE> -user-or-group Everyone

# 3. Verify the ACLs
vserver cifs share access-control show -vserver <SVM> -share <SHARE>
```

---

<a id="verification"></a>
## ✅ Step 5: Verification
*Everything is configured on the storage side. Now verify from the client side.*

1. Log in to a Windows machine connected to the domain.
2. Open File Explorer and navigate to `\\<SVM_Data_LIF_IP>\<SHARE>`.
3. Right-click in the folder, select **Properties**.
4. You should see the **Capacity** reported exactly as **100GB** (due to the Tree Quota intercepting the volume capacity report).
5. From Windows, you can now right-click the folder, go to the **Security** tab, and configure your granular NTFS file/folder permissions.
