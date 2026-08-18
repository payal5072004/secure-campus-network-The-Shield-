# secure-campus-network-The-Shield-
Secure Campus Network using Dynamic VLAN and RADIUS Authentication

        SHIELD PROJECT REPORT
       

Authors   : Payal, Mudit
Department: Information Technology
College   : Netaji Subhash University of Technology
            Dwarka, New Delhi
Date      : April 2026
==========================================================================================================================


1. PROJECT OVERVIEW
====================
SHIELD is a seven-layer secure campus network architecture
designed, implemented, and validated using GNS3 (Graphical
Network Simulator 3). The project addresses critical security
gaps in traditional campus networks by integrating:

- Identity-driven dynamic VLAN assignment
- Centralized AAA via FreeRADIUS
- IEEE 802.1X port-based access control
- ACL-based inter-VLAN traffic filtering
- Zero Trust time-based access control
- NTP-synchronized temporal enforcement
- BlastRADIUS (CVE-2024-3596) mitigation

The architecture was simulated using GNS3 with real Cisco IOS
images (c7200 for router, c2691 for switches) and a FreeRADIUS
server running on Ubuntu 24.04 LTS in VMware Workstation.


2. NETWORK TOPOLOGY
================================================================

2.1 Devices
-----------
Component              | Details
-----------------------|--------------------------------
Core Router            | R1 (Cisco c7200)
Core L3 Switch         | ESW1 (multilayer)
Distribution Switches  | switch2,switch3,switch4,switch5
Access Switches        | 15 (switch6 to switch21)
End Devices            | 44 VPCS nodes
AAA Server             | FreeRADIUS v3.0
Server OS              | Ubuntu 24.04 LTS (VMware)
FreeRADIUS IP          | 192.168.56.129
Simulation Tool        | GNS3 + VMware Workstation

2.2 Three-Layer Hierarchy
--------------------------
Core Layer       : R1 + ESW1
                   (routing, ACL, DHCP, NTP enforcement)

Distribution     : switch2,switch3,switch4,switch5
Layer              (zone aggregation, trunk links)

Access Layer     : 15 access switches
                   (end device connection, 802.1X)

2.3 VLAN Zone Mapping
----------------------
VLAN | Zone           | Subnet           | Dist. Switch
-----|----------------|------------------|-------------
10   | Academic       | 192.168.10.0/24  | switch 2
20   | Administrative | 192.168.20.0/24  | switch 3
30   | Residential    | 192.168.30.0/24  | switch 4
40   | Service        | 192.168.40.0/24  | switch 5


3. SEVEN SECURITY LAYERS
================================================================

LAYER 1 - VLAN SEGMENTATION
-----------------------------
- 4 VLANs configured on all 20 switches
- Command used : vlan database
- Verified via : show vlan-switch brief
- Trunking     : IEEE 802.1Q on all uplinks
- Command used : switchport mode trunk
- Result       : 75% broadcast reduction

LAYER 2 - FREERADIUS CENTRALIZED AAA
--------------------------------------
- FreeRADIUS v3.0 installed on Ubuntu 24.04
- Install cmd  : sudo apt install freeradius
- Users file   : /etc/freeradius/3.0/users
- Clients file : /etc/freeradius/3.0/clients.conf
- Auth tested  : radtest command
- Result       : Access-Accept / Access-Reject working
- Auth rate    : 85.7% success rate
- Latency      : less than 5ms average

LAYER 3 - DYNAMIC VLAN ASSIGNMENT
------------------------------------
- Auto VLAN placement on authentication
- VSA Attributes used:
  Tunnel-Type = VLAN
  Tunnel-Medium-Type = IEEE-802
  Tunnel-Private-Group-ID = VLAN number
- student1    -> VLAN 10
- faculty1    -> VLAN 10
- admin1      -> VLAN 20
- tnp1        -> VLAN 20
- residential1-> VLAN 30
- guest1      -> VLAN 40
- Identity follows user, not port

LAYER 4 - ACL BLOCKING
------------------------
- ACL Name : BLOCK_UNAUTHORIZED
- Applied on: ESW1 (core L3 switch)
- Blocks   : VLAN 30/40 -> VLAN 10/20
- Verified : 75 packets blocked during simulation
- Command  : ip access-list extended BLOCK_UNAUTHORIZED

LAYER 5 - IEEE 802.1X PORT SECURITY
--------------------------------------
- Deployed on all 15 access switches
- Command  : dot1x port-control auto
- Coverage : 37/37 ports = 100%
- Verified : show dot1x
- Status   : CONNECTING state on all ports
- Sysauthcontrol = Enabled on all switches

