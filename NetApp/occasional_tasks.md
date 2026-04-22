
# 🗓️ NetApp ONTAP: Advanced Scheduled Maintenance & SSL Lifecycle SOP 🚀

This Master Standard Operating Procedure (SOP) combines advanced scheduled maintenance activities (data mobility, hardware lifecycle management, and disaster recovery) with the complete SSL Certificate Lifecycle Management process for both self-signed and Trusted CA deployments.

> 🏷️ **Rule:** Replace placeholders like `<SVM>`, `<NODE>`, `<AGGR>`, `<VOL>`, and `<FQDN>` with your environment's specific details.

## 📑 Table of Contents
1. [📦 Change 1: Non-Disruptive Volume Migration (Data Mobility)](#change-1)
2. [🔀 Change 2: Aggregate Relocation (Load Balancing / HW Refresh)](#change-2)
3. [🚨 Change 3: Scheduled Disaster Recovery (DR) Drill (SVM-DR)](#change-3)
4. [💿 Change 4: Disk & Shelf Firmware Deployment](#change-4)
5. [🔐 Change 5: SSL Certificate Lifecycle Management (Self-Signed & CA)](#change-5)
6. [📚 Official NetApp Documentation Reference](#references)

---

```mermaid
graph TD
    %% --- Dark Mode Theme Definitions ---
    classDef prep fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    classDef mobility fill:#001e26,stroke:#00b8ff,stroke-width:3px,color:#fff
    classDef dr fill:#2a1215,stroke:#f85149,stroke-width:3px,color:#fff
    classDef hw fill:#2d2a1b,stroke:#d29922,stroke-width:3px,color:#fff
    classDef sec fill:#0f2d1a,stroke:#2ea043,stroke-width:3px,color:#fff

    Start((Scheduled<br/>Advanced Ops)):::prep

    VolMove["1. Volume Move<br/>(SAS to NVMe/Flash)"]:::mobility
    ARL["2. Aggregate Relocation<br/>(Shift Load to Partner)"]:::mobility
    DRTest["3. SVM-DR Drill<br/>(Failover Testing)"]:::dr
    FW["4. Disk/Shelf Firmware<br/>(Background Update)"]:::hw
    SSL["5. SSL Cert Renewal<br/>(Self-Signed & CA)"]:::sec

    Start --> VolMove
    Start --> ARL
    Start --> DRTest
    Start --> FW
    Start --> SSL

    linkStyle 0,1,2,3,4 stroke:#8b949e,stroke-width:2px
````

-----

\<a id="change-1"\>\</a\>

## 📦 Change 1: Non-Disruptive Volume Migration (Data Mobility)

*Used when retiring old aggregates, rebalancing capacity across the cluster, or moving a high-I/O volume from spinning disks (HDD) to All-Flash (SSD/NVMe). The move is completely transparent to clients.*

  * **Impact:** 🟢 Zero downtime. I/O is momentarily paused (usually \< 3 seconds) during the final cutover phase.

### 1.1 Initiate the Volume Move

```bash
# Start moving the volume to the new destination aggregate
volume move start -vserver <SVM_Name> -volume <VOL_Name> -destination-aggregate <New_AGGR_Name>
```

### 1.2 Monitor the Migration Progress

```bash
# Watch the background replication phase
volume move show -vserver <SVM_Name> -volume <VOL_Name>

# For detailed percent-complete statistics
volume move show -vserver <SVM_Name> -volume <VOL_Name> -fields state,phase,percent-complete
```

> **🛠️ Action to Take (If the move gets stuck in 'cutover' phase):**
>
>   * 👥 **Responsible Team:** **Storage Team**
>   * **Fix:** If the client is sending I/O faster than the NetApp can sync the final delta, the cutover will defer. You can force the cutover:
>     ```bash
>     volume move trigger-cutover -vserver <SVM_Name> -volume <VOL_Name>
>     ```

-----

\<a id="change-2"\>\</a\>

## 🔀 Change 2: Aggregate Relocation (Load Balancing / HW Refresh)

*Aggregate Relocation (ARL) physically transfers ownership of a group of disks (an aggregate) from one node to its HA partner. This is heavily used to balance CPU load or as part of a controller hardware refresh.*

  * **Impact:** 🟡 Low. SAN paths will shift to the HA partner. NAS traffic will experience a brief pause as ownership changes.

### 2.1 Pre-Relocation Health Check

```bash
# Ensure the HA pair is healthy and connected
storage failover show

# Verify the destination node has enough spare CPU/RAM to handle the new aggregate
node run -node <Destination_Node> -command sysstat -c 10
```

### 2.2 Execute the Relocation

```bash
# Move the aggregate to the HA partner
storage aggregate relocation start -node <Source_Node> -destination <Destination_Node> -aggregate-list <AGGR_Name>
```

### 2.3 Monitor and Verify

```bash
# Check the status of the relocation
storage aggregate relocation show

# Verify the new home owner of the aggregate
storage aggregate show -aggregate <AGGR_Name> -fields home-name,owner-name
```

-----

\<a id="change-3"\>\</a\>

## 🚨 Change 3: Scheduled Disaster Recovery (DR) Drill (SVM-DR)

*Enterprise compliance requires bi-annual DR testing. SVM-DR (Storage Virtual Machine Disaster Recovery) replicates the entire SVM (volumes, LIFs, CIFS shares, exports) to a remote site. This scheduled change breaks the mirror to test the DR site.*

  * **Impact:** 🟠 Moderate. Application teams must repoint DNS or remount storage to the DR IP addresses.

### 3.1 Quiesce and Break the Mirror (On DR Cluster)

*Stop replication and make the destination read/write.*

```bash
# 1. Stop active replication transfers
snapmirror quiesce -destination-path <DR_SVM_Name>:

# 2. Break the relationship (makes DR volumes writable)
snapmirror break -destination-path <DR_SVM_Name>:
```

### 3.2 Start the DR SVM and Network (On DR Cluster)

```bash
# 1. Start the SVM protocols
vserver start -vserver <DR_SVM_Name>

# 2. Bring up the DR Data LIFs
network interface modify -vserver <DR_SVM_Name> -lif * -status-admin up
```

*(At this point, Network/App teams test their applications against the DR site).*

### 3.3 Post-Drill Cleanup (Resync)

*Once the drill is over, discard the DR changes and resume replication from the Production site.*

```bash
# Resync the mirror (overwriting any changes made during the drill at the DR site)
snapmirror resync -destination-path <DR_SVM_Name>: -source-path <PROD_SVM_Name>:
```

-----

\<a id="change-4"\>\</a\>

## 💿 Change 4: Disk & Shelf Firmware Deployment

*NetApp frequently releases updated firmware for physical hard drives and disk shelves to fix bugs and improve performance. While ONTAP usually handles this in the background, enterprise environments often schedule manual pushes to monitor for I/O latency.*

  * **Impact:** 🟢 None. Firmware is applied to disks in the background, one at a time per RAID group.

### 4.1 Download the Firmware Package

*Download the `.zip` from the NetApp Support Site and host it on an internal web server.*

```bash
# Download the package into the cluster's memory
storage firmware download -node * -package-url http://<Web_Server_IP>/<all.zip>
```

### 4.2 Verify Current Firmware Levels

```bash
# Check current firmware versions of all disks
storage disk show -fields firmware-revision,model
```

### 4.3 Monitor Background Updates

*ONTAP automatically detects the new firmware in the system directory and begins updating eligible disks sequentially.*

```bash
# Monitor the background update process (will show 'updating' if active)
storage disk show -fields firmware-update-status

# Check for shelf firmware updates
system node run -node * -command sysconfig -v
```

-----

\<a id="change-5"\>\</a\>

## 🔐 Change 5: SSL Certificate Lifecycle Management (Self-Signed & CA)

*Failure to renew certificates breaks API integrations, SnapMirror authentications, AD LDAP over SSL, and ONTAP System Manager GUI access.*

### 5.1 Audit & Identify Expiring Certificates

```bash
# View all certificates expiring within the next 30 days
security certificate show -fields vserver,common-name,serial,expiration -expiration <30d
```

*(Write down the `serial` number of the expiring certificate for cleanup).*

### 5.2A The Self-Signed Renewal Process (Internal/Non-Prod)

```bash
# Generate a new 365-day self-signed certificate
security certificate generate-self-signed -vserver <SVM_Name> -common-name <FQDN_or_IP> -size 2048 -days 365
```

*(Write down the new `serial` number output by this command and skip to 5.3).*

### 5.2B The Trusted CA Renewal Process (CSR) (Production/Enterprise)

```bash
# 1. Generate the CSR (Adjust State, Locality, and Org as needed)
security certificate generate-csr -vserver <SVM_Name> -common-name <FQDN_or_IP> -size 2048 -country US -state NY -locality "New York" -organization "MyCompany"

# 2. Provide the output CSR text to your Security Team (DigiCert / MS AD CS).
# 3. Once they return the Base64 certificates, install the CA Chain FIRST:
security certificate install -vserver <SVM_Name> -type server-ca

# 4. Install the Server Certificate:
security certificate install -vserver <SVM_Name> -type server
```

### 5.3 Bind the New Certificate to SSL

*Applying the new certificate to the live SSL service.*

```bash
# Bind the new certificate using its Serial Number
security ssl modify -vserver <SVM_Name> -server-enabled true -client-enabled false -serial <NEW_Serial_Number>
```

### 5.4 Post-Renewal Cleanup

```bash
# Delete the old expired certificate to prevent audit warnings
security certificate delete -vserver <SVM_Name> -common-name <FQDN_or_IP> -serial <OLD_Serial_Number> -type server
```

-----

\<a id="references"\>\</a\>

## 📚 Official NetApp Documentation Reference

*All commands utilized in this SOP are sourced from the official NetApp ONTAP 9 Documentation Center.*

| Scheduled Change / Task | Official NetApp Documentation Reference |
| :--- | :--- |
| **Volume Move** | [Docs: volume move start](https://docs.netapp.com/us-en/ontap-cli/volume-move-start.html) |
| **Aggregate Relocation** | [Docs: storage aggregate relocation start](https://docs.netapp.com/us-en/ontap-cli/storage-aggregate-relocation-start.html) |
| **SVM DR (SnapMirror)** | [Docs: snapmirror break](https://docs.netapp.com/us-en/ontap-cli/snapmirror-break.html) |
| **SVM Operations** | [Docs: vserver start](https://docs.netapp.com/us-en/ontap-cli/vserver-start.html) |
| **Disk Firmware Update** | [Docs: storage firmware download](https://docs.netapp.com/us-en/ontap-cli/storage-firmware-download.html) |
| **Audit Certificates** | [Docs: security certificate show](https://docs.netapp.com/us-en/ontap-cli/security-certificate-show.html) |
| **Generate Self-Signed / CSR** | [Docs: security certificate generate](https://www.google.com/search?q=https://docs.netapp.com/us-en/ontap-cli/security-certificate-generate-self-signed.html) |
| **Bind SSL / Modify** | [Docs: security ssl modify](https://docs.netapp.com/us-en/ontap-cli/security-ssl-modify.html) |

```
