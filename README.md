🛡️ LAN-DMZ-Firewall: Enterprise Network Implementation <br>

🎯 Objective

Designed and deployed a comprehensive network architecture centered around a Fortinet FortiGate firewall, establishing secure routing, segmentation, and controlled access across LAN, DMZ, and WAN (Internet) zones. The primary security policy enforces stateful inspection, permitting outbound traffic from the internal network while strictly allowing return traffic only for established, authorized sessions.<br>

🛠️ Key Implementation Steps
<ul>
<li>Perimeter Security & NAT: Configured WAN connectivity translating private internal IP addresses to public IPs using Port Address Translation (PAT).</li>

<li>Network Segmentation: Implemented VLANs to break up broadcast domains and applied Access Control Lists (ACLs) to enforce security boundaries.</li>

<li>Remote Management & Automation: Enabled SSH for secure administrative CLI access and set up DHCP servers for automated dynamic IP allocation to endpoint devices.</li>

<li>Resilience & Testing: Verified and tested network resilience, route failover, and resource availability across all interfaces.</li> 
</ul>

📖 Skills & Competencies Learned

<ul>
<li>Fortinet Security Fabric: Advanced policy creation, interface configuration, and static routing.

<li>Redundancy Protocols: Configured hot-standby redundancy and gateway load balancing across VLANs.

<li>Dynamic Routing: Implemented OSPF (Open Shortest Path First) to ensure automatic route convergence and failover when links go down.

<li>Network Troubleshooting & Analysis: Utilized Wireshark for packet capture and DHCP troubleshooting, alongside systematic CLI verification commands.

<li>WAN Topologies: Configured and validated single-homed WAN connections.
</ul>

🧰 Tools & Technologies Used
<ul>
<li>Hardware / Platforms: Fortinet FortiGate Firewall, FortiOS CLI.</li>

<li>Protocols & Services: OSPF, DHCP, SSH, PAT/NAT, VLANs, ACLs.</li>

</ul>

## 🗺️ Topology Diagram

[Below is the structural layout of the simulated enterprise edge environment, showcasing dual multihomed wan/isp setup]





## Device Configurations

Layer 3 switch (primary): [Layer 3 switch-1.txt] — Handles primary internet outbound traffic. <br>
Layer 3 switch (secondary):[Layer 3 switch-2]
Internal "Firewall": [Firewall.txt] — Manages NAT/PAT. <br>
Service Provider Gateway A: [ISP-A-Router.txt] — Simulates primary upstream ISP peering. <br>


## ☑️Configuration & Verification Snippets

### 1. Static Policies

[Description]

[Configuration]

```text
! Configuration on edge-router-02

route-map AS_prepend permit 20
 set as-path prepend 64500 64500 64500
!
router bgp 64500
 neighbor 198.51.100.162 route-map route-map AS_prepend out
```
[Optional - Follow Description ] 

```text
inserthostname-here(config)#do sh bgp

     Network          Next Hop            Metric LocPrf Weight Path
 *>   203.0.113.1/32       203.0.113.249                           0 64502 64500 i
 *>   203.0.113.2/32       203.0.113.249                           0 64502 64500 i
```

Now if we shutdown the link to ISP-A (AS-64502), our route to ISP-B (AS-64501) takes over and shows up in the path. We see the path includes AS-64501, the Autonomous system ISP-B belongs too. Here is the output: 

```diff
inserthostname-here(config)#do sh bgp

     Network          Next Hop            Metric LocPrf Weight Path
+ *>   203.0.113.1/32       198.51.100.165                           0 64501 64500 64500 64500 64500 i
+ *>   203.0.113.2/32       198.51.100.165                           0 64501 64500 64500 64500 64500 i
```

### 2. Routing

[Description]

[Config]


### 1. NAT Translation Verification
Verify that the Gateway Router is actively translating private IP traffic to public-ready flows, since private IP's can't be routed over the internet:
```text
Gateway Router(config)#do sh ip nat translations

Pro Inside global      Inside local       Outside local      Outside global
icmp 192.0.2.3:20032  192.168.10.1:20032 203.0.113.8:20032      203.0.113.8:20032

```
### 3. BGP Path Attribute Verification

[Description]
[Configuration]
