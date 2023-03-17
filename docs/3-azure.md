---
title: Microsoft Azure
---

# Microsoft Azure

## Basic Concepts
<u>[Virtual Network (VNet)](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)</u>-  An Azure vNET provides an isolated network segment in which workloads and auxiliary services can be deployed.  vNETs in Azure are a regional construct, and as such will require additional configuration (i.e. vNET peering, vWAN, etc.) to facilitate multi-region connectivity.  Similarly, subnets within a vNET are also a regional construct (though unlike AWS, are not isolated to a specific availability zone within a region).

<u>[VNet Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)</u>- 

<u>Routing tables</u>-
There are two types of routing tables that may be applicable to Azure deployments:

[Subnet Routing Tables](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)

[vWAN Hub Route Tables](https://learn.microsoft.com/en-us/azure/virtual-wan/about-virtual-hub-routing)

<u>[Azure Route Server](https://learn.microsoft.com/en-us/azure/route-server/overview)</u>- 

<u>[Azure Virtual WAN (vWAN)](https://learn.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about)</u>- 

<u>[Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/overview)</u>- 
## Single Region, Single VNet- Edge
## Single Region, Single VNet- NSD
## Single Region, Multi-VNet
## Multi-Region, Multi-VNet
## Azure VMware Solution (AVS)