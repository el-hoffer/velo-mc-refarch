---
title: Microsoft Azure
---

# Microsoft Azure

## Basic Concepts
<u>[Virtual Network (VNet)](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)</u>-  An Azure vNET provides an isolated network segment in which workloads and auxiliary services can be deployed.  vNETs in Azure are a regional construct, and as such will require additional configuration (i.e. vNET peering, vWAN, etc.) to facilitate multi-region connectivity.  Similarly, subnets within a vNET are also a regional construct (though unlike AWS, are not isolated to a specific availability zone within a region).

<u>Route tables</u>-
There are two types of routing tables that may be applicable to Azure deployments:

[VNet/Custom Route Tables](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview) can be used to override or augment the default route table in a customer owned VNet (which by default just contains a default route and routes for the local VNet as well as any peered VNets) with static user-defined routes (UDRs).  Route tables can be applied to individual subnets within a VNet to provide differentiated routing (i.e. where workloads may need to route egress traffic to a virtual firewall which will in turn forward the traffic to an SDWAN edge).  It is important to note that UDRs will take precedence over any default VNet routes as well as routes dynamically learned routes (i.e. those advertised by an SDWAN edge to a Route Server) so when troubleshooting routing issues for Azure based workloads it can be helpful to navigate to the vnic of the virtual machine in question in the Azure portal and selecting "Effective Routes" as pictured:
<figure markdown>
  ![Image title](/images/azure/effective-routes.png){ width="800" }
  <figcaption></figcaption>
</figure>

[vWAN Hub Route Tables](https://learn.microsoft.com/en-us/azure/virtual-wan/about-virtual-hub-routing)

<u>[VNet Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)</u>-  VNet peering is a method of providing connectivity between two or more VNets to facilitate intercommunication.  Peered VNets can reside in a single region or span multiple regions, and in many cases (i.e. where traffic comes in via an SDWAN edge and must be routed through a virtual security appliance before being forwarded to the destination workload) traffic may traverse multiple VNets to reach its final destination.

When peering multiple workload and service VNets in a customer environment, Microsoft recommends a [hub and spoke architecture](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?tabs=cli) where the SDWAN edge is deployed in a hub VNet (either in an actual vWAN hub or a customer owned VNet acting as a "hub" in the peering topology).  In cases where spoke VNets are peered to a vWAN hub, flow paths through applicable VNets can then be controlled via vWAN route tables that dictate routes to be learned by (associated to) a vWAN hub attachment, or through UDRs and/or dynamic routes inhereted through peering link settings (in cases where two customer owned VNets are peered).

<u>[Azure Route Server](https://learn.microsoft.com/en-us/azure/route-server/overview)</u>- 

<u>[Azure Virtual WAN (vWAN)](https://learn.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about)</u>- 

<u>[Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/overview)</u>- 
## Single Region, Single VNet- Edge
## Single Region, Single VNet- NSD
## Single Region, Multi-VNet
## Multi-Region, Multi-VNet
## Azure VMware Solution (AVS)