# Lab 1 - Accessing the Lab

## Lab Overview

Lab time:  15 minutes

In this lab, we are going to explore the lab environment and access the various components of the lab.  The lab consists of the following components:
- 1x CX 6300 Switch which acts as the Spine 
- 2x CX 10000 Switches configured as Leaf Switches
- 1x ProLiant Server running ESXi and connected to both Leaf Switches  

![Lab Layout](images/lab1-lab-topology.png)
_Fig. Lab Physical Layout_  

## Lab 1.1 Management IP Addresses

The following table represents the Management IP Addresses we will use to access each of the different components within the labs:  

| DEVICE    | INTERFACE | IP ADDRESS  |  
| -------- | ------- | ------- |
| Aruba Fabric Composer (AFC) | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.30 |
| Pensando PSM | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.31 |
| LGXX-spine01 | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.103 |
| LGXX-leaf01 | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.101 |
| LGXX-leaf02 | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.102 |
| Workload-01 | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.201 |
| Workload-02 | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.202 |
| Workload-03 | Managment | 10.250.2<span style="color:orange">**[LG]**</span>.203 |

## 1.2 Lab Software Versions  
| DEVICE    | SOFTWARE VERSION |  
| -------- | ------- |  
| Aruba Fabric Composer (AFC) | 7.01 |  
| Policy Services Manager | 1.80.1-T-6 |  
| CX Switches | DL 10.13.1010 |  
 


## Lab 1.3 Pre-Staged Workshop Environment  

This is an accelerated workshop. A lot of the initial setup functions have been already configured. For this workshop to be successfully completed, in a reasonable amount of time, the lab gear must be configured to a specific level. The follow items are pre-staged:
- Discovery of the network switches  
- Creating a fabric  
- Adding the switches to the data center fabric  
- Configuring Network Time Protocol (NTP)  
- MAC and IPv4 pools  
- Configuring Domain Naming Services (DNS)  
- vSphere and Pensando integrations  

### Vsphere integration
For this workshop, the vSphere integration in AFC has been pre-staged. The Virtual Machines already have an identity tag applied. Two Endpoint groups should have automatically been created. These are the Endpoint Groups used in this workshop.  

There are three (3) virtual machines (VMs) being used for this workshop that will serve as our Endpoints; **LGXX-WL01, LGXX-WL02 and LGXX-WL03**. These workloads are Ubuntu Linux VM’s The **XX** will be replaced with the _Lab Group number_ that was assigned at the start of this workshop.  

These workload VMs are tagged in vSphere and will be leveraged for Rule and Policy mapping.  

For more information on Data Center deployment check out the Aruba Validated Solutions Guide’s (VSG) VMware vSphere section located here:  

https://www.arubanetworks.com/techdocs/VSG/docs/050-dc-deploy/esp-dc-deploy-120-fabric-deploy/#vmware-vsphere-integration  

- See also AFC User Guide (Version 6.5 used for this workshop): 
https://www.arubanetworks.com/techdocs/AFC/PDFs/64/AFC64UG.pdf

### Pensando integration  

For this workshop, the AMD Pensando Policy and Services Manager (PSM) integration has been pre-staged. 

### IPv4 Address Pool  
As the Aruba Fabric Composer relies on automation to deploy the data center infrastructure, an IPv4 address pool has been created and ready to use. (**10.100.0.0/24**)  

### MAC address pool
As the Aruba Fabric Composer relies on automation to deploy the data center infrastructure, a MAC address pool has been created. This is just a pool of address that AFC can use to assign MAC addresses when required. (**02:00:00:00:00:aa-02:00:00:00:00:ff**)
