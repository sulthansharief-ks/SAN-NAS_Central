# 🐧 RHEL NFS Command Cheat Sheet (Client & Firewall) 🛠️

This cheat sheet provides the critical discovery, mounting, tuning, and firewall commands required when connecting Red Hat Enterprise Linux (RHEL) hosts to enterprise storage arrays (like NetApp, Hitachi VSP, and Dell PowerMax). It includes the ideal target values and expected outputs for every validation step.

## 📑 Table of Contents
1. [🔍 Discovery Commands](#discovery)
2. [💾 Mounting Commands](#mounting)
3. [⚙️ Verification & Tuning](#verification)
4. [🔌 Unmounting Commands](#unmounting)
5. [🧱 Firewalld Commands (RHEL)](#firewall)

---

<a id="discovery"></a>
## 🔍 1. Discovery Commands
*Use this to verify the Linux host can see the target path on the storage controller before attempting to mount.*

```bash
# List all available NFS exports on the storage array
showmount -e <storage-IP>
```
* **Ideal Output:** A list of export paths that explicitly includes your target `/<export-path>` and shows it is exported to either your client's specific IP, subnet, or `*` (everyone).

---

<a id="mounting"></a>
## 💾 2. Mounting Commands


*Using `hard`, `tcp`, and large 64k/1M block sizes ensures stable, high-throughput connections to enterprise storage systems.*

```bash
# Temporarily mount the NFS export with optimized enterprise options
sudo mount -t nfs -o rw,hard,tcp,rsize=65536,wsize=65536 <storage-IP>:/<export-path> /<local-mount-point>
```
* **Ideal Output:** The command should return silently. Any error messages (like `access denied` or `connection timed out`) indicate an immediate failure.

### Persistent Mounting (`/etc/fstab`)
*Ensures the mount automatically reconnects after a Linux reboot. The `_netdev` flag is critical as it tells RHEL to wait for the network stack to initialize before attempting the mount.*

```text
# Add this syntax to your /etc/fstab file
<storage-IP>:/<export-path>    /<local-mount-point>    nfs    rw,hard,tcp,rsize=65536,wsize=65536,_netdev    0 0
```

---

<a id="verification"></a>
## ⚙️ 3. Verification & Tuning

```bash
# Show the exact mount options currently active in the Linux kernel
nfsstat -m
```
* **Ideal Value:** The output must explicitly confirm `hard`, `proto=tcp`, `vers=3` (or `4.1`), and your requested block sizes (e.g., `rsize=65536`, `wsize=65536` or `1048576`). It must **never** show `soft` or `udp` for enterprise databases/applications.

```bash
# Display the size, usage, and mount point of all active NFS filesystems
df -hT | grep nfs
```
* **Ideal Value:** The filesystem type should show as `nfs` or `nfs4`. The command should return instantly without hanging, and the total capacity should accurately reflect the volume size provisioned on the NetApp/Hitachi array.

```bash
# Check the current RPC slot table limit (concurrent requests)
sysctl sunrpc.tcp_slot_table_entries
```
* **Ideal Value:** For enterprise environments, the value should be `128` or `256`. If it is at the default (`16`) or misconfigured (e.g., `2`), the client will artificially bottleneck its own performance.

```bash
# Increase the concurrent RPC request limit dynamically (Fixes bottlenecks under heavy load)
sudo sysctl -w sunrpc.tcp_slot_table_entries=128
```
* **Ideal Output:** The terminal should echo back `sunrpc.tcp_slot_table_entries = 128`, confirming the new limit is actively applied to the kernel.

---

<a id="unmounting"></a>
## 🔌 4. Unmounting Commands

```bash
# Gracefully disconnect the NFS export
sudo umount /<local-mount-point>
```
* **Ideal Output:** Returns silently. If it says `target is busy`, a user or application is currently reading/writing to the path.

```bash
# Force unmount (Use this if the storage array is unreachable and the mount is hanging)
sudo umount -f /<local-mount-point>

# Lazy unmount (Immediately detaches the filesystem from the directory tree and cleans up in the background)
sudo umount -l /<local-mount-point>
```

---

<a id="firewall"></a>
## 🧱 5. Firewalld Commands (RHEL)

### For NFSv4 (Modern standard, single port)
*NFSv4 is firewall-friendly and only requires TCP port 2049.*

```bash
# Allow NFS service through the firewall permanently
sudo firewall-cmd --permanent --add-service=nfs

# Reload the firewall to apply the changes
sudo firewall-cmd --reload
```
* **Ideal Output:** Both commands should return `success`.

### For NFSv3 (Legacy, multi-port)
*NFSv3 requires `rpcbind` and `mountd` in addition to the main NFS port.*

```bash
# Allow NFS, RPC-Bind, and Mountd services
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd

# Reload the firewall to apply the changes
sudo firewall-cmd --reload
```
* **Ideal Output:** All commands should return `success`.

### Verification
```bash
# List all currently allowed services in the active zone
sudo firewall-cmd --list-services
```
* **Ideal Value:** The output string should contain `nfs` (and `rpc-bind`, `mountd` if using v3), verifying the ports are open and listening.
