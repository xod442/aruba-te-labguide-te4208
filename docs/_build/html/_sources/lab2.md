# Lab 2 - AFC Configuration  

## Lab Overview
Lab time:  45 minutes  

The spine leaf architecture provides for massive bandwidth between racks in the data center. Another reason for deploying a spine leaf is resiliency. The spine switches connect to leafs, the leafs, to the spines. The connections between them, are routed IP layer 3 links. They will not pass VLAN information. To get layer 2 traffic between the leafs, VXLAN must be used to create an overlay network.  

![Spine Leaf Architecture](images/lab2-spine-leaf-architecture.png)  
_Fig. Spine Leaf Architecture_  

```{note}
VSX Configuration is being skipped in this workshop due to limited hardware availability, but it is always recommended to configure a high-availability setup for critical server workloads. Refer to our Validated Solutions Guide (VSG) to learn more:  

https://www.arubanetworks.com/techdocs/VSG/docs/050-dc-deploy/esp-dc-deploy-120-fabric-deploy/#configure-vsx-on-leaf-switches
```  

You were _assigned a pod number_ at the start of the workshop. Add a zero in front of it and it becomes the _lab group number_. In the third octet of the IP address, start with the number 2 and add your lab group number to it. As an example, user1 = lab group 01 – 3rd octet, 201. The AFC IP for lab group 01 would be 10.250.201.30

## Lab 2.1 - Create Layer 3 Spine/Leaf Fabric  

### Description  
In this workflow the Aruba Fabric Composer (AFC) will automate configuring the routed interfaces and assigning IP addresses.  Once complete, the network will be ready to deploy the Underlay network.  

### Validate  
1. Open a new browser tab/window and log into the HPE Aruba Networking Fabric Composer (AFC).  Use the following URL and credentials to access the AFC.

|||
|---|---|
|URL|``https://10.250.2LG.30`` where **LG = your Lab Group Number**|
|Username|``admin``|
|Password|``admin``|  

![AFC Login](images/lab2-afc-login.png)  
_Fig. AFC Login_  

2. Once logged in, we will use the **L3 Leaf-Spine Configuration** worflow on the right hand side to launch the configuration workflow. This workflow will configure all the routed interfaces used by the spine-leaf.  

```{note}
If the workflows on the right-hand side are not present, click on the icon next to the person at the top right
```  

![AFC Wizard Menu](images/lab2-afc-wizard-menu.png)  
_Fig. AFC Wizard Menu_  

3. Follow the workflow by filling out the forms and clicking next. Finish by reviewing content and selecting **Apply**.  

![Workflow Screens](images/lab2-workflow-screens.png)  
_Fig. Workflow Screens_  

|||
|---|---|
| **Step 1:  Create Mode** ||
| Automatically generate L3 Leaf-Spine Pairs| Select |
| Click **NEXT** ||  

|||
|---|---|
| **Step 2:  Name** ||
| Name Prefix | dsa-L3 |
| Description | DSA Leaf-Spine Pairs |
| Click **NEXT** ||  

|||
|---|---|
| **Step 3:  Settings** ||
| IPv4 Address Pool | From the pulldown, select ``10.100.0.0/24`` **DO NOT CLICK ADD**|
| Click **NEXT** ||  
| Verify the settings and click **APPLY** to commit ||  


### Expected Results  
You should see the newly created configuration, similar to the below screenshot.  

![L3 Leaf-Spine Created](images/lab2-leaf-spine-created.png)  
_Fig. L3 Leaf-Spine Created_  


## Lab 2.2 - Configure the Underlay  

### Description  
The underlay network connects each switches loopback interface via OSPF routing. In this task OSPF routing will be configured on all switches. AFC has a built-in workflow that make it easy to create the underlay network.  

### Validate  
1. Begin by choosing the UNDERLAYS workflow from the right hand menu, to launch the configuration workflow.  

![Underlays Menu](images/lab2-underlays-menu.png)  
_Fig. Underlays Menu_  

```{note}
Also accessed via the Configuration/Routing/VRF/default/Underlays tab under the Actions menu
```  

2. Follow the workflow by filling out the forms and clicking next. Finish by reviewing content and selecting **Apply**.  

![Create Underlay Network](images/lab2-create-underlay-network.png)  
_Fig. Create Underlay Network_  

|||
|---|---|
| Step 1:  Name ||
| Name | dsa-underlay |
| Description | Underlay Network for the DSF Fabric |
| Click **NEXT** ||  

|||
|---|---|
| Step 2:  Underlay Type ||
| Underlay Type | OSPF |
| Click **NEXT** ||  

|||
|---|---|
| Step 3:  Settings ||
| IPv4 Address Pool | From the pulldown, select ``10.100.0.0/24`` **DO NOT CLICK ADD**|
| Transit VLAN | 1000 |
| Leave all other default settings and click **NEXT** ||  

