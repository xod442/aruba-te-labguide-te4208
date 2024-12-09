# Lab 3 - Create EPG's and Deploy a Policy  

## Lab Overview
Lab time:  30 minutes  

With all the systems in place, we can now leverage the various components to create a policy that can be deployed from a single pane of glass to multiple switches within the fabric.  

## Lab 3.1 - Redirect VLANs to DSM  

### Description  
The first step is to Configure the VRFs to sync with the Policy Service Manager that will utilize the defined networks..  

### Validate   
1. In the Guided Setup Pane, switch from the **Network** tab to the **Distributed Services** tab.  _Be sure to switch over to Distributed Services from the Network Tab_!  

![Distributed Services Menu](images/lab3-distributed-services-menu.png)  
_Fig. Distributed Services Menu_  

2. Click **Configure VRFs** to navigate to the VRFs page under **Configuration/Routing**  

3. In the Configure VRFs page, within the right-hand pane for Distributed Services, choose **dsa/over** as the selected VRF in the **Distributed Services Setup** drop-down, then select **Configure Networks** just below the VRF selection drop-down menu

```{note}
Be sure to select the dsa/over VRF
```  

![dsa over](images/lab3-dsa-over.png)  
_Fig. dsa/over_  

```{note}
Also accessed via Configuration/Routing/VRF - [over] - (3 dots on left) /Networks/From Networks Tab/Actions Menu (mid-right) - Add
```  

4. In the **ACTIONS** menu under the **Networks** tab, click **Add** and configure the following for VLAN 10.  

![Add Redirect](images/lab3-add-redirect.png)  
_Fig. Add Redirect_  

|||
|---|---|
| Step 1: Name ||  
| Name | vlan10 |  
| Description | vlan10 |  
| Click **NEXT** to continue ||  

|||
|---|---|
| Step 2: Settings ||  
| VLAN | 10 |  
| Click **NEXT** to continue ||  
| Click **Apply** ||  

5. Repeat the previous step, but for VLAN 20.  In the **ACTIONS** menu under the **Networks** tab, click **Add** and configure the following for VLAN 20.  

|||
|---|---|
| Step 1: Name ||  
| Name | vlan20 |  
| Description | vlan20 |  
| Click **NEXT** to continue ||  

|||
|---|---|
| Step 2: Settings ||  
| VLAN | 20 |  
| Click **NEXT** to continue ||  
| Click **Apply** ||  

### Expected Results  

1. You should see the newly created networks under the Networks tab for the VRF in a green Healthy state.  

![VLANs Created](images/lab3-vlans-created.png)  
_Fig. VLANs Created_  

2. Verify the VLANs are in operation by using the embedded **CLI Command Tool** to verify their status.  

The command to execute against the leaf switches is:  
- ``show vlan``  
- VLANs 10 & 20 should be listed  

## Lab 3.2 Configure Endpoint Groups  

### Overview 
Endpoint group objects represent source and destination IP addresses referenced in firewall rules that can represent one or multiple devices. **Dynamic** endpoint groups can be **auto-populated** based on the assignment of **vSphere Tags** to VMs identified through AFC’s vCenter integration along with the PSM. All VMs assigned the same tag should be auto-populated as a member of the dynamic group when on a **distributed vSwitch**.  _When member IP addresses change or tag assignments are changed in vCenter, dynamic group objects are updated automatically_.  

```{note}
In this workshop you will see workloads that are learned from vSphere. You can ignore them, and create your own! 
```  

### Validate  
1. In the **Guided Setup Pane**, under the **Distributed Services** tab, click **Configure Policy**, followed by **Endpoint Groups** on the left-hand menu.  

![Configure Policy](images/lab3-configure-policy.png)  
_Fig. Configure Policy_  

```{note}
Also accessed via Configuration/Policy/Endpoint Groups  
```  

```{note}
To proceed with manual endpoint group creation, begin by clicking Add under the ACTIONS Menu in the top-right under Configuration/Policy/Endpoint Groups/Actions Menu (top-right)/Add
```  

![Add Enpoint Group](images/lab3-add-endpoint-group.png)  
_Fig. Add Enpoint Group_  

3. Enter the following configuration to create an Endoint Group for Workload01 & Workload02:

|||
|---|---|
| Step 1: Name ||  
| Name | Web Servers |  
| Description | Web Server EPG |  
| Click **NEXT** to continue ||  

|||
|---|---|
| Step 2: Type ||  
| Type | Layer 3 ***(default setting)*** |  
| Click **NEXT** to continue ||  

|||
|---|---|
| Step 3: Endpoints ||  
| Criteria | VM Tag ***(default setting)*** |  
| VM Tag | AFC-integration.Workload01 _(select from the dropdown)_ |  
| VNIC | Network adapter 2 ||
| Scroll down and click **Add** ||  

