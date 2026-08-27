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

[Below is the structural layout of the simulated enterprise edge environment, showcasing a Fortinet Firewall connecting LAN > DMZ > Internet]

<img width="1857" height="816" alt="topology" src="https://github.com/user-attachments/assets/4ce74319-2c33-44ce-aea1-2133ca7bf72b" />

## Device Configurations

Layer 3 switch (primary): [Layer 3 switch-1.txt] — Handles primary internet outbound traffic. <br>
Layer 3 switch (secondary):[Layer 3 switch-2]
Internal "Firewall": [Firewall.txt] — Manages NAT/PAT. <br>
Service Provider Gateway A: [ISP-A-Router.txt] — Simulates primary upstream ISP peering. <br>

## ☑️Configuration & Verification Snippets

### 1. Static Policies

[Description]

Policies were created on the Fortinet Firewall to allow connections to the Internet, only from IP's that originated from VLAN10 and VLAN20. NAT was enabled in order for the devices in the LAN to use the outbound interface pointing towards the internet. I also created a connection from LAN > DMZ to allow access to a web server, ftp server, and DHCP server.

[Configuration]

<img width="1920" height="903" alt="Fortigate Policy" src="https://github.com/user-attachments/assets/d4e0c7a1-f729-4c10-aaf1-9277e3dc76b9" />

[Optional - Follow Description ] 

I created a FTP server running on Linux in the DMZ. Here is the policy I created for Core/Dist-SW1 to connect and download a file from the FTP server. 

<img width="1912" height="786" alt="FTP_Server" src="https://github.com/user-attachments/assets/34154cc0-aeb2-41fd-9b14-011dafbd9145" />


```text

```


```diff
```

### 2. Routing

[Description]

[Config]



### 3.  Verification

[Description]
[Configuration]

### 4. Implemented a Web filter
I imposed a web filter on the LAN > WAN Firewall policy to block certain websites
<img width="1005" height="752" alt="Block_Websites" src="https://github.com/user-attachments/assets/adf7e542-e94b-456d-a6f4-5e0cf1c1bf5a" />