|||
|---|---|
| Step 4:  Max Metric ||
| Leave all settings as default | |
| Click **NEXT** ||  
| Review the Summary page, and click **APPLY** to commit the changes ||  

![Underlay Created](images/lab2-underlay-created.png)  
_Fig. Underlay Created_  

 
### Expected Results  

1. You can see the underlay exists in the **Underlays Tab** above, use the embedded **CLI Command Tool** to verify the loopback interfaces and confirm the underlay routing tables have been established.  

![Show Commands Menu](images/lab2-show-commands-menu.png)  
_Fig. Show Commands Menu_  

2. Pull down the fabric menu and select the **dsa** fabric  

3. In the commands box, enter each of the following commands, one at a time and click the **RUN** button.  

- ``show run interface loopback 0``  
- ``show ip ospf routes``  

```{note}
Run each command individually for best results.  Do not truncate commands.

Examine the results of the commands. You just enabled OSPF underlay routing!
```  


## Lab 2.3 - Configure the Overlay  

### Description  
With the Underlay established, the next step is to configure the Overlay Network. The Aruba ESP Data Center spine-and-leaf design uses the **Internal Border Gateway Protocol (iBGP)** as the control plane for the fabric overlay. iBGP provides a mechanism to share host routes across the fabric using the EVPN address family. VTEP interfaces are the VXLAN encapsulation and decapsulation points for traffic entering and exiting the overlay.  

To briefly summarize from the primer presentation and external links from the workshop’s introduction, **EVPN-VXLAN** is an industry standard network fabric technology that extends Layer 2 connectivity as a network overlay over an existing physical network. **VXLAN** is an IEEE standard that is used to address scalability issues in cloud-based environments via switch-to-switch tunneling. This protocol encapsulates Layer 2 Ethernet frames in Layer 3 UDP packets, meaning **virtual L2 subnets** can span underlying L3 networks. The Layer 2 information is simply wrapped inside a Layer 3 frame.  

![Overlay Ethernet Frame](images/lab2-overlay-eth-frame.png)  
_Fig. Overlay Ethernet Frame_  

A **Virtual Network Identifier (VNI)** is mapped to traditional VLAN IDs. **VXLAN Endpoints**, in turn, terminate VXLAN tunnels, which can be virtual or physical, and are therefore referred to as **VXLAN Tunnel Endpoints (VTEPS)**.  

```{note}
This is very complex configuration. The AFC makes it very easy.
```  

Bearing all this in mind, this task will have you use the **AFC Overlay Configuration Workflow** to implement iBGP settings using a private Autonomous System Number (ASN) to establish VTEPs. VTEP IP addresses will be assigned using a pre-defined IP address range. Finally, iBGP neighbor relationships will be verified using the AFC CLI Command Processor.  

### Validate   

1. 3.1	In the Configuration workflows on the right-hand side, click **Overlays**.  

![Configure Overlays](images/lab2-configure-overlays.png)  
_Fig. Configure Overlays_  



2. In the Overlay Configuration Workflow, configure the following parameters for each page.  

![Overlay Workflow](images/lab2-overlay-workflow.png)  
_Fig. Overlay Workflow_  

|||
|---|---|
| Step 1:  Name ||
| Name | dsa-overlay |
| Description | Overlay for the DSA Fabric |
| Click **NEXT** ||  

|||
|---|---|
| Step 2:  Overlay Type ||
| Ovderlay Type | iBGP ***default setting*** |
| Click **NEXT** ||  

|||
|---|---|
| Step 3:  iBGP Settings ||
| Spine-Leaf ASN | 65000.0 ***default setting, ASPlain notation*** |
| Route Reflector Servers | Select **LG[LG]-Spine** from the drop-down |
| Leaf Group Name | leaf  |  
| Spine Group | spine-RR  |  
| Leave all other default settings and click **NEXT** ||  

|||
|---|---|
| Step 4:  IPv4 Network Address ||
| IPv4 Address Resource Pool | From the pulldown, select ``10.100.0.0/24`` **DO NOT CLICK ADD** |
| Click **NEXT** ||  

|||
|---|---|
| Step 5:  Settings ||
| Authentication Password | _Ensure that this field is not auto-filling_.  The password should be **Aruba123** (case-sensitive).  Leave all other settings as default |
| Click **NEXT** ||  
| Review the Summary page, and click **APPLY** to commit the changes ||  

![Overlay Created](images/lab2-overlay-created.png)  
_Fig. Overlay Created_  


### Expected Results  

1. Verify the overlay under the **Overlays Tab** and use the embedded **CLI Command Tool** to verify BGP has been established.  