![VM Tag](images/lab3-vm-tag.png)  
_Fig. VM Tag_  

```{note}
Stay on the Endpoints page and repeat the process again to add Workload02, you will have two workloads in this Endpoint Group!  
```  

|||
|---|---|
| Step 4: Endpoints ||  
| Criteria | VM Tag ***(default setting)*** |  
| VM Tag | AFC-integration.Workload02 _(select from the dropdown)_ |  
| VNIC | Network adapter 2 ||
| Scroll down and click **Add** ||  
| Click **Next** to continue ||  

|||
|---|---|
| Step 5: Summary ||  
| On the Summary page, verify the settings, then click **Apply** to save the configuration. |  

![Web Server EPG](images/lab3-web-server-epg.png)  
_Fig. Web Server EPG_  

4. Now create a second Endpoint Group for the Database Servers, by clicking **Add** under the **ACTIONS** menu.  

|||
|---|---|
| Step 1: Name ||  
| Name | Database-Servers |  
| Description | Database-Servers EPG |  
| Click **NEXT** to continue ||  

|||
|---|---|
| Step 2: Type ||  
| Type | Layer 3 ***(default setting)*** |  
| Click **NEXT** to continue ||  

|||
|---|---|
| Step 3: Endpoints ||  
| Criteria | VM Tag ***(default setting)*** |  
| VM Tag | AFC-integration.Workload03 _(select from the dropdown)_ |  
| VNIC | Network adapter 2 ||
| Scroll down and click **Add** ||  
| On the Summary page, verify the settings, then click **Apply** to save the configuration. |  


### Expected Results  

Verify the Endpoint Groups were created successfully, as shown in the following screenshot.  

![Endpoint Groups](images/lab3-endpoint-groups.png)  
_Fig. Endpoint Groups_  

## Lab 3.3 Create Firewall Rules  

### Description  
The goals of the rules that will be used to enforce the policy for this workshop, seek to achieve three things in this scenario:  

- Allow **workload01** and **workload02** to SSH into **workload03**  
- Block everything else between these two workloads  
- Allow everything else

### Validate  
1. Under the **Configuration** menu, go to **Policy/Rules**  

![Policy Rules Menu](images/lab3-policy-rules-menu.png)  
_Fig. Policy Rules Menu_  

2. First, we will create the ``Allow SSH Web to DB Rule``.  Under **ACTIONS**, click **ADD** to create a rule, and configure the following parameters.  

```{note}
Stick to lower-case & NO SPACES for the Rule Name (may result in errors when applying these rules to a policy if not entered the same as below as the UI does not restrict case at this time.)
```  

![Rule Workflow](images/lab3-rule-workflow.png)  
_Fig. Rule Worklow_  

|||  
|---|---|  
| Step 1: Name ||  
| Name | allow-ssh-web-db |  
| Description | Allow SSH between Web and Database Servers |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 2: Settings ||  
| Type | Layer 3 |  
| Action | Allow ***(default setting)*** |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 3: Endpoint Groups ||  
| Source Endpoint Group | Web-Servers |  
| Destination Endpoint Group | Database-Servers |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 4: Applications and Service Qualifiers  ||  
| Applications | ***leave empty*** |  
| Service Qualifiers | SSH ***(type ssh)*** |  
| Click **NEXT** to continue ||  
| Review the Summary and click **Apply** ||  

3. Now we will create the second rule: ``Web to Database Rule``.  Under **ACTIONS**, click **ADD** to create a rule, and configure the following parameters.  

|||  
|---|---|  
| Step 1: Name ||  
| Name | drop-all-web-db |  
| Description | Block all other traffic between web and database |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 2: Settings ||  
| Type | Layer 3 |  
| Action | Drop |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 3: Endpoint Groups ||  
| Source Endpoint Group | Web-Servers |  
| Destination Endpoint Group | Database-Servers |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 4: Applications and Service Qualifiers  ||  
| Applications | ***leave empty*** |  
| Service Qualifiers | all |  
| Click **NEXT** to continue ||  
| Review the Summary and click **Apply** ||  

4. Now we will create the third rule: ``Database to Web Rule``.  Under **ACTIONS**, click **ADD** to create a rule, and configure the following parameters.  

|||  
|---|---|  
| Step 1: Name ||  
| Name | drop-all-db-web |  
| Description | Block all other traffic between database and web |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 2: Settings ||  
| Type | Layer 3 |  
| Action | Drop |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 3: Endpoint Groups ||  
| Source Endpoint Group | Web-Servers |  
| Destination Endpoint Group | Database-Servers |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 4: Applications and Service Qualifiers  ||  
| Applications | ***leave empty*** |  
| Service Qualifiers | all |  
| Click **NEXT** to continue ||  
| Review the Summary and click **Apply** ||  

