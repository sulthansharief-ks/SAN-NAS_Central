# 🚨 The Exhaustive Brocade Port Error SOP (Root Cause & Justification Matrix)

To properly diagnose Brocade port issues, you must look at both `porterrshow` and `portstatsshow`. This exhaustive matrix covers every counter, the exact location of the fault (Server, Switch, Storage, or Cable), the recommended action, and the technical justification for that action.

---

## 🚦 Category 1: The "Big 3" Physical Drops
These indicate the physical link is dropping entirely.

* **`loss sig` (Loss of Signal):**
  * **Why:** The laser light physically stopped hitting the receiver.
  * **Fault Location:** Remote Port (Server HBA / Storage Target) or the Fiber Cable.
  * **Action:** Check if the remote server rebooted or the cable was unplugged. If unexpected, check cable seating or replace the remote SFP.
  * **Justification:** The switch's receiver is completely blind. If the RX power is 0, the issue is exclusively upstream. Replacing the switch SFP will not fix a lack of incoming light.

* **`loss sync` (Loss of Synchronization):**
  * **Why:** The switch sees light, but cannot lock onto the digital bit-stream (loss of word sync). 
  * **Fault Location:** Fiber Cable, Switch SFP, or Remote SFP (Server/Storage).
  * **Action:** Clean fiber optics. Check for speed negotiation mismatches (e.g., forcing 32G into an 8G HBA).
  * **Justification:** Light is reaching the switch, but it is distorted (scattered by dust) or arriving at the wrong frequency. Cleaning removes physical scattering, and speed matching ensures aligned clock rates.

* **`link fail` (Link Failure):**
  * **Why:** The port state machine dropped to an offline state. This almost always increments exactly after a `loss sig` or `loss sync`.
  * **Fault Location:** Remote Host (Reboot) or Physical Media.
  * **Action:** Treat as a physical layer fault. Clean fiber, swap cables, or replace SFPs.
  * **Justification:** A link failure is the definitive symptom of a broken physical layer protocol. Fixing the underlying signal/sync issue automatically resolves link failures.

---

## 🧬 Category 2: Data Link & Primitive Errors (The "Hidden" Counters)
These counters (often seen in `portstatsshow`) show the link trying to recover itself before fully failing.

* **`lr_in` / `lr_out` (Link Reset In/Out):**
  * **Why:** The port received or transmitted a primitive signal asking to reset the link. This is the port trying to fix a synchronization issue without completely dropping the connection.
  * **Fault Location:** The Transmitting Port (Switch for `lr_out`, Server/Storage for `lr_in`) or the Cable.
  * **Action:** If actively incrementing, the link is extremely marginal. Clean the optics or replace the patch cable.
  * **Justification:** The port is actively detecting marginal link quality and attempting to heal it without dropping FC traffic. Replacing the physical media removes the degradation causing the resets.

* **`ols_in` / `ols_out` (Offline Sequence In/Out):**
  * **Why:** The port received or transmitted an explicit command to go offline. 
  * **Fault Location:** Administrative action or Remote Host firmware.
  * **Action:** Usually happens during switch reboots, port disables, or server reboots. Ignore if historical; investigate the remote host if it happens randomly.
  * **Justification:** This is an intentional software command, not a hardware failure. Upgrading host drivers stops anomalous offline sequences.

* **`rx_bad_os` / `tx_bad_os` (Bad Ordered Sets):**
  * **Why:** Fibre channel uses "Ordered Sets" (Start of Frame, End of Frame, etc.). This counter means these control words arrived corrupted.
  * **Fault Location:** Fiber Cable or Transmitting SFP.
  * **Action:** Indicates a dirty optical connection or failing laser. Clean/replace physical media.
  * **Justification:** Ordered sets are generated at the lowest hardware level. Corruption here indicates a failing laser or dirty glass scattering the light beam.

---

## 🛡️ Category 3: Encoding & Forward Error Correction (16G/32G+)
With Gen 6 and Gen 7 switches, encoding errors and FEC are your primary leading indicators of failure.

