# 🌍 NetApp ONTAP: Global CIFS/SMB Outage Resolution SOP 🚨

A **Global CIFS Outage** means *zero* users can access the shares on a specific Storage Virtual Machine (SVM). Users are experiencing "Network Path Not Found" (Error 0x80070035), continuous credential prompts, or immediate timeouts. 

Because this affects everyone, the issue is not at the file/folder permission level. The failure lies in the core infrastructure: Network, Active Directory Trust, or Logical ONTAP State.

> 🏷️ **Rule:** Replace placeholders like `<SVM>`, `<LIF_NAME>`, `<VOL_NAME>`, and `<POLICY_NAME>` with your environment values. All commands verified against ONTAP 9.10+ official documentation.

---

## 📑 Table of Contents
1. [🌐 Phase 1: Network & LIF Reachability (The Foundation)](#phase-1)
2. [⏱️ Phase 2: Active Directory, DNS & NTP (The Trust)](#phase-2)
3. [⚙️ Phase 3: SVM & CIFS Server Status (The Engine)](#phase-3)
4. [📂 Phase 4: Volume & Junction Path State (The Map)](#phase-4)
5. [🛡️ Phase 5: SVM/Volume Export Policies (The Bouncer)](#phase-5)
6. [📚 Phase 6: Official NetApp Documentation Reference](#phase-6)

---

```mermaid
graph TD
    classDef base fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    classDef network fill:#001e26,stroke:#00b8ff,stroke-width:3px,color:#fff
    classDef ad fill:#2d2a1b,stroke:#d29922,stroke-width:3px,color:#fff
    classDef ontap fill:#0f2d1a,stroke:#2ea043,stroke-width:3px,color:#fff
    classDef block fill:#2a1215,stroke:#f85149,stroke-width:3px,color:#fff

    Start((🚨 GLOBAL<br/>CIFS OUTAGE)):::base

    P1["1. Network & LIFs<br/>Are Data IPs Pingable?"]:::network
    P2["2. AD & Time Sync<br/>Is Kerberos Trust Intact?"]:::ad
    P3["3. CIFS Engine<br/>Is CIFS Admin Status UP?"]:::ontap
    P4["4. Volume State<br/>Are Junction Paths Active?"]:::ontap
    P5["5. Export Policies<br/>Is the Subnet Blocked?"]:::block

    Start --> P1
    P1 --> P2
    P2 --> P3
    P3 --> P4
    P4 --> P5

    linkStyle 0,1,2,3,4 stroke:#8b949e,stroke-width:2px
```

---

<a id="phase-1"></a>
## 🌐 Phase 1: Network & LIF Reachability (The Foundation)
*If the storage IP is offline, the share drops for the entire company.*

### 1.1 Verify Data LIF Status
Ensure the CIFS IP addresses are administratively and operationally UP, and residing on healthy physical ports.

```bash
network interface show -vserver <SVM> -data-protocol cifs -fields status-admin,status-oper,is-home,address,curr-port
```
* **Target:** `status-admin` = `up`, `status-oper` = `up`, `is-home` = `true`.

**🛠️ Action to Take (If LIF is down or displaced):**

If Admin Status is down:
```bash
network interface modify -vserver <SVM> -lif <LIF_NAME> -status-admin up
```

If LIF is not on its home port (`is-home` = `false`):
```bash
network interface revert -vserver <SVM> -lif <LIF_NAME>
```

### 1.2 Verify Default Routing
If users are on a different subnet/VLAN than the NetApp, the SVM MUST have a default route back to the core switch.

```bash
network route show -vserver <SVM>
```
* **Target:** A route with destination `0.0.0.0/0` pointing to your core gateway IP must exist.

**🛠️ Action to Take (If Route is missing):**

Create the default route:
```bash
network route create -vserver <SVM> -destination 0.0.0.0/0 -gateway <GATEWAY_IP>
```

---

<a id="phase-2"></a>
## ⏱️ Phase 2: Active Directory, DNS & NTP (The Trust)
*CIFS relies exclusively on Kerberos authentication. If the NetApp's clock drifts by more than 5 minutes from the Domain Controller, or if the AD computer account password expires, the trust is broken, and ALL authentication requests are dropped.*

### 2.1 Verify NTP Time Synchronization
Check the cluster time against your AD Domain Controllers.

```bash
cluster date show
cluster time-service ntp server show
```
* **Target:** Cluster time must be within 5 minutes of the Windows Domain Controllers.

**🛠️ Action to Take (If Time is skewed):**

Add the Domain Controller as an NTP server and force sync:
```bash
cluster time-service ntp server create -server <DC_IP>
cluster time-service ntp resync
```

### 2.2 Verify Domain Controller Reachability & DNS
Ensure ONTAP can actively query your DNS servers and locate the Domain Controllers.

```bash
# 1. View configured DNS servers
vserver services name-service dns show -vserver <SVM>

# 2. Test DNS resolution
vserver services name-service dns check -vserver <SVM>

# 3. Verify discovered DCs
vserver cifs domain discovered-servers show -vserver <SVM>
```
* **Target:** DNS check should return `OK`. DC status should be `OK`. 

**🛠️ Action to Take (If Status is 'down' or 'undiscovered'):**

*(Network Team Escalation: If DNS is fine but DCs are unreachable, request the network team to check firewalls for dropped traffic on ports 53, 88, 389, and 445).*

### 2.3 Verify the Kerberos Secure Channel (Trust)
This proves if the NetApp is still securely authenticated to the Active Directory domain.

```bash
vserver cifs domain trusts show -vserver <SVM>
```
* **Target:** Trust status should be healthy and established.

**🛠️ Action to Take (If Trust is Broken):**

Reset the CIFS machine account password. *(Note: ONTAP requires the `-user` parameter. It will prompt you securely for the password. Do not pass it in plain text).*
```bash
vserver cifs domain password-reset -vserver <SVM> -user <AD_Admin_Username>
```

---

<a id="phase-3"></a>
## ⚙️ Phase 3: SVM & CIFS Server Status (The Engine)
*Is the actual CIFS protocol engine running on the Storage Virtual Machine?*

### 3.1 Check CIFS Administrative State

```bash
vserver cifs show -vserver <SVM>
```
* **Target:** `Administrative Status` must be `up`.

**🛠️ Action to Take (If CIFS Admin Status is Down):**

Start the CIFS Server Service:
```bash
vserver cifs start -vserver <SVM>
```

---

<a id="phase-4"></a>
## 📂 Phase 4: Volume & Junction Path State (The Map)
*NetApp uses a unified namespace. If a volume drops offline, or its junction path is removed from the namespace, the Windows share path instantly becomes invalid for everyone.*

### 4.1 Verify Volume Online State & Junction Path

```bash
volume show -vserver <SVM> -fields state,junction-path,junction-active
```
* **Target:** `state` = `online`, `junction-path` = `/<vol_name>` (cannot be `-`), and `junction-active` = `true`.

**🛠️ Action to Take (If Volume is offline or unmounted):**

If Volume is Offline:
```bash
volume online -vserver <SVM> -volume <VOL_NAME>
```

If Junction Path is Missing/Inactive (`-`):
```bash
volume mount -vserver <SVM> -volume <VOL_NAME> -junction-path /<VOL_NAME>
```

---

<a id="phase-5"></a>
## 🛡️ Phase 5: SVM/Volume Export Policies (The Bouncer)
*Many admins think Export Policies are only for NFS. **This is false.** ONTAP evaluates Export Policies at the IP layer before it even checks CIFS Share ACLs. A misconfigured export policy will block the entire company.*

### 5.1 Check the Assigned Policies
Identify which export policies are attached to both the **Root Volume** and the **Data Volume**.

```bash
# 1. Identify the root volume
vserver show -vserver <SVM> -fields rootvolume

# 2. Check the policy assigned to the root volume
volume show -vserver <SVM> -volume <ROOT_VOL_NAME> -fields policy

# 3. Check the policy assigned to the data volume
volume show -vserver <SVM> -volume <DATA_VOL_NAME> -fields policy
```
* **Target:** Note the names of the policies (e.g., `default` or `cifs_policy`).

**🛠️ Action to Take (If policies are incorrectly assigned):**

Assign the correct policy to the volume:
```bash
volume modify -vserver <SVM> -volume <VOL_NAME> -policy <CORRECT_POLICY_NAME>
```

### 5.2 Inspect and Fix the Export Rules
Look inside the policies identified in step 5.1 to ensure CIFS is allowed.

```bash
vserver export-policy rule show -vserver <SVM> -policyname <POLICY_NAME>
```
* **Target:** There MUST be a rule that matches your client subnets (e.g., `10.0.0.0/8` or `0.0.0.0/0`) that allows the `cifs` (or `any`) protocol for read/write access.

**🛠️ Action to Take (If rules are missing or too restrictive):**

Create a rule allowing all internal IPs for CIFS. *(Note: ONTAP requires a `-ruleindex` parameter, and the protocol flag is singular `-protocol`).*
```bash
vserver export-policy rule create -vserver <SVM> -policyname <POLICY_NAME> -ruleindex 1 -clientmatch 0.0.0.0/0 -rorule any -rwrule any -protocol cifs
```
*(Note: Apply this to both the data volume policy and the root volume policy to ensure namespace traversal is permitted).*

---

<a id="phase-6"></a>
## 📚 Phase 6: Official NetApp Documentation Reference
*Below are the verified ONTAP 9 official documentation links for the critical diagnostic commands utilized in this SOP.*

| Command / Protocol | Official NetApp Documentation Reference |
| :--- | :--- |
| `network interface` | [Docs: network interface commands](https://docs.netapp.com/us-en/ontap-cli/network-interface-show.html) |
| `network route` | [Docs: network route commands](https://docs.netapp.com/us-en/ontap-cli/network-route-show.html) |
| `cluster time-service ntp` | [Docs: cluster time-service ntp](https://docs.netapp.com/us-en/ontap-cli/cluster-time-service-ntp-server-show.html) |
| `vserver cifs domain` (Trusts/Reset) | [Docs: vserver cifs domain commands](https://docs.netapp.com/us-en/ontap-cli/vserver-cifs-domain-trusts-show.html) |
| `vserver services dns check` | [Docs: vserver services name-service dns check](https://docs.netapp.com/us-en/ontap-cli/vserver-services-name-service-dns-check.html) |
| `vserver cifs show/start` | [Docs: vserver cifs commands](https://docs.netapp.com/us-en/ontap-cli/vserver-cifs-show.html) |
| `volume show/online/mount` | [Docs: volume commands](https://docs.netapp.com/us-en/ontap-cli/volume-show.html) |
| `vserver export-policy rule` | [Docs: vserver export-policy rule commands](https://docs.netapp.com/us-en/ontap-cli/vserver-export-policy-rule-create.html) |
