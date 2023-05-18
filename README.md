# Deploying Distributed Fabrics with Aruba Central NetConductor (ACN)
## Introduction
:wave: **Welcome to the Deploying Distributed Fabrics TSS lab.**
* In this lab you will use Aruba Central NetConductor to build and configure your own network fabric running EVPN-VXLAN for reachability and micro-segmentation. Then put the configuration to the test with the new Central 'Global Roles' functionality.
* You have access to dedicated hardware running in our 'workbench' facility, and your own, tailor-made, Aruba Central account.
* Plus there is a Windows Active Directory Domain and ClearPass up and running to enable you to test user 802.1X AuthN and role assignment.
* Don't worry if this is your first time with Aruba Central or EVPN-VXLAN - this guide will step you through the process.

### Lab Setup
Ok so what toys do you have to play with then?
1. A dedicated spine and leaf network comprised of:
    * 2 x 8325
    * 2 x 6300
2. An Aruba Central Account
3. 2 x Wins 10 clients - to emulate user login/802.1X
   
All of the above is dedicated to your own lab 'pod'.
There is also a shared ClearPass instance and a Windows 2019 server running Active Directory. These have already been configured for you and are shared amongst the whole lab so:

**Please do not reconfigure the CPPM or Windows 2019 server - everyone needs to share those resources.**

### Lab Diagram


### A word about scope of the lab and what to expect
#### Scope
* Aruba Central NetConductor is a platform which unifies Aruba networks for wired, wireless across campus LAN, Data Centers and WAN networks.
* In this lab we will solely focus on the large-to-medium wired design architecture, namely the 'Distributed' approach, so named because the enforcement of policy is distributed across the network, rather than in a 'Centralized' point.
* Moreover, Aruba Central Netconductor is designed to off-load much of the configuration overhead and operational complexity of running an EVPN-VXLAN network.
* Normally ACN handles all the config management, so you don't have to.
* But then, this is a technical event, and you're here for more than just ticking a few boxes in a UI.
* As such, we are going to deeper into the network configuration, to really delve into the device setup and EVPN tables.
* Now, this is advanced networking, so don't worry if some things do not click in this short lab. An EVPN table is something of a beast when first experienced.
* We do hope you enjoy the lab, and if you need any assistance, just ask!