# 🛠️ SOP: Brocade SAN Switch Base Network Configuration

After deploying a new Brocade SAN switch or performing a factory reset, it is critical to configure core network and alerting services. This SOP covers Hostname configuration, NTP, Timezones, DNS, SMTP (Email Alerts), Syslog, and SNMP (v2c/v3).

## 📑 Table of Contents
* [📛 1. Hostname Configuration](#-1-hostname-configuration)
* [⏱️ 2. NTP & Timezone Configuration](#️-2-ntp--timezone-configuration)
* [🌐 3. DNS Configuration](#-3-dns-configuration)
* [📧 4. SMTP & Email Alerting (MAPS)](#-4-smtp--email-alerting-maps)
* [📜 5. Remote Syslog Configuration](#-5-remote-syslog-configuration)
* [📡 6. SNMP Configuration (v2c & v3)](#-6-snmp-configuration-v2c--v3)
* [✅ 7. Final Verification](#-7-final-verification)

---

### 📛 1. Hostname Configuration
Setting a descriptive hostname is the first step in making your switch easily identifiable in logs, monitoring tools, and fabric topology views.

**1. Set the new switch name:**
    
    switchname "Your_New_Switch_Name"

*(⚠️ Note: The switch name can be up to 30 characters long. It must begin with an alphabetic character and can contain alphanumeric characters, hyphens, and underscores. Spaces are not allowed.)*

**2. Verify the updated name:**
    
    switchname

---

### ⏱️ 2. NTP & Timezone Configuration
Accurate timekeeping is crucial for logs, troubleshooting, and SSL certificate validation.

**1. Set the Timezone:**
    
    tstimezone --interactive

*(Follow the interactive prompts to select your specific region and country).*

**2. Configure the NTP Server(s):**
If you have multiple internal NTP servers, separate them with a semicolon.
    
    tsclockserver "10.0.0.51;10.0.0.52"

**3. Verify Time Settings:**
    
    date
    tsclockserver

---

### 🌐 3. DNS Configuration
DNS allows the switch to resolve hostnames for syslog, SMTP, and NTP servers instead of relying purely on IP addresses.

**1. Set the DNS domain and name servers:**
    
    dnsconfig

*The system will prompt you interactively:*
* `Domain Name:` (e.g., storage.company.local)
* `Name Server IP Address 1:` (e.g., 10.0.0.10)
* `Name Server IP Address 2:` (e.g., 10.0.0.11)

**2. Verify DNS:**
    
    dnsconfig

---

### 📧 4. SMTP & Email Alerting (MAPS)
Brocade uses the Monitoring and Alerting Policy Suite (MAPS) to send email notifications for hardware or fabric events.

**1. Configure the SMTP Relay Server:**
Point the switch to your internal mail relay server.
    
    relayconfig --config -relay 10.0.0.25

**2. Set the Alert Email Addresses:**
Specify where the alerts should be sent (use a comma for multiple addresses).
    
    mapsconfig --emailcfg -address "san-admin@company.com,soc@company.com"

**3. Enable Email Actions in MAPS:**
Ensure MAPS is globally configured to send emails when rules are triggered.
    
    mapsconfig --actions mail

**4. Test the Email Alert:**
Send a test email to verify the relay is accepting messages from the switch IP.
    
    mapsconfig --testmail

---

### 📜 5. Remote Syslog Configuration
Sending switch logs to a centralized Syslog server (like Splunk or LogRhythm) is essential for auditing and historical troubleshooting.

**1. Set the Syslog Server IP:**
    
    syslogadmin --set -ip 10.0.0.100

**2. Verify Syslog Configuration:**
    
    syslogadmin --show -ip

---

### 📡 6. SNMP Configuration (v2c & v3)
SNMP is used by monitoring tools (like Broadcom SANnav, SolarWinds, etc.) to poll switch health and receive traps.

#### Option A: Configure SNMP v2c (Legacy/Basic)
*Note: Brocade FOS groups v1 and v2c configurations together.*

**1. Start the interactive configuration:**
    
    snmpconfig --set snmpv1

**2. Answer the prompts:**
* `Community (ro):` Enter your read-only community string (e.g., `public` or a custom string).
* `Trap Recipient's IP address:` Enter your monitoring server IP.
* `Trap recipient Severity level:` Set to `4` (Warning) or `5` (Error) depending on your noise preference.

#### Option B: Configure SNMP v3 (Secure/Recommended)
SNMP v3 provides encryption and authentication.

**1. Start the interactive configuration for a specific user (e.g., user1):**
    
    snmpconfig --set snmpv3 -user 1

**2. Answer the prompts for secure access:**
* `Auth Protocol [MD5(1)/SHA(2)/noAuth(3)]:` Select `2` for SHA.
* `Auth Password:` Enter your secure authentication password.
* `Priv Protocol [DES(1)/noPriv(2)/AES128(3)/AES256(4)]:` Select `3` or `4` for AES encryption.
* `Priv Password:` Enter your encryption privacy password.

**3. Configure the SNMP v3 Trap Destination:**
    
    snmpconfig --set snmpv3 -trap_recipient

*(Follow the prompts to enter the IP address of your monitoring server and link it to the user created in the previous step).*

#### 4. Verify SNMP Configuration
    
    snmpconfig --show snmpv1
    snmpconfig --show snmpv3

---

### ✅ 7. Final Verification
Run a final check to ensure all environmental variables are actively running without errors.

    # Check overall switch status and the new hostname
    switchstatusshow
    switchshow

    # Check MAPS status and dashboard
    mapsconfig --show
    mapsdb --show

---