* **`enc out` (Encoding Out):**
  * **Why:** 8b/10b encoding failed *outside* data frames (during idle primitives). Normal during port initialization.
  * **Fault Location:** Fiber Cable or Transmitting SFP (Server/Storage).
  * **Action:** Ignore if it only happened during a server boot. If constantly incrementing while online, clean/swap the fiber.
  * **Justification:** Bit encoding is Layer 1. If bits are flipping during idle transmission, the laser is weak or the cable is severely degraded.

* **`pcs err` (Physical Coding Sublayer):**
  * **Why:** The 64b/66b equivalent of `enc out`, used on 16G, 32G, and 64G links. 
  * **Fault Location:** Fiber Cable or Transmitting SFP.
  * **Action:** Same as `enc out`. Replace cable/SFP if actively incrementing.
  * **Justification:** Same as `enc out`; indicates Layer 1 physical degradation.

* **`cor_fec` (Corrected FEC Blocks):**
  * **Why:** FEC found bit errors and mathematically fixed them. 
  * **Fault Location:** Normal link degradation.
  * **Action:** **Do nothing.** This is normal at 32G/64G speeds. The switch is doing its job.
  * **Justification:** This is expected behavior at high speeds. The switch ASIC is doing its job to maintain data integrity over high-speed links.

* **`uncor_fec` (Uncorrected FEC Blocks):**
  * **Why:** The link is so degraded that FEC could not mathematically fix the corruption. This will result in dropped frames and `crc err`.
  * **Fault Location:** Fiber Cable or Transmitting SFP.
  * **Action:** Immediate physical layer troubleshooting (Clean -> Swap Cable -> Swap SFP).
  * **Justification:** The physical degradation has surpassed the algorithm's ability to recover data, leading directly to dropped frames and CRC errors.

---

## 🧩 Category 4: Payload Integrity (Corrupted Frames)
The link is active, but the actual data inside the pipe is getting scrambled.

* **`crc err` (Cyclic Redundancy Check):**
  * **Why:** A frame's checksum did not match its payload. The frame was corrupted in flight.
  * **Fault Location:** Transmitting Port (Server HBA / Storage) or the Cable.
  * **Action:** The transmitter on the *other* side of the link is usually failing. Swap the patch cable, then replace the SFP in the remote HBA/Storage array.
  * **Justification:** The switch is successfully receiving light, but the data is scrambled. The switch is the victim; the remote sender is the culprit corrupting the frame before or during transit.

* **`crc g_eof` (CRC with Good End-of-Frame):**
  * **Why:** The switch received a corrupted frame, but the EOF marker was intact. It flags it so downstream switches don't count it.
  * **Fault Location:** Exclusively the physical link attached to this specific switch port.
  * **Action:** This proves the corruption happened on *this exact physical link*. Replace cables/SFPs here.
  * **Justification:** The intact EOF proves the frame was not corrupted upstream and simply passed along. It guarantees the corruption happened on this specific wire.

* **`enc in` (Encoding In):**
  * **Why:** Bit-level encoding failed *inside* the data payload.
  * **Fault Location:** Transmitting Port or Cable.
  * **Action:** Almost always increments with `crc err`. Replace physical media.
  * **Justification:** Often accompanies CRC errors; indicates bit-flips during payload transmission due to marginal media.

* **`bad eof` (Bad End-of-Frame):**
  * **Why:** A frame arrived with a corrupted or missing EOF delimiter.
  * **Fault Location:** Transmitting Port or Cable.
  * **Action:** Indicates physical corruption. Clean fiber, swap SFPs.
  * **Justification:** Hardware-level corruption of the delimiter indicates failing lasers or dirty optics.

* **`too shrt` (Too Short):**
  * **Why:** A frame arrived smaller than the minimum 38 bytes.
  * **Fault Location:** Remote Host HBA (Hardware or Driver).
  * **Action:** Usually a host HBA driver issue or faulty HBA firmware generating bad packets.
  * **Justification:** Cables do not resize frames. Only a malfunctioning host ASIC or buggy driver can generate and transmit mathematically illegal frame sizes.

