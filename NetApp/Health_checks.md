# 🏥 NetApp ONTAP Health Check Guide 🚀

This document provides a comprehensive checklist and command set for performing health checks on NetApp ONTAP clusters. It covers system health, high availability, storage, networking, and logs.

---

## 📋 1. Cluster & System General Health
*Check the heartbeat of the system to ensure all nodes are online and eligible.*

```bash
# Verify all nodes are healthy and eligible (Look for "true")
cluster show

# Check the overall system health status (Target: "ok")
system health status show

# View active health alerts and recommended actions
system health alert show

# Check specific subsystems (Fans, Power, SAS, etc.)
system health subsystem show

# Verify node model, uptime, and ONTAP version
system node show -fields uptime, model, version
```

---

## ⚡ 2. High Availability (HA) & Failover
*Critical checks to ensure the cluster can survive a controller failure.*

```bash
# 🚨 CRITICAL: Verify takeover is possible (Target: "true")
storage failover show

# Check the status of the HA interconnect (Heartbeat)
storage failover interconnect show

# Verify Hardware Assist status (accelerates failover)
storage failover hwassist show

# Check for any unassigned disks (Spare disks are good, unassigned are not)
storage disk show -container-type unassigned
```

---

## 💾 3. Storage Health (Aggregates & Disks)
*Ensure physical media is healthy and logical containers have space.*

```bash
# List any broken or failed disks
storage disk show -state broken

# Check for physical disk errors
storage disk show -errors

# Verify all aggregates are online
storage aggregate show

# ⚠️ SPACE CHECK: Show aggregates over 90% capacity
storage aggregate show-space -percent-used >90%

# Verify shelf connectivity and multi-pathing
storage shelf show -connectivity
```

---

## 🌐 4. Network & Connectivity
*Verify that data paths (LIFs) and physical ports are stable.*

```bash
# Check status of Logical Interfaces (LIFs)
# Ensure 'Status/Oper' is up/up and 'Is Home' is true
network interface show

# Identify LIFs that are NOT at home (migrated due to failure)
network interface show -is-home false

# Check physical port link status and speed
network port show

# Verify the internal Cluster Network is healthy
network port show -role cluster

# Check failover groups (Where do LIFs go if a port dies?)
network interface failover-groups show
```

---

## 📝 5. Events, Logs & AutoSupport
*Review historical data for intermittent issues.*

```bash
# Show critical alerts from the last 24 hours
event log show -severity emergency|alert|critical

# Verify AutoSupport (Call Home) is working
system node autosupport history show -status failed|transmission-failed

# Check for environmental issues (Temp, Chassis, etc.)
system health alert show -subsystem Environment
```

---

## ⚡ Quick Reference: The "Morning Coffee" One-Liner
*Run this single command string to check the pulse of the cluster in 5 seconds.*

```bash
cluster show; storage failover show; system health alert show; network interface show -is-home false
```

---
*Sulthan Sharief K S*
