# 🚀 Ultimate Brocade SAN Administration Cheat Sheet (Advanced Edition)

This comprehensive cheat sheet covers everything from essential day-to-day operations to advanced fabric configurations, ISL trunking, Virtual Fabrics, and performance monitoring.

## 📑 Table of Contents
* [🏥 1. Switch Health & General Status](#-1-switch-health--general-status)
* [🔍 2. Device Discovery & Name Server](#-2-device-discovery--name-server)
* [🗺️ 3. Zoning Operations (Aliases, Zones, Configs)](#️-3-zoning-operations-aliases-zones-configs)
* [🔌 4. Port Management & NPIV](#-4-port-management--npiv)
* [🌉 5. Advanced ISL & Trunking](#-5-advanced-isl--trunking)
* [🪟 6. Virtual Fabrics (Logical Switches)](#-6-virtual-fabrics-logical-switches)
* [📈 7. Performance & Traffic Monitoring](#-7-performance--traffic-monitoring)
* [🛡️ 8. Security & User Management](#️-8-security--user-management)
* [💾 9. Backup, Restore & Firmware](#-9-backup-restore--firmware)

---

### 🏥 1. Switch Health & General Status
Use these commands to get a quick pulse on the switch hardware and fabric status.

    # Show overall switch status, IP, and basic port states
    switchshow
    
    # Show high-level health status of the switch (Healthy, Marginal, Down)
    switchstatusshow
    
    # Show all switches connected in the current fabric (Domain IDs, IPs)
    fabricshow
    
    # Identify which switch is the Principal switch in the fabric
    fabricprincipal
    
    # Display hardware details (Serial Number, Part Numbers)
    chassisshow
    
    # View environmental sensors (Temperatures, Fans, Power Supplies)
    sensorshow
    
    # Dump the switch error and event logs
    errdump

---

### 🔍 2. Device Discovery & Name Server
Use these commands to track down host HBAs or storage controllers connected to the SAN.

    # Find which switch and port a specific WWPN is connected to
    nodefind 10:00:00:90:fa:12:34:56
    
    # Show detailed information about a specific port (including logged-in WWPNs)
    portshow <port_number>
    
    # Display local Name Server database (devices logged into this specific switch)
    nsshow
    
    # Display the fabric-wide Name Server database (all logged-in devices everywhere)
    nsallshow

---

### 🗺️ 3. Zoning Operations (Aliases, Zones, Configs)
Brocade zoning follows a strict hierarchy: **WWPN ➡️ Alias ➡️ Zone ➡️ Configuration**.

#### Step A: Aliases (Friendly Names for WWPNs)
    # Create a new alias for a WWPN
    alicreate "Alias_Name", "WWPN"
    
    # Add/Remove a WWPN to/from an existing alias
    aliadd "Alias_Name", "WWPN"
    aliremove "Alias_Name", "WWPN"
    
    # Delete an alias completely
    alidelete "Alias_Name"

#### Step B: Zones (Grouping Hosts and Storage)
    # Create a new zone and add members (Aliases or WWPNs)
    zonecreate "Zone_Name", "Alias_Host; Alias_Storage"
    
    # Add/Remove a member to/from an existing zone
    zoneadd "Zone_Name", "New_Alias"
    zoneremove "Zone_Name", "Alias_to_Remove"
    
    # Delete a zone entirely
    zonedelete "Zone_Name"

#### Step C: Configurations (The Active Ruleset)
    # Create a new configuration containing specific zones
    cfgcreate "Config_Name", "Zone1; Zone2"
    
    # Add/Remove a zone to/from an existing configuration
    cfgadd "Config_Name", "New_Zone"
    cfgremove "Config_Name", "Zone_to_Remove"
    
    # Save all uncommitted zoning changes to the switch database (CRITICAL)
    cfgsave
    
    # Push the configuration to the active fabric to apply the rules (CRITICAL)
    cfgenable "Config_Name"

---

### 🔌 4. Port Management & NPIV
Manage physical interfaces and enable virtualization features for hypervisors.

    # Disable/Enable a specific port (turns the laser off/on)
    portdisable <port_number>
    portenable <port_number>
    
    # Assign a friendly name/description to a port
    portname <port_number> -n "Description_String"
    
    # Enable NPIV on a port (Required for VMware/Hyper-V virtual HBAs)
    portcfgnpivport <port_number>, 1
    
    # Clear port error statistics (useful after replacing a bad cable/SFP)
    portstatsclear <port_number>

---

### 🌉 5. Advanced ISL & Trunking
Manage Inter-Switch Links (ISLs) and optimize bandwidth between switches.

    # Show all active Inter-Switch Links (ISLs) on the switch
    islshow
    
    # Show active Trunking groups (combined ISLs for higher bandwidth/failover)
    trunkshow
    
    # Enable trunking on a specific port (requires Trunking license)
    portcfgtrunkport <port_number>, 1
    
    # Show fabric routing topology and pathing
    topologycheck

---

### 🪟 6. Virtual Fabrics (Logical Switches)
Brocade allows partitioning a single physical chassis into multiple Logical Switches.

    # Show all Logical Switches (Virtual Fabrics) created on the physical chassis
    lscfg --show
    
    # Change your administrative context to a different Logical Switch (by Fabric ID)
    setcontext <FID_Number>
    
    # Move a physical port from one Logical Switch to another
    lscfg --config <FID_Number> -port <port_number>

---

### 📈 7. Performance & Traffic Monitoring
Identify bottlenecks, slow-draining devices, and high-traffic nodes.

    # Show accumulated port errors (loss of sync, encoding errors, CRC)
    porterrshow
    
    # Show buffer credit usage (Crucial for identifying slow-drain/bottleneck devices)
    portbuffershow
    
    # Show real-time port throughput and bandwidth utilization
    portperfshow
    
    # Display top bandwidth consumers (Requires Flow Vision/Advanced licenses)
    top --show
    
    # Display active traffic flows
    flow --show

---

### 🛡️ 8. Security & User Management
Lock down access to the SAN infrastructure.

    # Add a new administrative user to the switch
    userconfig --add <username> -r admin
    
    # Change the password for the current or specified user
    passwd <username>
    
    # Show active IP filtering rules (Firewall for the management interface)
    ipfilter --show
    
    # Show SSH configuration and active keys
    seccryptocfg --show

---

### 💾 9. Backup, Restore & Firmware
Standard maintenance commands for disaster recovery and upgrades.

    # Upload the switch configuration to an FTP/SCP server (Backup)
    configupload
    
    # Download a saved configuration from an FTP/SCP server to the switch (Restore)
    configdownload
    
    # Check the currently running Fabric OS (FOS) version
    firmwareshow
    
    # Initiate a firmware upgrade (requires FTP/SCP server with FOS files)
    firmwaredownload