* **`too long` (Too Long):**
  * **Why:** A frame exceeded the maximum 2148 bytes.
  * **Fault Location:** Remote Host HBA (Hardware or Driver).
  * **Action:** Host HBA driver bug or corrupted EOF causing two frames to merge. Update HBA drivers.
  * **Justification:** Similar to short frames, this is usually a host ASIC/driver flaw, or an EOF marker was lost, causing the switch to read two frames as one.

* **`trunc_rx` / `trunc_tx` (Truncated Frames):**
  * **Why:** A frame was cut off mid-transmission.
  * **Fault Location:** Physical Link (Cable/SFP).
  * **Action:** Usually caused by a sudden `loss sig` or `loss sync` while data was actively flowing. 
  * **Justification:** Data was flowing and the physical link abruptly died, cutting the frame in half. Fix the underlying physical issue.

---

## 🐢 Category 5: Congestion & Flow Control
These errors prove the physical cables are perfect, but the devices are overwhelmed or out of buffer credits.

* **`disc c3` (Discard Class 3):**
  * **Why:** The switch was forced to drop a frame.
  * **Fault Location:** Storage Target (Slow Drain) or Switch Routing.
  * **Action:** Check `c3timeout`. If timeouts are high, it's a slow drain. If timeouts are zero, check if the destination port is offline or zoning is incorrect.
  * **Justification:** Fibre Channel guarantees delivery via buffer credits. If a switch drops a frame, it means a destination device violated the credit rules by holding up the fabric.

* **`c3timeout tx` (Transmit Timeout - The "Slow Drain" Counter):**
  * **Why:** The switch waited to send a frame out this port, but the destination device had zero buffer credits available. After ~2 seconds, the switch threw the frame away.
  * **Fault Location:** The Storage Target or ISL connected to this port.
  * **Action:** The device connected to this port is your bottleneck. Check storage array performance, host IOPS limits, or allocate more buffer credits to the port.
  * **Justification:** The switch is ready to transmit, but the storage array is too busy processing disk I/O to accept new network frames. Fixing the storage bottleneck stops the timeouts.

* **`c3timeout rx` (Receive Timeout):**
  * **Why:** A frame sat in the switch's receive buffer too long because it couldn't find an outbound route.
  * **Fault Location:** Core Switch ISLs or Fabric-Wide Congestion.
  * **Action:** Indicates fabric-wide congestion. Check your ISL utilization.
  * **Justification:** The host is sending data fine, but the switch lacks the core bandwidth to forward the traffic to its final destination.

* **`c3 drops`:**
  * **Why:** Aggregate of Class 3 frames dropped due to timeouts or unreachable destinations.
  * **Fault Location:** Zoning or Storage Paths.
  * **Action:** Review zoning and storage paths.
  * **Justification:** A high-level indicator of routing or congestion issues requiring architectural review.

---

## 🛑 Category 6: Fabric Routing Rejections
These are logical rules being violated, not hardware failures.

* **`frjt` (Fabric Reject):**
  * **Why:** The switch rejected a frame. Usually because a host is trying to talk to a WWPN it is not zoned to see.
  * **Fault Location:** Switch Zoning Configuration or Host LUN Masking.
  * **Action:** Check your active zoning configuration. Verify the host HBA is not attempting to scan LUNs on unzoned arrays.
  * **Justification:** The switch ASIC enforces security. A reject means the host is actively trying to communicate with an unauthorized or offline WWPN. Fixing the zoning map resolves the violation.

* **`fbsy` (Fabric Busy):**
  * **Why:** The destination port was too busy to accept traffic (very rare in modern Class 3 SANs).
  * **Fault Location:** Destination Port.
  * **Action:** Check the destination port for extreme congestion or bottlenecking.
  * **Justification:** The target ASIC is overwhelmed and actively telling the fabric to hold off.

* **`prjt` / `pbsy` (Port Reject / Port Busy):**
  * **Why:** Similar to fabric reject/busy, but generated directly by the N_Port (the end device), not the switch.
  * **Fault Location:** Remote Storage Array Controller.
  * **Action:** The end device is overwhelmed or rejecting logins. Check the storage array controller logs.
  * **Justification:** The switch successfully delivered the frame, but the storage array's internal CPU or security rules rejected it. Action must be taken on the storage management console.
