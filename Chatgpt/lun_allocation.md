# Fibre Channel (FC): Create a LUN (LDEV) and allocate/map it to a host ✅⚙️
Applies to: Hitachi Vantara VSP G-series (e.g., G130, G/F350, G/F370, G/F700, G/F900) using **HDvM - Storage Navigator** (SVOS 9.6.x).

---

## Before you start 🧩
- ✅ You have the **host HBA WWNs**.
- ✅ FC **zoning** is planned/complete (single-initiator zoning is common practice).
- ✅ The storage **FC ports are configured** (topology/switch settings as applicable).
  - (If you need it) FC topology procedure:  
    https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/managing-logical-volumes/configuring-fibre-channel-ports/setting-the-fibre-channel-topology

FC end-to-end workflow reference (ports → hosts → LU paths → security/auth):  
https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-97hm85026/managing-logical-volumes/lun-manager-overview/workflow-for-configuring-logical-units-fibre-channel

---

## 1) Create the volume (LDEV/LUN) 🆕📀
Official procedure:  
https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/configuring-custom-sized-provisioning/creating-ldevs/creating-one-or-more-ldevs

1. Open **Storage Navigator** → click **Storage Systems** → **expand** the storage system in the tree.
2. Go to either:
   - **Parity Groups** (if you want to choose a specific parity group), OR
   - **Logical Devices** (if you’re okay with any parity group).
3. Click **Create LDEVs**.
4. Select **Provisioning Type**.
5. (If internal) In **Parity Group Selection**, select drive type/RAID level → **Select Free Spaces** → choose free space → **OK**.
6. Set:
   - **LDEV Capacity** (and unit)
   - **Number of LDEVs** (if creating more than one)
7. Set naming/formatting as required:
   - **LDEV Name** (prefix/initial number)
   - **Format Type** (Normal/Quick/etc. depending on your policy)
8. (Optional) Click **Options** to confirm **Initial LDEV ID** and related settings (e.g., MP unit / T10 PI if applicable).
9. Click **Add** (moves to Selected LDEVs) → **Finish**.
10. In the confirmation window, click **Apply** ✅

➡️ Result: your LDEV(s) now exist.

---

## 2) Create a Host Group on the FC port and register the host WWNs 🧷🖥️
Official procedure:  
https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/managing-logical-volumes/configuring-hosts/creating-a-host-group-and-registering-hosts-in-the-host-group

> ⚠️ In FC, **Host Groups are per storage port**.  
> If you are presenting via **two ports for multipathing**, repeat this step for **each port** (e.g., CL1-A and CL1-B), typically with the **same WWNs** and same Host Mode/Options.

1. **Storage Systems** → expand the storage system.
2. Click **Ports/Host Groups/iSCSI Targets**.
3. Click **Create Host Groups**.
4. Enter/select:
   - **Host Group Name**
   - **Resource Group** (if used in your environment)
   - **Host Mode** (match the host OS/platform)
5. Register the host WWNs:
   - Select from **Available Hosts**, OR
   - Click **Add New Host** → enter **HBA WWN** (optional nickname) → select it
6. Select the **FC port(s)** to add this host group to (create on the exact port you will present from).
7. Click **Host Mode Options** and select the required options → click **Add**.
8. Click **Finish** → then **Apply** ✅

➡️ Result: a Host Group exists on the selected FC port with your host WWN(s) registered.

---

## 3) Map (allocate) the LDEV to the Host Group = Define LU Paths 🗺️🔗
Official procedure:  
https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/managing-logical-volumes/configuring-lu-paths/defining-lu-paths

1. **Storage Systems** → expand the storage system.
2. Go to **Ports/Host Groups/iSCSI Targets**.
3. Select the **Host Group** you created on the FC port.
4. Click **Add LUN Paths**.
5. In **Add LUN Paths**:
   - Select your **LDEV ID(s)** in **Available LDEVs**
   - Click **Add** (moves them to Selected LDEVs)
   - Click **Next**
6. In **Selection Object**:
   - Choose **Fibre** (FC)
   - Select the target **Host Group**
   - Click **Add** → **Next**
7. Confirm the LU paths:
   - (Optional) **Change LUN IDs** to set/adjust **LUN numbering** (Initial LUN ID)
   - (Optional) Change LDEV name/settings if needed
8. Click **Finish** → then **Apply** ✅

➡️ Result: the volume is now **presented/mapped** to the host via that FC port’s Host Group.

🔁 If you’re doing multipathing across multiple FC ports:
- Repeat **Step 2** (host group on the other port) and **Step 3** (map the same LDEV) for the second port.

---

## (Optional but recommended) LUN Security 🔒
Hitachi recommends controlling access using LUN security on ports where appropriate.  
(See “Applying LUN security on ports” in the same SVOS 9.6 guide index from the host/LU path section.)

---

## References 📚
- Create LDEVs: https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/configuring-custom-sized-provisioning/creating-ldevs/creating-one-or-more-ldevs
- Create host group + register WWNs: https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/managing-logical-volumes/configuring-hosts/creating-a-host-group-and-registering-hosts-in-the-host-group
- Define LU paths (map LUNs): https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-98rd9015/managing-logical-volumes/configuring-lu-paths/defining-lu-paths
- FC LU workflow overview: https://docs.hitachivantara.com/r/en-us/svos/9.6.0/mk-97hm85026/managing-logical-volumes/lun-manager-overview/workflow-for-configuring-logical-units-fibre-channel
