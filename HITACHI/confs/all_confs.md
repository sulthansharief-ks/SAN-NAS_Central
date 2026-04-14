# 📘 Hitachi VSP Administration Guide: Naming, Syslog, and Security 
**System:** Hitachi VSP G-Series / F-Series / E-Series / Enterprise  
**Management Tool:** Hitachi Maintenance Utility (MU)  

---

## **Table of Contents 📑**
* [1. Configuring Storage System Name & SVP Hostname 🏷️](#1-configuring-storage-system-name--svp-hostname)
    * [Part A: Changing the Storage System Name](#part-a-changing-the-storage-system-name)
    * [Part B: Changing the SVP Hostname & Network Settings](#part-b-changing-the-svp-hostname--network-settings)
* [2. Configuring Syslog Forwarding 📡](#2-configuring-syslog-forwarding)
* [3. Renewing SSL/TLS Certificates 🔐](#3-renewing-ssltls-certificates)
    * [Step 1: Generate the CSR (Certificate Signing Request)](#step-1-generate-the-csr-certificate-signing-request)
    * [Step 2: Install the Signed Certificate](#step-2-install-the-signed-certificate)
    * [Step 3: Restart Web Services](#step-3-restart-web-services)

---

## <a id="1-configuring-storage-system-name--svp-hostname"></a>**1. Configuring Storage System Name & SVP Hostname 🏷️**

### **Purpose 🎯**
Proper naming conventions are critical for enterprise environments with multiple arrays. This procedure covers updating the logical **Storage System Name** (visible in HDvM-SN and API) and the **SVP Hostname** (the network identity of the Service Processor).

### **Procedure 🛠️**

#### **Part A: Changing the Storage System Name**
1. Log in to the **Maintenance Utility (MU)** using an account with Storage Administrator privileges.
2. In the left navigation pane, expand **Administration** and select **Storage System**.
3. Click the **Edit Storage System** button (bottom right).
4. Enter the new **Storage System Name** (e.g., `PROD_VSP_G800_01`). 
5. *(Optional)* Update the Location or Contact Information fields for better CMDB tracking.
6. Click **Finish**, review the summary, and click **Apply**. The change is immediate and does not disrupt I/O.

#### **Part B: Changing the SVP Hostname & Network Settings**
*Note: Changing the SVP hostname/IP will temporarily drop your web management session.*
1. In the MU, navigate to **Administration** > **Network Settings**.
2. Select **Set up Network Settings**.
3. In the Host Name field, enter the new **SVP Hostname**.
4. If updating the IP address, Subnet Mask, or Default Gateway, enter them here.
5. Click **Apply**. 
6. ⚠️ **Warning:** The SVP will restart its network services. You will lose connection to the GUI for approximately 5–10 minutes. Reconnect using the new IP/Hostname.

---

## <a id="2-configuring-syslog-forwarding"></a>**2. Configuring Syslog Forwarding 📡**

### **Purpose 🎯**
To ensure hardware failures (SIMs) and security events (Audit Logs) are centrally monitored, the VSP must forward its logs to a corporate SIEM (e.g., Splunk, QRadar, or a standard Syslog server).

### **Procedure 🛠️**
1. Log in to the **Maintenance Utility (MU)**.
2. Navigate to **Administration** > **Alert Notifications** > **Syslog**.
3. Click **Set up Syslog** (or **Edit Syslog** if already configured).
4. **Enable** the Syslog feature.
5. Enter the **IP Address** or **Hostname** of your Syslog/SIEM server.
6. Enter the **Port Number** (Standard UDP is `514`; if using TCP or TLS, enter the port specified by your security team).
7. Select the **Event Types** to forward:
   * **SIM (Service Information Messages):** Always check this for hardware/logical alerts.
   * **Audit Log:** Always check this to track administrator logins, configuration changes, and failed login attempts.
8. **Test the Connection:** Click the **Send Test Message** button. Verify with your SOC/NOC team that they received the Hitachi test string.
9. Click **Apply** to save the configuration.

---

## <a id="3-renewing-ssltls-certificates"></a>**3. Renewing SSL/TLS Certificates 🔐**

### **Purpose 🎯**
By default, the SVP and GUM use self-signed certificates, which trigger security warnings in browsers and fail enterprise vulnerability scans. This procedure outlines how to generate a Certificate Signing Request (CSR) and install a CA-signed SSL/TLS certificate.

### **Prerequisites 📋**
* Access to the Corporate Certificate Authority (CA) to sign the generated request.
* Security Administrator privileges on the VSP.
* Scheduled maintenance window (installing the certificate requires a restart of the web services).

### **Procedure 🛠️**

#### **Step 1: Generate the CSR (Certificate Signing Request) 📝**
1. Log in to the **Maintenance Utility (MU)**.
2. Navigate to **Administration** > **Security** > **SSL/TLS Server Certificate**.
3. Click **Create CSR**.
4. Fill out the mandatory X.509 fields:
   * **Common Name (CN):** The FQDN of the SVP (e.g., `vsp01.yourdomain.local`).
   * **Organization (O) / Organizational Unit (OU) / City / State / Country.**
   * **Key Size:** Select `2048` or `4096` bits.
5. Click **Finish** and **Apply**. 
6. Once generated, click **Download CSR**. Send this `.csr` text file to your security team to be signed.

#### **Step 2: Install the Signed Certificate 📥**
*Do not proceed until your security team returns the signed certificate (usually a `.cer`, `.crt`, or `.pem` file).*
1. Navigate back to **Administration** > **Security** > **SSL/TLS Server Certificate**.
2. Click **Install Certificate**.
3. You will be prompted to upload the certificate file. 
   * *Important:* Ensure the certificate format matches what Hitachi expects (Base64 encoded X.509). If a chain (Intermediate/Root CA) is required, ensure it is bundled correctly per Hitachi documentation or uploaded in the specific CA chain section of the MU.
4. Click **Apply**.

#### **Step 3: Restart Web Services 🔄**
1. A warning will appear stating that the SVP web services must be restarted for the new certificate to take effect.
2. Acknowledge the prompt. The SVP Tomcat services will restart.
3. Wait 10 minutes, then navigate to the SVP's FQDN in your browser (`https://vsp01.yourdomain.local`). 
4. Verify the browser padlock icon shows a secure connection with your corporate CA details.