2. Use the show commands again and examine the results. The command to execute against the “DSA” fabric is:  ``show run bgp``  


## Lab 2.4 - EVPN Configuration  

### Description  
With the underlay and overlay established, specific EVPN parameters can be set using the steps below for use within the overlay. An EVPN instance joins a distributed set of VLANs across multiple switches into a single logical broadcast domain with a common L2 VNI. A prefix value is provided for automatic generation of route targets, and a resource pool is used to assign the EVPN system MAC addresses. Once this task is completed, distributed L2 connectivity across both switches in the fabric will be established.  

### Validate   

1. In the Configuration Workflows on the right-hand side, click **EVPN Configuration**  

![EVPN Configuration](images/lab2-evpn-configuration.png)  
_Fig. EVPN Configuration_  

```{note}
Also access via Configuration/Routing/EVPN/Actions Menu/Add
```  

2. In the EVPN Configuration Workflow, configure the following parameters for each page.

![EVPN Workflow](images/lab2-evpn-workflow.png)  
_Fig. EVPN Workflow_  

|||
|---|---|
| Step 1:  Intro ||
| Read the Introduction adn click **NEXT** ||  

|||
|---|---|
| Step 2:  Switches ||
| Create EVPN instances across the entire Fabric and all Leaf and Border Leaf Switches contained within it | Yes **(default setting)** |
| Click **NEXT** ||  

|||
|---|---|
| Step 3:  Name ||
| Name | dsa |
| Description | EVPN Configuration for the DSF Fabric |
| Click **NEXT** ||  

|||
|---|---|
| Step 4:  VNI Mappings ||
| VLANs | 10,20 |  
| Base L2VNI | 2000|  
| Click **NEXT** ||  

|||
|---|---|
| Step 5:  Settings ||
| MAC Address Resource Pool | 02:00:00:00:00:aa-02:00:00:00:00:ff **pull down and select predefined MAC address pool** |
| Route Target Type | ASN:VLAN (Scroll down to see in the drop-down) |
| Route Target ASN Prefix | 65001 |  
| Leave all other default settings and click **NEXT** ||  
| Review the Summary and click **APPLY** to commit ||  


![EVPN Created](images/lab2-evpn-created.png)  
_Fig. EVPN Created_  


### Expected Results  

Verify the configuration under the EVPN Tab. Observe that VLAN entries with corresponding L2VNIs have been automatically generated for each Leaf Switch based on the parameters defined in the step above. Imagine the time savings at scale!  

At this point any workstations on VLAN 10 should be able to reach each other. No matter where in the network they are. To get a workstation on VLAN 10 (Switch 1) to reach a workstation on VLAN 20 (Switch2), Symmetrical IRB, or Integrated Route Bridging will need to be configured. This is commonly referred to as an **L3-VNI** or Virtual Network Interface. 


## Lab 2.5 - Create Overlay VRF

### Description  
Overlay VRFs are used to provide the Layer 3 virtualization and macro segmentation required for flexible and secure data centers and are distributed across all leaf switches. A VRF instance on one switch is associated to the same VRF on other leaf switches using a common Layer 3 VNI and EVPN route-target, binding them together into one logical routing domain.  

The below steps will utilize the **Virtual Routing & Forwarding** workflow to create an overlay network VRF and associate that VRF with a **Layer 3 VNI** and **EVPN route target**. The VNI and route target for the overlay VRF must be unique to preserve traffic separation.

### Validate  
1. To create a VRF for the overlay, navigate to **Configuration/Routing/VRF** using the top menu.  

2. In the **ACTIONS** menu, select **Add** to create a new **VRF**. The following screen will appear.  

![Create VRF](images/lab2-create-vrf.png)  
_Fig. Create VRF_  

3. Configure the following parameters:  

|||  
|---|---|  
| Step 1: Name ||  
| Name | over |  
| Description | Overlay VRF |
| Click **NEXT** ||  

|||  
|---|---|  
| Step 2: Scope ||  
| Apply the VRF to the entire fabric and all switches contained within it | Select ***default setting*** |  
| Click **NEXT** ||  

|||  
|---|---|  
| Step 3: Routing ||  
| L3 VNI | 3000 |  
| Click **NEXT** ||  

|||  
|---|---|  
| Step 4: Route Target ||  
| Route Target Mode | both |  
| Route Target Ext-Community | 65001:1 |  
| Address Family | EVPN |  
| Click **ADD** to add the entry to the table||  
```{note}
Stay on the same page!
```

|||  
|---|---|  
| ***Create a second Route Target Entry using the following parameters*** ||  
| Route Target Mode | both |  
| Route Target Ext-Community | 65001:2 |  
| Address Family | IPv4 Unicast |  
| Click **ADD** to add the entry to the table||  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 5: Summary ||  
| Verify the settings, then click **APPLY** ||  