LAYER 6 - ZERO TRUST TIME-BASED ACL
--------------------------------------
- ACL Name : ZERO_TRUST
- Applied on: ESW1
- NTP Source: FreeRADIUS server (192.168.56.129)
- NTP Status: Stratum 4, synchronized at IST 10:53
- Time Ranges:

  STUDENT_HOURS  -> VLAN 10 -> 08:00 to 22:00
  ADMIN_HOURS    -> VLAN 20 -> 08:00 to 20:00
  RESIDENTIAL    -> VLAN 30 -> 06:00 to 23:00
  SERVICE        -> VLAN 40 -> 00:00 to 23:59

- Access denied outside permitted hours
- Even authenticated users blocked after hours

LAYER 7 - BLASTRADIUS MITIGATION
----------------------------------
- CVE ID   : CVE-2024-3596
- Attack   : MD5 MITM on RADIUS UDP
- Fix file : /etc/freeradius/3.0/radiusd.conf
- Config   : require_message_authenticator = yes
- Result   : All forged 0x00 packets REJECTED
- Verified : FreeRADIUS debug logs
- Legit auth: Still working at 85.7% success rate


4. KEY RESULTS SUMMARY
================================================================
Metric                    | Result
--------------------------|----------------------
VLANs configured          | 4
Total switches            | 20
Access switches           | 15
End devices (VPCS)        | 44
802.1X port coverage      | 37/37 = 100%
Packets blocked by ACL    | 75 confirmed
Auth latency (avg)        | 2.1ms
Auth latency (max@50users)| 4.9ms
Broadcast reduction       | 75%
NTP Stratum               | 4 (synchronized)
BlastRADIUS               | Detected + Mitigated
Max scalable devices      | 15,240


5. ATTACK SIMULATION RESULTS
================================================================
Attack Type               | Result    | Layer Used
--------------------------|-----------|------------------
Cross-VLAN Intrusion      | BLOCKED   | Layer 4 (ACL)
Brute Force Attack        | REJECTED  | Layer 2 (RADIUS)
Unknown User Access       | REJECTED  | Layer 2 (RADIUS)
Zero Trust Time Bypass    | DENIED    | Layer 6 (ZT ACL)
BlastRADIUS CVE-2024-3596 | MITIGATED | Layer 7


6. RADIUS AUTHENTICATION TEST RESULTS
================================================================
User          | Password      | Response      | VLAN
--------------|---------------|---------------|------
student1      | Student@123   | Access-Accept | 10
faculty1      | Faculty@123   | Access-Accept | 20
admin1        | Admin@123     | Access-Accept | 20
tnp1          | Tnp@123       | Access-Accept | 20
residential1  | Resident@123  | Access-Accept | 30
guest1        | Guest@123     | Access-Accept | 40
wronguser     | wrongpass     | Access-Reject | --


7. IP ADDRESSING TABLE
================================================================
PC Node | Zone        | VLAN | Assigned IP        | Gateway
--------|-------------|------|--------------------|---------------
PC1     | Academic    | 10   | 192.168.10.1/24    | 192.168.10.1
PC5     | Academic    | 10   | 192.168.10.5/24    | 192.168.10.1
PC6     | Admin       | 20   | 192.168.20.6/24    | 192.168.20.1
PC10    | Admin       | 20   | 192.168.20.10/24   | 192.168.20.1
PC36    | Residential | 30   | 192.168.30.36/24   | 192.168.30.1
PC18    | Residential | 30   | 192.168.30.18/24   | 192.168.30.1
PC11    | Service     | 40   | 192.168.40.11/24   | 192.168.40.1
PC14    | Service     | 40   | 192.168.40.14/24   | 192.168.40.1
PC17    | Service     | 40   | 192.168.40.17/24   | 192.168.40.1


8. RESEARCH CONTRIBUTIONS
================================================================
1. First paper with all 7 security layers in single GNS3
   campus simulation

2. Novel Zero Trust time-based ACL with NTP-synchronized
   FreeRADIUS as time source

3. First campus network paper to demonstrate BlastRADIUS
   (CVE-2024-3596) detection AND mitigation in GNS3

4. 100% IEEE 802.1X port coverage across 15 access switches
   in large-scale simulation


9. SECTIONS IN THIS SUBMISSION
================================================================
01_Project_Report.txt      
02_R1_Config.txt            
03_ESW1_Config.txt         
04_FreeRADIUS_Config.txt 
05_Access_Switches_Config.txt 
06_VPCS_Config.txt          
07_Screenshots/             
Research_Paper.pdf        




