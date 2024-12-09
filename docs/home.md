![Lab Overview](images/hpe-logo2.svg)

![Disclosure](images/disclose.png)


<h1>Data Center Networking - Microsegmentation with AFC & CX 10K</h1>

<h2>Technical Enablement Hands-On Lab Guide</h2>

### Lab Overview
This Lab Guide will walk you through the various parts of this Hands On Lab session.  During this test dive session, you will:

* Use Aruba Fabric Composer to depoly endpoint groups, rules, and stateful policies

* Deploy microsegmentation as part of a distributed services architecture based on the CX 10000 Switch integrated stateful firewall preventing the east-west traffic associated with security threats

* Leverage the CX 10000 telemetry to gain full visibility into every flow on the network

#### Objective
This Hands-On Lab is composed of the following activities:

1. Deploy micro segmentation to apply and enforce a traffic policy between two VMs (Workloads)
2. Test the policy applied between the Workloads

### Microsegmentation, the basics.
Traditionally, if you wanted to deploy a firewall policy between two different hosts, it would have to be defined on a Layer 3 boundry, like a router interface. This is called MACRO segmentation. Microsegemtation, is the capability to deploy a firewall policy between two hosts on the same subnet. This solution is built on Private VLAN. When a host is assigned to a private vlan, it can only communicate with the primary vlan interface. It cannot communicate with any other host on the same Layer 2 network. With the implementation of the DPU chips in the CX10K, A packet arriving at the Primary vlan, on the VMware distributed virtual switch, it is forwarded to the DPU where a stateful firewall decision can be made. That frame can be sent back the the VMware distributed virtual switch and forwared to any other host defined by the security policy. For an example, in the workshop, a policy will be defined that controls the traffic sent between the ubuntu workloads.
