# Layer 2 EtherChannel (LACP) Lab
A Cisco Packet Tracer lab demonstrating how to bundle multiple redundant switch links into a single logical **EtherChannel** using **LACP (802.3ad)**, and how this eliminates the Spanning Tree Protocol (STP) blocked ports that normally result from redundant Layer 2 links.

**Table of Contents**
  - Topology
  - Objective
  - Part 1 - Verify the Problem (STP Blocking Redundant Links)
  - Part 2 - Configure EtherChannel (LACP)
  - Part 3 - Verify EtherChannel
  - Key Takeaways

---

**Topology**
Two Cisco Catalyst 2960-24TT switches connected by **four** FastEthernet links (Fa0/1-Fa0/4): 

<img width="1915" height="1030" alt="Layer2 EtherChannel LACP Topology" src="https://github.com/user-attachments/assets/9a1d77aa-94eb-4531-8998-16a13e037a68" />

Before configuration, these four links are seen by STP as four **separate, redundant paths** between the same two switches. 
---

**Objective**
  1. Confirm that four individual parallel links trigger STP to block three of them (loop prevention).
  2. Bundle all four links into a single **Port-channel** using **LACP active mode.**
  3. Confirm the Port-channel is treated by STP as **one logical link,** removing the wasted blocked ports.

---

**Part 1 - Verify the Problem (STP Blocking Redundant Links)** 
**Confirm neighbors and check Layer 2 topology from Switch1:**

<img width="1906" height="1014" alt="Show CDP Neighbor and Show Spanning Tree" src="https://github.com/user-attachments/assets/3e812d0d-1341-44e8-bc99-99c495896aad" />

**Check Spanning Tree State:**

<img width="1915" height="1031" alt="Show Spanning Tree Switch1" src="https://github.com/user-attachments/assets/956e425d-e967-446c-9603-1bd8070efdcf" />

**Observation**
**Interface**      **Role**      **Status**
---
Fa0/1                Root        **FWD** (Forwarding)
---
Fa0/2                Alternate   **BLK** (Blocked)
---
Fa0/3                Alternate   **BLK** (Blocked)
---
Fa0/4                Alternate   **BLK** (Blocked) 
---

Only one of the four links is actually forwarding traffic. STP blocks the other three to prevent a Layer 2 loop - meaning 75% of the available bandwidth between the switches is sitting idle. 

---

**Part 2 - Configure EtherChannel (LACP)**
Bundle all four interfaces into a single Port-channel using LACP, on **both** switches. 

<img width="1904" height="985" alt="Int range " src="https://github.com/user-attachments/assets/a44d3330-0885-415b-a86c-660da443917d" />
Repeat the identical commands on **Switch2.** LACP requires both ends of the bundle to be configured before the links come up as a channel - this is why the switch temporarily reports the ports as suspended. 

**Command Reference**
**Command**                      **Purpose**
---
**int range fa0/1-4**              Select all four interfaces to configure at once
---
**channel-group 1 mode active**    Assign interfaces to Port-channel 1 and enable **LACP active** mode (actively initiates negotiation)
---

**LACP mode notes: active** initiates negotiation; **passive** only responds to negotiation requests. At least one side of a bundle must be **active** - two **passive** ends will never negotiate. 

---

**Part 3 - Verify EtherChannel**

<img width="1915" height="1031" alt="Show Spanning Tree Switch1" src="https://github.com/user-attachments/assets/9eaa4dbf-1788-4531-8aa6-5d961663a317" />

**Before vs. After**
                                        **Before EtherChannel**                               **After EtherChannel**
---
**Interfaces seen by STP**                Fa0/1, Fa0/2, Fa0/3, Fa0/4, (4 separate ports)        Po1 (logical port)
---
**Forwarding links**                      1 of 4 (Fa0/1)                                        All 4, bundled as Po1
---
**Blocked links**                         3 (Fa0/2, Fa0/3, Fa0/4)                               0
---
**STP root port cost**                    19                                                    8
---

STP now sees a single logical interface, **Port-channel1**, in the **Root/Forwarding** state, with a lower path cost (8vs. 19) because the aggregated bandwidth is higher. All four physical links carry traffic simultaneously, load-balanced by the switch, instead of three of them sitting idle in a blocked state. 

---

**Key Takeaways**
  - Without EtherChannel, redundant parallel links between two switches force STP to block all but one, wasting bandwidth.
  - **LACP** (channel-group N mode active**) bundles multiple physical links into one logical **Port-channel** interface.
  - LACP must be configured on **both ends** of the bundle, or the ports remain suspended.
  - STP treats the resulting Port-channel as a single link - no blocked ports, lower cost, and full use of the aggregated bandwidth.
  - This is the standard way to add redundancy and bandwidth between switches without triggering STP to disable the extra capacity. 
