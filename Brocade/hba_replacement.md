# 🔄 SOP: Brocade SAN Zoning Update for HBA Replacement

When an HBA is replaced on a server, its underlying WWPN changes. Depending on your organization's zoning standards, you will either need to update an existing host-based alias or create a brand new WWPN-based alias. Both methods are detailed below.

## 📑 Table of Contents
* [📋 Prerequisites](#-prerequisites)
* [1️⃣ Method 1: Update Existing Alias (Hostname-based)](#1-method-1-update-existing-alias-hostname-based)
* [2️⃣ Method 2: Create a New Alias (WWPN-based)](#2-method-2-create-a-new-alias-wwpn-based)
* [💾 Step 3: Save and Enable the Configuration](#-step-3-save-and-enable-the-configuration)
* [✅ Step 4: Final Verification](#-step-4-final-verification)

---

### 📋 Prerequisites
* **🛑 Old WWPN:** (e.g., 10:00:00:90:fa:12:34:56)
* **✅ New WWPN:** (e.g., 10:00:00:90:fa:98:76:54)
* **📂 Active Config Name:** (Find this by running `cfgshow`)
* **🏷️ Zone Name:** (e.g., Zone_Server01_Storage)

---

### 1️⃣ Method 1: Update Existing Alias (Hostname-based)
*Use this method if your aliases are named after the server (e.g., Server01_HBA1). This is the recommended industry standard.*

**1. Verify the New HBA is Logged In:**
Ensure the new HBA is physically connected and logging into the fabric.
    
    nodefind 10:00:00:90:fa:98:76:54

**2. Add the New WWPN to the Existing Alias:**
Inject the new WWPN into the server's current alias.
    
    aliadd "Server01_HBA1", "10:00:00:90:fa:98:76:54"

**3. Remove the Old WWPN from the Alias:**
Strip the dead WWPN out of the alias so only the new hardware remains.
    
    aliremove "Server01_HBA1", "10:00:00:90:fa:12:34:56"

**4. Verify the Alias Modification:**
Check your work to ensure the alias now only contains the new WWPN.
    
    alishow "Server01_HBA1"

*(Skip to Step 3 to save and enable)*

---

### 2️⃣ Method 2: Create a New Alias (WWPN-based)
*Use this method if your aliases are named directly after the WWPNs themselves (e.g., Alias_10000090fa123456).*

**1. Verify the New HBA is Logged In:**
    
    nodefind 10:00:00:90:fa:98:76:54

**2. Create the New Alias:**
Create a brand new alias for the new WWPN.
    
    alicreate "Alias_10000090fa987654", "10:00:00:90:fa:98:76:54"

**3. Add the New Alias to the Existing Zone:**
Inject the newly created alias into the active zone.
    
    zoneadd "Zone_Server01_Storage", "Alias_10000090fa987654"

**4. Remove the Old Alias from the Zone:**
Take the old HBA's alias out of the zone.
    
    zoneremove "Zone_Server01_Storage", "Alias_10000090fa123456"

**5. Delete the Old Alias (Cleanup):**
Remove the dead alias from the database to keep the fabric configuration clean.
    
    alidelete "Alias_10000090fa123456"

---

### 💾 Step 3: Save and Enable the Configuration
Regardless of which method you used above, you must save the changes and push them to the active fabric.

    # Save the changes to the defined configuration database
    cfgsave

    # Push the changes to the active fabric (Crucial for the storage to see the change)
    cfgenable "Your_Active_Config_Name"

*(⚠️ Note: Press `y` when the switch prompts you to confirm. This is non-disruptive to other traffic).*

---

### ✅ Step 4: Final Verification
Ensure the zone is active and the new WWPN/Alias is now successfully zoned.

    zoneshow "Your_Zone_Name"
