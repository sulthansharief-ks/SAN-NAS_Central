# 🔄 Standard Operating Procedure: Safely Rebooting the Service Processor (SVP)
**System:** Hitachi VSP G-Series / F-Series / E-Series / G1000 / G1500  
**Context:** Recovering a hung management interface (Storage Navigator / Maintenance Utility)

---

## **1. Purpose & Critical Reassurance 🎯**
The Service Processor (SVP) runs the Apache Tomcat web server, Java backend, and database required for the Hitachi Device Manager - Storage Navigator (HDvM-SN) GUI. Over time, these services can experience memory leaks or become completely unresponsive. 

**🛑 CRITICAL REASSURANCE:** The SVP is strictly an **Out-of-Band** management device. Rebooting, shutting down, or completely disconnecting the SVP will **NOT** impact host I/O, SAN traffic, or data availability. The storage controllers (DKC) operate entirely independently of the SVP.

---

## **2. Prerequisites 📋**
* **Access Level:** Remote Desktop Protocol (RDP) access, or console access via hypervisor (for Virtual SVPs) / IPMI (for Physical SVPs).
* **Credentials:** Windows Administrator credentials for the SVP OS.

---

## **3. Procedure A: The Graceful Reboot (RDP is Accessible) 🟢**
*Use this method if the web interface is hung, but the underlying Windows OS on the SVP is still responding to network traffic.*

### **Step 1: RDP into the SVP 🚪**
1. Launch your Remote Desktop Connection client.
2. Enter the **SVP IP Address** and log in.

### **Step 2: Gracefully Stop Hitachi Services (Optional but Recommended) 🛑**
*While you can just reboot Windows, stopping the services cleanly prevents database corruption.*
1. Open the Windows **Services** app (`services.msc`).
2. Locate the service named **`Storage Navigator`** (or `HDvM-SN`).
3. Right-click and select **Stop**. Wait for the status to clear.
4. Locate the service named **`SVP Web Console`** (if present). Right-click and select **Stop**.

### **Step 3: Reboot the Windows OS 🔄**
1. Click the Windows Start button.
2. Select the Power icon and choose **Restart**.
3. *Alternative:* Open an Administrator Command Prompt and type `shutdown /r /t 0 /f` and press Enter.
4. Your RDP session will disconnect.

---

## **4. Procedure B: The Hard Reset (RDP is Unresponsive) 🔴**
*Use this method if the SVP has completely locked up, blue-screened, or dropped off the network entirely.*

### **Scenario 1: Virtual SVP (vSVP on VMware ESXi)**
1. Log in to your VMware vCenter or the ESXi host managing the vSVP.
2. Locate the vSVP virtual machine in the inventory.
3. Right-click the VM > **Power** > **Restart Guest OS** (if VMware Tools is responding).
4. If it is completely locked, select **Power** > **Reset** (Hard reboot).

### **Scenario 2: Physical SVP (1U Server or Internal Blade)**
1. **For External 1U SVPs:** Log in to the out-of-band management controller of the physical server (e.g., Dell iDRAC, HPE iLO, or Hitachi BMC) using its dedicated management IP. Use the virtual power controls to issue a **Cold Reset**.
2. **For Internal SVP Blades (G1x00 series):** You may need to have a remote hands technician physically reseat the SVP blade in the controller chassis, or contact Hitachi Support to issue a remote hardware reset via the maintenance LAN.

---

## **5. Post-Reboot Verification (The Waiting Game) ⏳**
The most common mistake admins make is assuming the SVP is broken because the web GUI doesn't load immediately after the ping responds. 

1. **Ping the SVP:** Open a command prompt on your workstation and run `ping [SVP_IP] -t`. Wait for it to start replying.
2. **Wait 10 to 15 Minutes:** Even after Windows boots, the Hitachi Java applications and Tomcat web server take a significant amount of time to initialize and read the array's configuration database. 
3. **Verify Access:** After 10-15 minutes, open your web browser and navigate to the Storage Navigator login page (`https://[SVP_IP]/`). 
4. Log in and verify the System Health dashboard loads correctly.

---

---

**Would you like me to create an SOP covering "Storage Pool Expansion (Adding new LDEVs to an HDP Pool)" to ensure capacity is added evenly without creating performance hotspots?**