### Expected Results  
You shoudl see the newly created Overlay VRF in the VRF list as in the screenshot below.  

![Overlay VRF Created](images/lab2-overlay-vrf-created.png)  
_Fig. Overlay VRF Created_  

## Lab 2.6 - Create VLAN Interfaces

### Description  
Now that we have the newly created Overlay VRF, we will add VLAN interfaces to it.  

### Validate  
1. Click the _three green dots_ on the left-hand side of the **Overlay VRF** you created called **over** and select **IP Interfaces**.  

2. In the **IP Interfaces** Tab, click **ACTIONS** on the right-hand side and select **Add**  

![IP Interfaces](images/lab2-ip-interfaces.png)  
_Fig. IP Interfaces_  

3. Create the Switched Virtual Interfaces (SVI) VLAN 10 with the following parameters:  

|||  
|---|---|  
| Step 1: Interface Type ||  
| Enable this IP Interface | Select ***(default setting)*** |  
| Type | SVI ***(default setting)*** |  
| VLAN | 10 |  
| Switches | Select both CX 10000 Leaf switches |  
| IPv4 Subnetwork Address | ``10.0.10.0/24`` |  
| IPv4 Addresses | ``10.0.10.10-10.0.10.20`` |  
| Active Gateway IP Address | ``10.0.10.1`` |  
| Active Gateway MAC Address | ``02:00:00:00:00:01`` |  
| Enable VSX Shutdown on Split | Select ***(does not apply to this lab, but turn on)*** |  
| Enable Local Proxy ARP | Select |  
| Click **NEXT** ||  

|||  
|---|---|  
| Step 2: Name ||  
| Name | SVI10 |  
| Description | Interface VLAN 10 |  
| Click **NEXT** ||  

|||  
|---|---|  
| Step 3: Summary ||  
| Verify the settings and click **APPLY** to save the configuration ||  

![VLAN10 SVI](images/lab2-vlan10-svi.png)  
_Fig. VLAN10 SVI_  

4. Repeat the same process to create the SVI for VLAN 20 now - In the **IP Interfaces** Tab, click **ACTIONS** on the right-hand side and select **Add** and enter the following paramaters:  

|||  
|---|---|  
| Step 1: Interface Type ||  
| Enable this IP Interface | Select ***(default setting)*** |  
| Type | SVI ***(default setting)*** |  
| VLAN | 20 |  
| Switches | Select both CX 10000 Leaf switches |  
| IPv4 Subnetwork Address | ``10.0.20.0/24`` |  
| IPv4 Addresses | ``10.0.20.10-10.0.20.20`` |  
| Active Gateway IP Address | ``10.0.20.1`` |  
| Active Gateway MAC Address | ``02:00:00:00:00:01`` |  
| Enable VSX Shutdown on Split | Select ***(does not apply to this lab, but turn on)*** |  
| Enable Local Proxy ARP | Select |  
| Click **NEXT** ||  

|||  
|---|---|  
| Step 2: Name ||  
| Name | SVI20 |  
| Description | Interface VLAN 20 |  
| Click **NEXT** ||  

|||  
|---|---|  
| Step 3: Summary ||  
| Verify the settings and click **APPLY** to save the configuration ||  

![VLAN20 SVI](images/lab2-vlan20-svi.png)  
_Fig. VLAN20 SVI_  

### Expected Results  
1. Verify the underlay use the **Underlays Tab**, and then use the embedded **CLI Command Tool** to verify the loopback interfaces and confirm the underlay routing tables have been established  

![Show Commands](images/lab2-show-commands.png)  
_Fig. Show Commands_  

2. Pull down the fabric menu and select the **dsa** fabric

3. In the commands box enter each of the following, one at a time and click the **RUN** button:  

- ``show ip route vrf over``
- ``show evpn evi``

```{note}
Run each command individually for best results
```

Examine the results of the commands. When you show the evpn EVI both L2 VNI's are displayed along with the L3 VNI in support of the Symmetrical **Integrated Route Bridge (IRB)**

## Lab 2 Summary  

During this short lab, you used the highly automated AFC workflows to do the following tasks:

- Created a Layer 3 Spine / Leaf Fabric
- Created an OSPF Underlay
- Created an EVPN-VXLAN Overlay
- Created a new VRF
- Created VLAN Interfaces for VLANs that we will test out later on

The AFC provides built in workflows to automate normally complex tasks.

## Lab 2 Learninig Check

- The underlay OSPF network carries advertisments for each switches loopback **IP** address
- The overlay network is used to extwnd layer 2 networks beyond router boundries
- Layer 2 networks are assigned a **virtual network identifier** or **VNI**
- Different layer 2 networks can communicate with others via a Layer 3 VNI
