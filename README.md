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
 **Congratulations 🥳** - your hardwork, dedication and general grindset mentality has earned you the prized *'employee of the month'* award at TSSlab Corp (like Weyland-Utani Corporation but with better gym membership perks). 

* In recognition of your success, you've been asked to head up the all-important new campus network build using Aruba's new Enterprise Architecture platform, NetConductor (we'll call it ACN from now on.)
* Your task is to deploy a new network fabric on a AOS-CX switching spine-and-leaf underlay.
* The aim of the network build is to support both the Production and Development networks, keeping them separate across the shared fabric core.
* In addition, TSSlab Corp has had some outages recently. Cause: The ACLs across the network have never been tidied up and even small changes are creating big issues. The C-suite have noticed and are asking questions (like 'can we replace the networking team with chatGPT?')
* You and your network team are going solve this headache and have decided to use this greenfield site as your chance to improve things, to move beyond the ageing ACL / VLAN security model, and deploy dynamic role-based segmentation!
* The new build is the perfect opportunity because, while the site is up for Employees, who must communicate, there are a number of contractors on-site, who should be kept apart from the Employee data. Rather than carve up the network in new VLANs for the temporary contractors, you plan to use a Contractor role and deny traffic to your Employee users, all enabled dynamically at user log in.

To get sign-off on this duty, you need to complete the following:
    
    * Build the EVPN-VXLAN fabric.
    * Deploy two VRFs - 'Prod' and 'Dev'
    * Configure and Deploy the Employee and Contractor Global Roles in Central.
    * Configure at least one virtual network across site.
    * Prove Employee to Employee East-West traffic flows are permitted, while Contractor-Employee flows are denied. (depending on whether your friendly neighbourhood lab admin (me) can get the WinsAD/CPPM/cert to play nicely!)

### Lab Task 1 - Get logged in
1. There are five virtual network pods. Ask your admin for your pod number and login credentials.
2. For each pod there is:
    * An assigned workbench platform - Navigate to http://tssemea.arubademo.net - use the external_user username and password to login *(get this from Joe)* Note that the workbench ID does not match the pod number - because *reasons* 🤷‍♂️
    * A dedicated Central user account - Navigater to https://common.cloud.hpe.com/ - use the dedicated workbench user and password *(ask Joe for deets)*
3. Once you have gained access - have a look around but do not make any change just yet! Please do ensure that your workbench username and Central account matches the one assigned.

* N.B. With the two platform you have the option to configure the devices either way, directly on the device or through Central - to emulate a Central-focused operational workflow, you should try to use Central as much as possible to configure the devices and this will be the primary approach.
* However, for show commands, we will console in AOS-CX devices directly using the workbench console functionality. 
* Ad-hoc config changes can be performed using Multi-edit.



### Lab Task 2 - Verify the AOS-CX devices
* *First we need to ensure the devices are reset and ready in the 'Global' group, then move them into a new UI group for tge Fabric config push.*
* *The devices have already been configured to instantiate the necessary underlay configuration.*
* *The underlay consists of point-to-point L3 links, loopbacks and OSPF.*
* *The role of the underlay IGP is essentially just a transport for the EVPN-VXLAN device loopbacks. These loopbacks are the source and destination addresses for the VXLAN encapsulation and decapsulation AKA the overlay tunnel, which carry the customer traffic.*
1. View the default group for your Central account. Right after log nativate to **Applications -> My Applications - Aruba Central -> Deployment Regions - Aruba Central US West - Launch**
2. You should now be on the Aruba Central Front Page. Under the **Manage** banner on the left side, click on **Devices**, then click on **Switches** to show your AOS-CX ready to be deployed as a fabric.
3. Check to see that the switches all have a **Config Status** of **'In sync'**. If not, ask your admin to check them.


![Global](/images/task2-1-global.png)

### Lab Task 3 - Create and Populate the Fabric UI group

1. Create the new UI group. Go ***Global - Maintain - Organization -> Groups -> Add Group*** (click the plus sign top right).
    ![step-one](/images/task2-2-1.png)
    ![step-one](/images/task2-2-2.png)


2. Give the group a name - something like 'tss-lab3' - '3' being the pod number, and check **Switches**.


    ![step-two](/images/task2-2-3.png)


3. Set the type of group to AOS-CX only, click **Add**.


    ![step-three](/images/task2-2-4.png)


4. Now move all four devices to the newly created UI group - expand the default group by clicking the arrow on the left -> select each device by clicking on them -> Click the **Move Devices** icon.


    ![step-four](/images/task2-2-5.png)


5. Select the destination group and ensure the 'Retain CX-Swtich configuration' box is checked, then click **Move**!


    ![step-five](/images/task2-2-6-1.png)


6. Head to your new UI group view, you should see that your devices have a **Config Status** of 'Not in sync'. That is to be expected after the device group move.


   ![step-six](/images/not-in-sync.png)

7. Once the device move has completed, you should see a pop-up banner prompting you to set an Administrator password. Please set this to **Aruba123!**, we will need it to log into the device consoles directly later in the lab. (If you do set the password to something different, please make a note of it!)

### Lab Task 4 - Configure the Global Roles
*In this task we configure the Global Client Roles to be pushed to our fabric, these are the basis of our security policy.* 

1. Navigate back to **Global -> Manage - Security -> Client Roles.
2. Ensure that the **'Role-to-Role Policy Enforcement'** is set to the right.
3. Also, check that the **'Use a switch fabric for role propagation?** radio button is set to **Yes**. We are creating the global Client Roles here, they will then be pushed to the devices when they are configured as part of a fabric later on.
4. Click **Save** then hit the plus sign, top right to create a new role!

   ![task-4-setup](/images/task4-1.png)

5. You will see the 'Create a new role' pane.
6. Create a role with the name 'Employee', give it a short description. The Policy Identifier is pre-populated with a suitable number, *100 by default*.
7. Check the **Allow default role to source role...** box.
8. Hit the pen icon in the Permission box on the right.

      ![task-4-2-setup](/images/task4-2.png)

    > **It is important to ensure you type the role names as shown. The role name string must match the string sent in the RADIUS VSA from ClearPass. In this lab, we use the role names 'Employee' and 'Contractor'.**

9. In the Assign Permissions pane, the permissions are built from the point of view of the role in question, this is the *source role*. The table is built to allow you to build combinations of permit and deny between the source role, and any active *Destination roles*, which are listed in the left-most column.
10. We only have one role at present, and we want Employees to speak to Employees, so check both boxes and hit **Assign**.


     ![task-4-3](/images/Task4-3.png)


11. Repeat the process above for the Role 'Contractor'. But this time, DO NOT tick the **Allow default role to source role...** box.
12. With in the Assign Permissions pane, we only want the Contractors to be able to communicate with other Contractors, and not our Employees. Remember that the permissions are from the point of view of the role being configured, so the Permissions should look like this:

     ![task-4-4](/images/Task4-4.png)

That's for the roles build, now on to the Fabric configuration!

### Lab Task 5 - Configure the Fabric
* *In this task we will configure the device for the EVPN-VXLAN fabric. This will setup the VRF and BGP peering sessions.*
* *To simplify the configuration tasks, ACN groups the job of different devices required on the fabric into **'Personas'**. Rather than juggling the complex configurations per device type, you simply assign the **'Persona'** to the device.*
* *In addition, because we are propagating the Client Roles via the fabric (Task 4.3 above), our role configuration will be pushed to the devices.*
* *Note that the overlay and role actions will not be active as of yet though, that comes in the next task, when we configure the actual overlay networks and tenet subnets. 'Tenet' the term used in EVPN RFCs to describe the customer side networks I.E. not our fabric.*
1. Navigate to the UI fabric group that you created, go **Manage - Devices -> Config**.
2. You should see the ability to configure **Fabrics** in the **Routing** box. If not, check that the **MultiEdit** switch is grey out / off. 

    ![task-5-1](/images/task5-1.png)

3. Click the plus sign, top right, to add a new fabric.
4. You're now in the **Create a New Fabric** workflow. There's only five panes in total, and we are going to skip one! Start by adding a fabric name, this can be a string of your choice.
5. The BGP AS Number for the fabric is pre-populated with 65001. That works for our build, hit **Next**.

> **The EVPN-VXLAN fabric we are building uses Internal BGP, the AS number provided is one of the assigned Private Autonomous System numbers. This is something akin to RFC 1918 private addressing for internal networks I.E. should never be seen on the public internet.**

6. Now we are in the **Add Devices** pane. Here you should see all four of your devices. (Let your admin know if not). This is where you assign the Personas. We have two groups of devices, akin to spine group and a leaf group. You can tell which is which by the device **Names**.
    
    * The 6300s are the Edge devices.
    * The 8325s are the **RR** (Route-reflector) and the **Border** personas.
    * We do not have any AOS gateways in this lab, so no device has the **Stub** persona.

7. Against each device check the correct persona to assign it. You'll see a helpful diagram appears, graphically explaining each persona.

![personas](/images/personas.png)

> #### **Personas**
>   * Edge - The devices that sits at the boundary of the EVPN-VXLAN fabric. It is a VTEP - encapsulating and decapsulating VXLAN traffic. Our client roles are also enforced on its tenet-facing port (the port access config goes on those ports).
>   * RR - The EVPN-VXLAN fabric is Internal BGP. BGP uses AS_PATH to prevent routing loops, but IBGP cannot rely on this because everything is happening in a single AS. Thus, IBGP uses a full-mesh of sessions to prevent loops. But, next problem, this doesn't scale well. Route-reflectors are used to allow greater scale, IBGP devices per to the RRs and the RR handling the relevant forwarding. RRs are the spines in our network!
>  * Stub - Stub devices form the fabric connection between the EVPN-VXLAN fabric and devices that do not speak EVPN, namely AOS 10 gateways. Stubs extend the fabric segmentation functionality over static VXLAN tunnels, forming the bridge between wired and wireless Aruba fabrics.
>   * Border - Border devices sit at the EVPN-VXLAN fabric edge, handing off to device external to the fabric.

8. Next up is **Add Overlay Network**. The Overlay Networks here are the VRFs on the network. The VNI number is VXLAN ID number to identify the VRF traffic as it traverses the VXLAN fabric. Name the overlay network **Prod** and add in another, called **Dev**. These are our VRFs. Enter **10020** for the **Dev** VNI. Click **Next**
   
   >Note VRFs are only ever local to the device on which they are configured, once the traffic leaves the device (in the data plane) we need some form of ID to ensure the traffic from different VRFs stays separate on the wire. With VXLAN, the VNI is used.


![vnis](/images/vnis.png)

9. We do not have any **Stub** devices, so you can skip that.
10. That's it! Now you're at the **Summary**. Click **Save**.
11. You will return to the main fabric build pane. If you click **List** (top right), you will see that the devices are **Not in Sync** as ACN pushes the configuration to the device.
12. Goto **Analyze -> Audit Trail** to see the audit of messages as the config is pushed to devices.
13. It may take a few minutes for the config push to complete. Once it has, and the devices are in sync, it is time to move on to building the overlays proper!

### Lab Task 6 - Configure the Overlays
* *The fabric configured so far is more the core residing portion of the overlay, now with we configure the virtual networks that map to the customer-side VLANs. In VXLAN terms, these are the L2VNI; ACN calls them 'segments'.*
* *Against these segments we configure the customer VLAN, the active default gateway and any DHCP configuration.*
* *It is on these segments that we apply the clients roles that we wish to be active on them.*
* *Yes, segments are important!*

1. With all the devices *In Sync*, go back to the Fabric configuration page **Manage - Devices -> Routing - Fabrics -> Hover over your fabric and the small 'Create Segment' icon will appear.** It looks like a couple of computers on a LAN.
2. Click the **Create Segment** and you will see the **New Segment** pane. Start by clicking the arrow next to 'Overlay Network'. You will see your overlay networks. Select one to build the segment against.

    ![segment](/images/task6-1.png)

3. Now you need to enter the VLAN Name and VLAN ID, note that this is the tenet(customer) side VLAN. By creating a segment you are mapping the customer-side VLAN to a fabric-side VXLAN Virtual network (a VNI). The enter a name of your choosing, the VLAN ID doesn't really matter but 10 is a good one!
4. The next line is the default gateway configuration. This is the configuration of the L3 VLAN interface that sits on the Edge device. 

> **EVPN utilises an anycast default gateway model, so the same virtual MAC and IP address can be configured on all Edge devices. This enables VM migration, their MAC & ARP tables entries for the default gateway are valid if they move between gateways, no need to ARP again!**

5. Here's the IP addressing that you should use for your gateway.

    | Pod        | Default Gateway IP|
    | ----------|-------------:|
    |    1      | 172.21.10.254 | 
    |    2      | 172.22.10.254 |
    |    3      | 172.23.10.254 | 
    |    4      | 172.24.10.254 |
    |    5      | 172.25.10.254 |


    ![segment-config](/images/task6-2.png)


6. The section at the bottom of the page is for DHCP Relay, this enables the customer-side devices to pick up IP addresses from a centralized DHCP server. This is particularly important within EVPN-VXLAN networks, because the DHCP server has to return the OFFER linked to the correct VRF. For this lab, at the time of writing, the admin hasn't configured this, so we just skip it, the connect customer-side Windows clients are statically configured. 

7. Next we add our Client Roles to the Segment. Check both role boxes then hit 'Next'.
   

    ![roles-config](/images/task6-3.png)


>**In ACN terminology, an EVPN-VXLAN fabric is called 'Distributed'. That is because the role enforcement happens on the traffic's egress from the Edge device. This is a *Distributed* enforcement model - rather than Centralized.**

8. Now, for the final piece of configuration, we apply the 'segment' to the Edge devices. Check the box for both 6300s and hit 'Next'. Take a moment to admire the summary of your toil....then smash that 'Save' to kick off the configuration process.

    ![edge-config](/images/task6-4.png)

9. Same rules apply, you'll see the 6300 devices go 'Not in sync', you can view the config push in the **Analyze - Audit Trail**


### Lab Task 7 - Verify the configuration
*We are now going to log into the console of the devices & clients via workbench to run a number of verification commands.*

1. Log into workbench at http://tssemea.arubademo.net/ (your admin has your credentials)
2. Go to **View Topology**
   
    ![view-topology](/images/task7-1.png)


3. You will now see your topology, it is essentially a spine-and-leaf, with a couple of conneced clients, and supporting infra.


    ![topology](/images/task7-2.png)


 4. Right-click on one of the 6300 (go for 6300A - this is your 6300-x-1) and click **Console Access**


    ![console-access](/images/task7-3.png)


5. Log into the 6300 using the admin password you set earlier (that's why it was important to remember - default '*admin/Aruba123!*')

6. Here's a few commands to verify the configuration:

```
6300-3-1# sh bgp l2vpn evpn summary 
VRF : default
BGP Summary
-----------
 Local AS               : 65001        BGP Router Identifier  : 192.168.0.3    
 Peers                  : 2            Log Neighbor Changes   : No             
 Cfg. Hold Time         : 180          Cfg. Keep Alive        : 60             
 Confederation Id       : 0              

 Neighbor        Remote-AS MsgRcvd MsgSent   Up/Down Time State        AdminStatus
 192.168.0.1     65001       1176    1169    16h:51m:37s  Established   Up         
 192.168.0.2     65001       1186    1173    16h:51m:37s  Established   Up         
 ```

* The Edge devices only peer with the RRs, hence why you only see two BGP sessions, check that their **State** is **Established**.


```
6300-3-1# show evpn evi 
L2VNI : 10
    Route Distinguisher        : 192.168.1.3:10
    VLAN                       : 10
    Status                     : up
    RT Import                  : 65001:268435466
    RT Export                  : 65001:268435466
    Local MACs                 : 1
    Remote MACs                : 1
    Peer VTEPs                 : 1

L3VNI : 10010
    Route Distinguisher        : 192.168.0.3:10010
    VRF                        : Prod
    Status                     : up
    RT Import                  : 65001:10010
    RT Export                  : 65001:10010
    Local Type-5 Routes        : 2
    Remote Type-5 Routes       : 4
    Peer VTEPs                 : 3

L3VNI : 10020
    Route Distinguisher        : 192.168.0.3:10020
    VRF                        : Dev
    Status                     : up                            
    RT Import                  : 65001:10020
    RT Export                  : 65001:10020
    Local Type-5 Routes        : 1
    Remote Type-5 Routes       : 3
    Peer VTEPs                 : 3

```
* This shows us the EVPN Instance configuration.
Here you can see the **segment** configuration, the L2VNI mapped to the customer-side VLAN 10. ACN uses the same ID for the VXLAN virtual network.
* Essentially this tells us that if customer traffic is received tagged with VLAN 10, will be be encapsulated with VXLAN and the VXLAN VNI (the ID) of 10 as well.
The two L3VNIs are created to support the two VRFs we created.

> * In EVPN-VXLAN networks, the L2VNIs are mapped to the customer-side VLANs. They are the virtual networks that are the continuation of the customer-side broadcast domain into the VXLAN fabric. The L2VNIs can be configured on different VTEPs, to stretch the broadcast domain across the network (allowing the cusomter VLAN at multiple sites) or they can just be local to a single site.
>  * The L3VNIs are more complex - ACN supports routing between customer-side networks using a feature called Symmetrical Integrated Routing and Bridging (IRB). The L3VNIs enable this IRB by providing a VRF-specific VNI that is shared by all the VTEPs in a single VRF. This allows these VTEPs to route between each other, without having to be configured with every L2VNI on the network. Yes, this is pretty complex stuf. Bottom line - you want to see a L3VNI for each VRF. 


```
6300-3-1# show run bgp
router bgp 65001
    bgp router-id 192.168.0.3
    neighbor fabric_3 peer-group
    neighbor fabric_3 remote-as 65001
    neighbor fabric_3 fall-over
    neighbor fabric_3 update-source loopback 0
    neighbor 192.168.0.1 peer-group fabric_3
    neighbor 192.168.0.2 peer-group fabric_3
    address-family l2vpn evpn
        neighbor 192.168.0.1 activate
        neighbor 192.168.0.1 send-community extended
        neighbor 192.168.0.2 activate
        neighbor 192.168.0.2 send-community extended
    exit-address-family
!
    vrf Dev
        address-family ipv4 unicast
            redistribute connected
            redistribute local loopback
        exit-address-family
        address-family ipv6 unicast
            redistribute connected
        exit-address-family
!                                                              
    vrf Prod
        address-family ipv4 unicast
            redistribute connected
            redistribute local loopback
        exit-address-family
        address-family ipv6 unicast
            redistribute connected
        exit-address-family
        
```
* Here you can see the full BGP configuration. We usew a peer-group for the RRs - note L2VPN EVPN address-family, and VRF-specific unicast AFs.


### Lab Task 8 - Generate client traffic and verify the RTs
*Let's generate some traffic from the clients.*

1. Back on the workbench topology, log into the *Windows Wired Client* connected to 6300A (right-click & hit **Console Access** - this will log you in using the default 'Aruba' user account)
2. Hit **Start** and type **cmd** or **powershell**, if that's your thing!
3. Eth0 is the mgmt interface **DO NOT TOUCH THAT**, we are going to work with the other interfacem, should be **Eth2**.
4. Hit Start, type **Control Panel -> Network and Internet -> Network and Sharing Center -> Change adapter settings -> *Right-click* Ethernet 2 -> Properties**
5. You should now have the Ethernet 2 Properties box. (Make sure you are not on Eth0 or you will cut yourself off!). **Enable IIIE 802.1X authentication** should be ticked and the authentication method set to PEAP.
6. Click **Authentication -> Additional Settings**

    ![eth2](/images/eth2-properties.png)

7. Ensure that the settings are set to **User authentication** Hit the box next to it, which will say something like *Replace credentials*

    ![additional](/images/additional.png)

8. Enter the authentication credentials of one of the Employees:
    luke/admin12345!
9. *Ok* through the boxes back to the 'Network Connections' box. Windows will now fire off the 802.1X authentication process using Luke's credentials.
10. Jump back onto your 6300-x-1 device (6300A in the Workbench topology) and enter:
    
    `show port-access clients int 1/1/5`

Now you will be able to see the details of the employee, then VLAN dynamically assigned, and the VXLAN-GBP policy applied:

```
6300-3-1# show port-access clients int 1/1/5 detail 

Port Access Client Status Details:

Client a0:ce:c8:1d:5b:6b, TSSLAB\luke
=====================================
  Session Details
  ---------------
    Port         : 1/1/5
    Session Time : 98015s
    IPv4 Address : 
    IPv6 Address : 
    Device Type  : 

  VLAN Details
  ------------
    VLAN Group Name : 
    VLANs Assigned  : 10
      Access          : 10
      Native Untagged : 
      Allowed Trunk   : 

  Authentication Details
  ----------------------
    Status          : dot1x Authenticated                      
    Auth Precedence : dot1x - Authenticated, mac-auth - Not attempted
    Auth History    : dot1x - Authenticated, 794s ago
                      dot1x - Authenticated, 7824s ago
                      dot1x - Unauthenticated, Server-Reject, 7842s ago
                      dot1x - Unauthenticated, Server-Reject, 8130s ago
                      dot1x - Unauthenticated, Server-Timeout, 8263s ago

  Authorization Details
  ----------------------
    Role   : Employee
    Status : Applied


Role Information:

Name  : Employee
Type  : local
----------------------------------------------
    Reauthentication Period             : 
    Cached Reauthentication Period      : 
    Authentication Mode                 : 
    Session Timeout                     : 
    Client Inactivity Timeout           :                      
    Description                         : 
    Gateway Zone                        : 
    UBT Gateway Role                    : 
    UBT Gateway Clearpass Role          : 
    Access VLAN                         : 
    Native VLAN                         : 
    Allowed Trunk VLANs                 : 
    Access VLAN Name                    : Corp-access
    Native VLAN Name                    : 
    Allowed Trunk VLAN Names            : 
    VLAN Group Name                     : 
    MTU                                 : 
    QOS Trust Mode                      : 
    STP Administrative Edge Port        : 
    PoE Priority                        : 
    PVLAN Port Type                     : 
    Captive Portal Profile              : 
    Policy                              : 
    GBP                                 : Employee_r2r_policy
    Device Type                         : 


Access GBP Details:                                            

GBP Name   : Employee_r2r_policy
GBP Type   : Local
GBP Status : Applied

SEQUENCE    CLASS                            TYPE     ACTION
----------- -------------------------------- -------- -----------------------
10          Employee_ALLOW                   gbp-ipv4 permit   
20          Employee_ALLOW                   gbp-ipv6 permit   
30          Employee_ALLOW                   gbp-mac  permit   


Class Details:

class gbp-ip Employee_ALLOW
    1 match any Employee Employee
class gbp-ipv6 Employee_ALLOW
    1 match any Employee Employee
class gbp-mac Employee_ALLOW
    1 match Employee Employee any
 
```

11. Now that the client is deemed to be sending traffic in VLAN 10, the 6300 will record the client's source MAC in its EVPN table, to be advertised to EVPN peers as a Route-Type 2. Verify this by entering the following:

    `show bgp l2vpn evpn vni 10`

```
6300-3-1# show bgp l2vpn evpn vni 10
Status codes: s suppressed, d damped, h history, * valid, > best, = multipath,
              i internal, e external S Stale, R Removed, a additional-paths
Origin codes: i - IGP, e - EGP, ? - incomplete

EVPN Route-Type 2 prefix: [2]:[ESI]:[EthTag]:[MAC]:[OrigIP]
EVPN Route-Type 3 prefix: [3]:[EthTag]:[OrigIP]
EVPN Route-Type 5 prefix: [5]:[ESI]:[EthTag]:[IPAddrLen]:[IPAddr]
VRF : default
Local Router-ID 192.168.0.3

     Network                                               Nexthop         Metric     LocPrf    Weight   Path
------------------------------------------------------------------------------------------------------------
Route Distinguisher: 192.168.1.3:10       (L2VNI 10)
*>  [2]:[0]:[0]:[00:00:00:00:00:01]:[172.23.10.254]        192.168.1.3     0          100        0       ?
*>  [2]:[0]:[0]:[a0:ce:c8:1d:5b:6b]:[172.23.10.51]         192.168.1.3     0          100        0       ?
*>  [2]:[0]:[0]:[a0:ce:c8:1d:5b:6b]:[]                     192.168.1.3     0          100        0       ?
*>  [3]:[0]:[192.168.1.3]                                  192.168.1.3     0          100        0       ?

Route Distinguisher: 192.168.1.4:10       (L2VNI 10)
*>i [2]:[0]:[0]:[00:00:00:00:00:01]:[172.23.10.254]        192.168.1.4     0          100        0       ?
* i [2]:[0]:[0]:[00:00:00:00:00:01]:[172.23.10.254]        192.168.1.4     0          100        0       ?
*>i [3]:[0]:[192.168.1.4]                                  192.168.1.4     0          100        0       ?
* i [3]:[0]:[192.168.1.4]                                  192.168.1.4     0          100        0       ?
Total number of entries 8                                      

```

In the above example, the Wins10 client is this entry:
```
[2]:[0]:[0]:[a0:ce:c8:1d:5b:6b]:[172.23.10.51]
```
* The leading [2] tells us this is a Route-Type 2 - the BGP UPDATE used to share MAC/IP reachability information
* You can also see the MAC address of the client and the IP address

12. If this is a little unfamiliar, the information actually just comes from the MAC and ARP tables. You can see this information there:

```
6300-3-1# show mac-address-table 
MAC age-time            : 300 seconds
Number of MAC addresses : 1

MAC Address          VLAN     Type                      Port      
--------------------------------------------------------------
a0:ce:c8:1d:5b:6b    10       port-access-security      1/1/5      
6300-3-1# 
6300-3-1# 
6300-3-1# 
6300-3-1# show arp vrf Prod 

IPv4 Address     MAC                Port         Physical Port              State      VRF                             
-------------------------------------------------------------------------------------------------------------------
172.23.10.51     a0:ce:c8:1d:5b:6b  vlan10       1/1/5                      reachable  Prod                             

Total Number Of ARP Entries Listed: 1.
-------------------------------------------------------------------------------------------------------------------
6300-3-1# 

```

### Lab Task 9 - Let's test
*To finish, log in another couple of users across the fabric and check the traffic flows*