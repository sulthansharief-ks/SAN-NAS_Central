Yes, you absolutely can! 🎯 

While using a floating DNS name is the "traditional" method, Commvault officially supports a much more seamless architecture that completely eliminates the need for manual DNS changes during a failover. 

You can achieve this by implementing a **Commvault Proxy** (also known as a Network Gateway) in front of your CommServe nodes. 

Here is exactly how this works and how to set it up, based on Commvault's official network topology documentation:

### 🏗️ The Proxy Architecture
Instead of clients trying to resolve a floating DNS name to reach the CommServe, you configure all your clients and MediaAgents to communicate with a dedicated **Commvault Proxy Server**. 
*   The proxy sits in the middle.
*   The clients *only* know the IP/hostname of the proxy.
*   The proxy maintains an active connection tunnel with whichever CommServe node (Primary or Standby) is currently active. 
*   During a failover, the newly active Standby CommServe reaches out to the proxy, updates the route, and the proxy automatically starts forwarding client traffic to the new server. Zero DNS flush required! 🙌

### ⚙️ How to Configure It Step-by-Step

**1. Install the Commvault Proxy**
*   Deploy a dedicated, standalone Windows or Linux machine.
*   Install the Commvault **File System Core** package on it. (It doesn't need a full MediaAgent or CommServe role, just the base package to act as a gateway).
*   Ensure this proxy can reach both the Primary and Standby CommServe hosts, and that all clients can reach the proxy over port 8403.

**2. Create the Network Topology (CommCell Console)**
*   Open the **CommCell Console**.
*   Right-click **Network Topologies** > **New Topology**.
*   Enter a name (e.g., `LiveSync_Proxy_Routing`).
*   For the Topology Type, select **CommServe Port Forwarding** (or proxy-based routing depending on your exact v11 feature release). 

**3. Define the Groups**
The topology wizard will ask you to define three groups:
*   🔽 **CommServe Group:** Put *both* your Primary CommServe and Standby CommServe physical clients into this group.
*   🔽 **Proxy Group:** Put the newly installed Proxy machine into this group.
*   🔽 **Client Group:** Put all your enterprise MediaAgents and endpoint clients into this group.

**4. Push the Network Configuration**
*   Once the topology is created, right-click it and select **Push Network Configuration**.
*   This pushes new firewall routing rules to all machines. 
*   *What this option does:* It updates the local `fwconfig.txt` file on every client, instructing them to stop talking directly to the CommServe and to start sending all CommServe control traffic (like job status and heartbeats) directly to the Proxy.

### 🔥 What Happens During a Failover Now?
If your primary CommServe crashes, you still log into the Standby node's Process Manager and click **Initiate Failover -> Production** just like before. 

But this time, when the standby node comes online, it automatically establishes a network tunnel with the Proxy. Because all your clients are already happily talking to the Proxy, the Proxy seamlessly connects their existing traffic to the newly promoted CommServe. 

No active directory logins, no DNS record updates, no waiting for DNS propagation. The failover is entirely handled at the application layer! 🚀
