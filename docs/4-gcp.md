---
title: Google Cloud Platform (GCP)
---

# Google Cloud Platform (GCP)

## Basic Concepts
<u>[Virtual Private Cloud (VPC)](https://cloud.google.com/vpc/docs/overview)</u>- VPCs within GCP are global/multi-region constructs (unlike AWS and Microsoft, with whom the VPC/VNet construct is confined to a specific region).  As a multi-region construct, one VPC setting to pay particular attention to is the “Dynamic routing mode”, which is a VPC wide setting that controls whether Google Cloud Routers (which in many scenarios will peer with SDWAN edges) will learn routes originating from all VPC regions (“Global” mode) or only routes originating from the local region (“Regional” mode).  

Another unique aspect of VPCs within GCP is that Google Cloud Engine (GCE) VM instances do not necessarily reside in a single VPC, providing the ability for a single edge to have interfaces in multiple VPCs.

<u>[VPC Peering](https://cloud.google.com/vpc/docs/vpc-peering)</u>- As with other IaaS providers, VPCs can be configured to allow traffic to transit between different VPCs.  It is however important to note that while doing so does allow you to specify routes to be imported/exported to from/to the peer VPC, doing so will only populate those routes to the local VPC route table (but not to the cloud router that subsequently advertises those routes to SDWAN edges via BGP).  As such, in addition to VPC peering, in designs where SDWAN edges peer with the Google Cloud Router, the Google Cloud Router must also be configured with “Custom routes” for the subnets in the peer VPC that need to be advertised to/reachable from the SDWAN edge (this configuration is covered in step 6 of the VMware SDWAN/Google NCC integration guide).

<u>[Routing tables](https://cloud.google.com/vpc/docs/routes)</u>- Unlike AWS and Azure, GCP has no standalone routing table element that can be tied to disparate subnets/VPCs and instead has a single routing table for the entire VPC.  That said, manually adding routes to the route table, instance tags can be specified for a route which will cause it to only apply to instances with a matching tag.  This type of routing generally shouldn’t be needed to facilitate most designs, however it’s worth noting as pre-existing routes using instance tags could be difficult to troubleshoot.

<u>[Google Cloud Router/Network Connectivity Center (NCC)](https://cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview)</u>- In scenarios where dynamic routing with GCP is required, edges can peer via BGP to a Google Cloud router via the NCC hub.  This not only provides an efficient way to learn routes for GCP subnets that are not directly attached to the edge, but more importantly allows edges to dynamically populate the GCP routing table(s) with overlay prefixes.  As alluded to in the Design Constants section, care must be taken to ensure that quotas (specifically prefix limits) are not exceeded as routes received in excess of them can be ignored.

NCC leverages a hub and spoke nomenclature where an NCC hub is defined, and SDWAN edge peers are defined as spokes to the hub.  Architecturally, the NCC hub itself is just a logical management object wherein spokes (which in our case are the SDWAN edge VM instances) can be defined to configure the actual BGP sessions with the cloud router.  

The Cloud Router itself peers with the edge VM instances to advertise GCP subnet routes, and also populates routes learned from edges into the underlying GCP routing tables.  It provides control plane functionality only and does not physically sit in the data plane.  While Cloud Routers are local to a given region within a VPC, they can optionally advertise VPC subnets from other regions to leverage the GCP backbone as transit.  This can be controlled at the VPC level by specifying the Dynamic Routing Mode as “Regional” (so that only routes local to the Cloud Router’s region will be advertised) or “Global” (so that routes from all regions will be advertised):  
<figure markdown>
  ![Image title](/images/gcp/routing-mode.png){ width="800" }
  <figcaption></figcaption>
</figure>
As of this writing, the NCC hub only supports a single VPC, however, the VPC that the LAN side of the edge connects to can be peered with multiple other VPCs (up to 25 as of this writing per [GCP’s VPC quota](https://cloud.google.com/vpc/docs/quota#vpc-peering)) as well.  It is, however, important to note that while peered VPCs are reachable from a dataplane perspective as soon as the peering connection is created, since the Cloud Router exists in a specific VPC, “Custom Routes” must be configured in order for peer VPC routes to be advertised to the SDWAN Edge(s):
<figure markdown>
  ![Image title](/images/gcp/custom-route.png){ width="800" }
  <figcaption></figcaption>
</figure>

## Single Region, Single VPC- Edge
Despite the need for only a single workload VPC in this scenario, an edge deployment in GCP leverages 3 separate VPCs; one for a dedicated management interface (GE1, used only for VM console output and not for production traffic), one for internet/WAN connectivity (GE2), and one for LAN side/workload connectivity (GE3) as depicted below.
<figure markdown>
  ![Image title](/images/gcp/single-region-single-vpc.png){ width="800" }
  <figcaption></figcaption>
</figure>
In this model workloads across all regions/subnets that are part of the workload VPC can be routed via the edge.  To establish overlay reachability, the LAN side subnet in which GE3 resides can be advertised as a connected route.  For all other subnets (either discrete subnets local to the region in the VPC or subnets in other regions) static routes and/or peering with a Cloud Router via the NCC hub are required.

## Single Region, Single VPC- NSD


## Single Region, Multi-VPC


## Multi-Region, Multi-VPC


## Google Cloud VMware Engine (GCVE)
