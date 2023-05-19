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

## Lab
### Scenario
Congratulations - your hardwork, dedication and general grindset mentality has earned you the prized 'employee of the month' award at TSSlab Corp. 
In recognition of your success, you've been asked to head up the all-important new campus network build using Aruba's new Enterprise Architecture platform, NetConductor (we'll call it ACN from now on.)
Your task is to deploy a new network fabric on a AOS-CX switching spine-and-leaf underlay.
The aim of the network build is to support both the Production and Development networks, keeping them separate across the shared fabric core.
In addition, TSSlab Corp has had some outages recently. Cause: The ACLs across the network have never been tidied up and even small changes are creating big issues. The C-suite have noticed and are asking questions (like 'can we replace the networking team with chatGPT?')
You and your network team are going solve this headache and have decided to use this greenfield site as your chance to improve things, to move beyond the ageing ACL / VLAN security model, and deploy dynamic role-based segmentation!
The new build is the perfect opportunity because, while the site is up for Employees, who must communicate, there are a number of contractors on-site, who should be kept apart from the Employee data. Rather than carve up the network in new VLANs for the temporary contractors, you plan to use a Contractor role and deny traffic to your Employee users, all enabled dynamically at user log in.

To get sign-off on this duty, you need to complete the following:
* Build the EVPN-VXLAN fabric.
* Deploy two VRFs - 'Prod' and 'Dev'
* Configure and Deploy the Employee and Contractor Global Roles in Central.
* Configure at least one virtual network across site.
* Prove Employee to Employee East-West traffic flows are permitted, while Contractor-Employee flows are denied. *depending on whether your friendly neighbourhood lab admin (me) can get the WinsAD/CPPM/cert to play nicely! 😅

### Lab Task 1 - Get logged in
1. There are five virtual network pods. Ask your admin for your pod number and login credentials.
2. For each pod there is:
    * An assigned workbench platform - Navigate to http://tssemea.arubademo.net - use the external_user username and password to login *(get this from Joe)* Note that the workbench ID does not match the pod number - because *reasons* 🤷‍♂️
    * A dedicated Central user account - Navigater to https://common.cloud.hpe.com/ - use the dedicated workbench user and password *(ask Joe for deets)*
3. Once you have gained access - have a look around but do not make any change just yet! Please do ensure that your workbench username and Central account matches the one assigned.

* N.B. With the two platform you have the option to configure the devices either way, directly on the device or through Central - to emulate a Central-focused operational workflow, you should try to use Central as much as possible to configure the devices and this will be the primary approach.
* However, for show commands, we will console in AOS-CX devices directly using the workbench console functionality. 
* Ad-hoc config changes can be performed using Multi-edit.



### Lab Task 2 - Prepare the AOS-CX devices for Fabric Configuration
1. View the default group for your Central account. Right after log nativate to **Applications -> My Applications - Aruba Central -> Deployment Regions - Aruba Central US West - Launch**
2. You should now be on the Aruba Central Front Page. Under the **Manage** banner on the left side, click on **Devices**, then click on **Switches** to show your AOS-CX ready to be deployed as a fabric.
3. Check to see that the switches all have a **Config Status** of **'In sync'**. If not, ask your admin to check them.


![Global](/images/task2-1-global.png)
