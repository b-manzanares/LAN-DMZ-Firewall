# LAN-DMZ-Firewall

##Objective - The goal was to implement a firewall policy, static routing, and interface setup for a Fortinet firewall. This provided connectivity to a LAN, DMZ, and internet. Allowing connectivity from LAN to internet, and return only if a connection was established.

* Set up a Fortinet firewall to connect to internet translating IP's from Private to Public via Port Address Translation (PAT).
* Incorporated VLAN's for segmentation, secure shell for secure remote access, and Access-lists for security.   
* Created DHCP for devices to receive IP addresses automatically.
* Verified and tested network resilience and resource availability.

## 📖 Skills Learned

* Fortinet polices & routing.
* Hot standy redundancy protocol load balancing between VLANs.
* Use of Wireshark for DCHP troubleshooting.
* OSPF routing, for route updates when a link goes down. 
* Single-home WAN setup.
* Network troubleshooting.

## 🛠️ Tools & Technologies Used

* Verification commands
* ACL's
* Fortinet CLI for basic setup

## 🗺️ Topology Diagram

[Below is the structural layout of the simulated enterprise edge environment, showcasing dual multihomed wan/isp setup]





## Device Configurations

Layer 3 switch (primary): [Layer 3 switch-1.txt] — Handles primary internet outbound traffic. <br>
Layer 3 switch (secondary):[Layer 3 switch-2]
Internal "Firewall": [Firewall.txt] — Manages NAT/PAT. <br>
Service Provider Gateway A: [ISP-A-Router.txt] — Simulates primary upstream ISP peering. <br>


## ☑️Configuration & Verification Snippets

### 1. Static Policies

To prevent ISP-B from being used for inbound corporate traffic during normal operations, the secondary edge router prepends its Autonomous System (AS) number three times to advertisements sent to ISP-B. We might wish to alter how traffic arrives from the internet, since an ISP could charge more money or a stateful firewall could block packets on the return trip, because any connection that wasn't previously established is assumed to be malicious. 

```text
! Configuration on edge-router-02

route-map AS_prepend permit 20
 set as-path prepend 64500 64500 64500
!
router bgp 64500
 neighbor 198.51.100.162 route-map route-map AS_prepend out
```

We use out at the end of the route-map because we want routers on the ISP side to believe that it is receiving a longer path to AS-64500 (Edge-router 1 or 2), from edge-router-02. This influences what route is picked as the best path in BGP, since the router will chose a lower AS path length.  If we check the output we see AS-64502 (ISP-A) listed as part of the path to network 203.0.113.1 & 203.0.113.2, our edge router 1 and 2 loopback interface ip addresses.  

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

Running show ip bgp summary confirms that peerings are properly established (State/PfxRcd shows a numerical value of prefixes received rather than an active state like Active or Idle).

```text
inserthostname-here(config)#do sh ip bgp sum

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
198.51.100.165    4       64501      49      49       17    0    0 00:39:14        8
203.0.113.249    4       64502      51      49       17    0    0 00:39:10        13
```
### 3. BFD Session Verification

Verify sub-second path tracking state across the shared Layer 2 switch network. We see here that packets are being sent at 300 ms with a multiplier 3, allowing for faster convergence and almost instant fail-over. Sometimes this is needed in the case of a switch connecting two routers. Unless the routers are directly connected, they will not know that others links went down until OSPF or BGP dead timers hit 0, which can be a long time by default. 

```diff

NeighAddr                              LD/RD         RH/RS     State     Int
203.0.113.254                             1/1          Up        Up        Gi0/1
+Session state is UP and using echo function with 300 ms interval.
Session Host: Software
OurAddr: 203.0.113.253
Handle: 1
Local Diag: 0, Demand mode: 0, Poll bit: 0
+MinTxInt: 1000000, MinRxInt: 1000000, Multiplier: 3

``` 

### 1. NAT Translation Verification
Verify that the Gateway Router is actively translating private IP traffic to public-ready flows, since private IP's can't be routed over the internet:
```text
Gateway Router(config)#do sh ip nat translations

Pro Inside global      Inside local       Outside local      Outside global
icmp 192.0.2.3:20032  192.168.10.1:20032 203.0.113.8:20032      203.0.113.8:20032

```
### 3. BGP Path Attribute Verification
Verify that the Local Preference and prefix matching are successfully manipulating paths:

```diff
edge_router_1(config)#do sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 *>   203.0.113.1/32       0.0.0.0                  0         32768 i
 r>i  203.0.113.2/32       203.0.113.2                  0    100      0 i
+*>   203.0.113.3/32       203.0.113.254              0    130      0 64502 i
+*>   203.0.113.4/32       203.0.113.254              0    130      0 64502 i
+*>   203.0.113.5/32       203.0.113.254              0    130      0 64502 i
+*>   203.0.113.6/32       203.0.113.254              0    130      0 64502 i
+*>   203.0.113.7/32       203.0.113.254              0    130      0 64502 i
+*>   203.0.113.8/32       203.0.113.254              0    130      0 64502 i
+*>   203.0.113.9/32       203.0.113.254              0    130      0 64502 i
 * i  10.10.10.8/30    203.0.113.2                  0    100      0 i
 *>                    0.0.0.0                  0         32768 i
 *>i  203.0.113.11/32   203.0.113.2                  0    100      0 64501 i
 *>   192.0.2.35/32    203.0.113.254                  130      0 64502 650003 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>   198.51.100.164/30  203.0.113.254                   130      0 64502 64503 i
 *>   198.51.100.160/30  203.0.113.254                   130      0 64502 64503 i
 * i  192.0.2.0/29    203.0.113.2                  0    100      0 i
 *>                    0.0.0.0                  0         32768 i
 r>   203.0.113.252/30   203.0.113.254              0    130      0 64502 i
```
BGP decides what route is the best path by a checking a hierarchy of attributes. If a specific attribute breaks the tie than that route is selected. In this case the Local preference was changed from the default 100 to 130 forcing the router to prefer ISP-A, and this happens for edge-router-2 since they lie within the same Autonomous system, unlike weight which is local to the router. 