5. Now we will create the last and catch all rule: ``Allow Any to Any Rule``.  Under **ACTIONS**, click **ADD** to create a rule, and configure the following parameters.  

|||  
|---|---|  
| Step 1: Name ||  
| Name | allow-all |  
| Description | Allow all traffic |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 2: Settings ||  
| Type | Layer 3 |  
| Action | Allow |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 3: Endpoint Groups ||  
| Source Endpoint Group | ***leave blank*** |  
| Destination Endpoint Group | ***leave blank*** |  
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 4: Applications and Service Qualifiers  ||  
| Applications | ***leave empty*** |  
| Service Qualifiers | all |  
| Click **NEXT** to continue ||  
| Review the Summary and click **Apply** ||  

### Expected Results  
When finished, you should see four rules like in the screenshot below.  

![Rule List](images/lab3-rule-list.png)  
_Fig. Rule List_  

## Lab 3.4 Create Firewall Policy  

### Description 

Now that the rules are in place, create a Policy to enforce those rules, showcasing the power of the CX10K to apply stateful firewall rules at scale and effectively govern east-west traffic.

### Validate - Ingress policy 
1.  In the **Configuration Menu**, go to **Policy/Policies**.

```{note}
Also accessed via the ‘Guided Setup Pane/Distributed Services Tab/Configure Policy’
```
2.  Under the Actions Menu, select **Add** to create a policy and configure the following parameters:

![Policy](images/pol1.png)  
_Fig. Policy_  

|||  
|---|---|  
| Step 1. Name ||  
| Name | ``dsa-ingress-policy`` |     
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 2. Policies ||  
| Type | ``distributed firewall`` |     
| Click **NEXT** to continue ||  

Step 3.

Add rules to the policy by using the **Actions** menu and select **Add**. You will add existing rules to this policy, make sure they are in the correct order.

Using the arrows on the left move the rules into the correct order. Be sure to move the allow-all rule to the bottom of the sequence (rule #4)

Click **NEXT** to continue


![Arange Rules](images/pol2.png)  
_Fig. Arange Rules_  


3. Add the enforcer information.

![Arange Rules](images/pol3.png)  
_Fig. Enforcers_  

|||  
|---|---|  
| Step 1. Name ||  
| Faric | ``dsa`` |
| Policy Distribution Type | ``leaf`` |
| Direction | ``ingress`` |
| VRF | ``over`` |
| Networks | ``Pick both vlan 10 & 20`` |
| Click **ADD** , next to the **CLEAR** button |    
| Click **NEXT** and **APPLY** ||  

### Define Egress Policy

1. Using the same process, add another policy for the **egreess** policy.

2.  Under the Actions Menu, select **Add** to create a policy and configure the following parameters:

![Policy](images/pol1.png)  
_Fig. Policy_  

|||  
|---|---|  
| Step 1. Name ||  
| Name | ``dsa-egress-policy`` |     
| Click **NEXT** to continue ||  

|||  
|---|---|  
| Step 2. Policies ||  
| Type | ``distributed firewall`` |     
| Click **NEXT** to continue ||  

Step 3.

Add rules to the policy by using the **Actions** menu and select **Add**. You will add existing rules to this policy, make sure they are in the correct order.

Using the arrows on the left move the rules into the correct order. Be sure to move the allow-all rule to the bottom of the sequence (rule #4)

Click **NEXT** to continue


![Arange Rules](images/pol2.png)  
_Fig. Arange Rules_  


3. Add the enforcer information.

![Arange Rules](images/pol3.png)  
_Fig. Enforcers_  

|||  
|---|---|  
| Step 1. Name ||  
| Faric | ``dsa`` |
| Policy Distribution Type | ``leaf`` |
| Direction | ``egress`` |
| VRF | ``over`` |
| Networks | ``Pick both vlan 10 & 20`` |
| Click **ADD** , next to the **CLEAR** button |    
| Click **NEXT** and **APPLY** ||  

```{note}
NOTE: If you don’t see both policies are healthy, click on the refresh wheel next to the actions menu
```

### Expected Results  

Verify the policies.
![Arange Rules](images/pol5.png)  
_Fig. Policies_  


## Lab 3 Summary

In this lab you redirected vlan 10 and 20 to be examined by the DPU chips on the HPE Aruba Networking CX10K switch. You have to create endpoint groups and add VMs to them. Finally, you created rules and policies to limit traffic flow between the two endpoint groups.

## Lab 3 Learning Check

- Endpoint Groups are just logical containers for sever VMs with a similar function.
- Ingress is for traffic leaving the DPU
- Egrees is for traffic coming into the DPU.
- Rules are added to policies, a rule can belong to more than one policy.
