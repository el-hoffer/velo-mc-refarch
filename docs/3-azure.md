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

[vWAN Hub Route Tables](https://learn.microsoft.com/en-us/azure/virtual-wan/about-virtual-hub-routing) are applicable to vWAN hub connections (e.g. NVAs, VNet Connections, Express Routes, etc.) and are not specific to a given VNet or peering connection.  As such provide a more flexible way to control which routes are associated to and propagated from a given connection as well as being more repeatable in the sense that a single route table can be applied to multiple connections.

<u>[VNet Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)</u>-  VNet peering is a method of providing connectivity between two or more VNets to facilitate intercommunication.  Peered VNets can reside in a single region or span multiple regions, and in many cases (i.e. where traffic comes in via an SDWAN edge and must be routed through a virtual security appliance before being forwarded to the destination workload) traffic may traverse multiple VNets to reach its final destination.

When peering multiple workload and service VNets in a customer environment, Microsoft recommends a [hub and spoke architecture](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?tabs=cli) where the SDWAN edge is deployed in a hub VNet (either in an actual vWAN hub or a customer owned VNet acting as a "hub" in the peering topology).  

In cases where spoke VNets are peered to a vWAN hub, flow paths through applicable VNets can then be controlled via vWAN route tables that dictate routes to be learned by (associated to) a vWAN hub VNet Connection.  This capability represents a major advantage of using vWAN over a customer managed Azure Route Server as the hub route tables allow for far more flexibility as it pertains to the learning/advertising of both static and dynamic routes, as well as the ability to re-use route tables across multiple connections (whereas manual peering must be defined on a per peering connection basis).  When defining vWAN hub VNet connection, pertinent routing settings include the route table to associate to the connection (i.e. the routes to be advertised to the spoke VNet being connected to the hub), the route table(s) to propagate routes to (i.e. which route tables will learn the routes of the spoke VNet), "labels" to propagate routes to (which provides an abstraction to tag multiple route tables that can then be referenced in a single rule), as well as any needed static routes to add as depicted:
<figure markdown>
  ![Image title](/images/azure/VNet-cxn.png){ width="800" }
  <figcaption></figcaption>
</figure>

When customer owned VNets are peered, routing is controlled through UDRs and/or dynamic routes inhereted through peering link settings, which allow the selection of one VNet/route server from which to inherit dynamically learned routes (generally the hub VNet where the SDWAN edge will reside) as shown below:
<figure markdown>
  ![Image title](/images/azure/peering-settings.png){ width="800" }
  <figcaption></figcaption>
</figure>

<u>[Azure Route Server](https://learn.microsoft.com/en-us/azure/route-server/overview)</u>- 

<u>[Azure Virtual WAN (vWAN)](https://learn.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about)</u>- 

<u>[Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/overview)</u>- 
## Single Region, Single VNet- Edge
## Single Region, Single VNet- NSD
## Single Region, Multi-VNet
## Multi-Region, Multi-VNet
## Azure VMware Solution (AVS)
