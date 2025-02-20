---
title: Google Cloud Platform (GCP)
---

# Google Cloud Platform (GCP)

## Basic Concepts
<u>[Virtual Private Cloud (VPC)](https://cloud.google.com/vpc/docs/overview)</u>- VPCs within GCP are global/multi-region constructs (unlike AWS and Microsoft, with whom the VPC/VNet construct is confined to a specific region).  As a multi-region construct, one VPC setting to pay particular attention to is the “Dynamic routing mode”, which is a VPC wide setting that controls whether Google Cloud Routers (which in many scenarios will peer with SDWAN edges) will learn routes originating from all VPC regions (“Global” mode) or only routes originating from the local region (“Regional” mode).  

Another unique aspect of VPCs within GCP is that Google Cloud Engine (GCE) VM instances do not necessarily reside in a single VPC, providing the ability for a single edge to have interfaces in multiple VPCs.

<u>[VPC Peering](https://cloud.google.com/vpc/docs/vpc-peering)</u>- As with other IaaS providers, VPCs can be configured to allow traffic to transit between different VPCs.  It is however important to note that while peering VPCs does allow you to specify routes to be imported/exported to from/to the peer VPC, doing so will only populate those routes to the local VPC route table (but not to the cloud router that subsequently advertises those routes to SDWAN edges via BGP).  As such, in addition to VPC peering, in designs where SDWAN edges peer with the Google Cloud Router, the Google Cloud Router must also be configured with “Custom routes” for the subnets in the peer VPC that need to be advertised to/reachable from the SDWAN edge (this configuration is covered in step 6 of the [VMware SDWAN/Google NCC integration guide](https://docs.vmware.com/en/VMware-SD-WAN/services/VMware-SD-WAN-Google-Network-Connectivity-Center-Integration-Guide/GUID-B375C923-3CB9-4CFA-898E-4E2ED3DBF7B3.html)).

<u>[Private services access](https://cloud.google.com/vpc/docs/private-services-access)</u>-  Similar to VPC peering, Private services access provides cross-VPC connectivity, but with a focus on connecting to VPCs housing non-customer owned services (referred to as a "Service Producer VPC") via private connectivity within the GCP backbone.  Specific to SDWAN integration, this is the method used to connect customer owned VPCs (i.e. a transit VPC hosting virtual edges) to VPCs containing GCVE SDDCs.  There are some nuanced differences in [how private services access is configured](https://cloud.google.com/vpc/docs/configure-private-services-access) vs traditional VPC peering.  One such difference is that the IP address range to be routed across the the private services access connection must be allocated at the time of configuration, with the ability to [add additional address ranges](https://cloud.google.com/vpc/docs/configure-private-services-access#modify-ip-range) as needed to support GCVE environment with multiple/non-contiguous IP subnets.

<u>[Routing tables](https://cloud.google.com/vpc/docs/routes)</u>- Unlike AWS and Azure, GCP has no standalone routing table element that can be tied to disparate subnets/VPCs and instead has a single routing table for the entire VPC.  That said, manually adding routes to the route table, instance tags can be specified for a route which will cause it to only apply to instances with a matching tag.  This type of routing generally shouldn’t be needed to facilitate most designs, however it’s worth noting as pre-existing routes using instance tags could be difficult to troubleshoot.

<u>[Cloud VPN](https://cloud.google.com/network-connectivity/docs/vpn#docs)</u>-  Cloud VPN in GCP parlance refers to the ability to connect via traditional IPSec tunnels to Google hosted VPN Gateways, which in turn connect to workload VPCs.  Both VMware and Google best practices advise the use of BGP over IPSec for route exchange with a Google Cloud Router, and to provide additional resiliency across diverse tunnels.

<u>[Google Cloud Router/Network Connectivity Center (NCC)](https://cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview)</u>- In scenarios where dynamic routing with GCP is required, edges can peer via BGP to a Google Cloud router via the NCC hub.  This not only provides an efficient way to learn routes for GCP subnets that are not directly attached to the edge, but more importantly allows edges to dynamically populate the GCP routing table(s) with overlay prefixes.  As alluded to in the Design Constants section, care must be taken to ensure that quotas (specifically prefix limits) are not exceeded as routes received in excess of them can be ignored.

NCC leverages a hub and spoke nomenclature where an NCC hub is defined, and SDWAN edge peers are defined as spokes to the hub.  Architecturally, the NCC hub itself is just a logical management object wherein spokes (which in our case are the SDWAN edge VM instances) can be defined to configure the actual BGP sessions with the cloud router for a given region.  

The Cloud Router itself peers with the edge VM instances in a given region to advertise GCP subnet routes, and also populates routes learned from edges into the underlying GCP routing tables.  It provides control plane functionality only and does not physically sit in the data plane.  While Cloud Routers are local to a given region within a VPC, they can optionally advertise VPC subnets from other regions to leverage the GCP backbone as transit.  This can be controlled at the VPC level by specifying the Dynamic Routing Mode as “Regional” (so that only routes local to the Cloud Router’s region will be advertised) or “Global” (so that routes from all regions will be advertised):  
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
In this model workloads across all regions/subnets that are part of the workload VPC can be routed via the edge.  To establish overlay reachability, the LAN side subnet in which GE3 resides can be advertised as a connected route.  For all other subnets (either discrete subnets local to the region in the VPC or subnets in other regions) static routes and/or peering with a Google Cloud Router via the NCC hub are required.

## Single Region, Single VPC- NSD
NSD connectivity options for designs with a single workload VPC will leverage Google's Cloud VPN service to provide multiple redundant tunnels for connectivity directly into the workload VPC.  In all cases, Google and VMware best practices dictate leveraging BGP over IPsec to exchange routes with a Google Cloud Router as well as provide an extra layer of resiliency for tunnel failover.

Both options below leverage the [general Cloud VPN configuration process](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn) with slightly different configurations.  On the VMware side, both will use the "Generic IKEv2 Router" NSD type. 

For NSD via edge scenarios, leverage the "Generic IKEv2 Router" NSD type with the secondary VPN gateway option enabled to enable redundant VPN tunnels to the Google Cloud VPN gateways as shown.
<figure markdown>
  ![Image title](/images/gcp/nsd-single.png){ width="800" }
  <figcaption></figcaption>
</figure>

With NSD via gateway, maximum resiliency is realized by leveraging both primary and secondary Google Cloud VPN gateways, as well as selecting the "Redundant VeloCloud Cloud VPN" option in the NSD configuration to also build tunnels to GCP from a diverse VMware PoP, resulting in a total of four tunnels as shown.
<figure markdown>
  ![Image title](/images/gcp/nsdgw-single.png){ width="800" }
  <figcaption></figcaption>
</figure>


## Single Region, Multi-VPC
When there is an additional need to connect to multiple VPCs within a given region, VPC peering can be configured with workload VPCs as described in the basic concepts section.

For virtual VCEs running in GCP, the LAN interface will reside in a transit VPC (which in some cases may also contain other workloads) that will peer with multiple  workload VPCs (up to 25 as of this writing with quotas published [here](https://cloud.google.com/vpc/docs/quota#vpc-peering)).
<figure markdown>
  ![Image title](/images/gcp/multi-native-edge.png){ width="800" }
  <figcaption></figcaption>
</figure>

Similarly, in NSD based deployments the Google Cloud VPN gateway will be associated with a transit VPC peered with multiple workload VPCs.
<figure markdown>
  ![Image title](/images/gcp/multi-native-nsd.png){ width="800" }
  <figcaption></figcaption>
</figure>

In either case, take care to read/understand the VPC peering portion of the basic concepts section above to see how routes can be both populated to VPC routing tables as well as learned by the Cloud Router.

## Multi-Region, Multi-VPC
As mentioned in the Basic Concepts section of this page, the ability to connect to multiple regions via a single edge/cluster can be enabled by setting the "Dynamic Routing" setting for the VPC to "Global".  This by itself will only ensure that subnets from other regions within the Transit VPC are advertised from the Cloud Router, so the same steps to enable VPC peering and create the associated custom routes as described in the Basic Concepts section are still required as well.

While many of the same constructs used in the Single Region, Multi-VPC design still apply, leveraging edges in a subset of the reachable workload regions requires additional consideration in order to ensure optimal routing from branch edges to the GCP workload, and similarly to ensure that return traffic takes the same path to avoid asymmetric routing.  

Optimal routing is of course subjective, and may in some cases mean that the overlay (and associated performance benefits of DMPO) is leveraged to terminate on an edge that's as close to the destination workload region as possible and traverse the GCP backbone from there (which will also potentially provide transit cost savings) as depicted below:
<fix this>

In other cases the workload topology may prove too distributed to accomplish this without an undesirable amount of complexity/manual intervention (i.e. disparate route maps due to the fact that the GCP cloud routers will advertise workload VPC prefixes with identical BGP AS path length, metric, weight, etc. in all regions) dictating a design where all workload VPC bound traffic flow through the GCP based hub geographically closest to the branch, and traverse the GCP backbone from there (in some cases also avoid ISP peering performance issues) as depicted below:
<fix this too>

In either case, the use of hub ordering in the branch SDWAN edge profile will provide an optimal way to ensure that both ingress and egress traffic 

## Google Cloud VMware Engine (GCVE)
[GCVE](https://cloud.google.com/vmware-engine/docs) is a managed service offered by Google which, similar to VMC and AVS, utilizes VM networking based on NSX.  As such, integrating SDWAN connectivity is not a simple matter of deploying a virtual edge(es) directly into the GCVE workload domain (since the NSX Tier 1 routers that are L3 adjacent to workloads do not exchange routing information with anything other than NSX Tier 0 routers).  This being the case, connectivity options for GCVE with VMware SDWAN fall into one of the following two categories:

### NSD via edge or GW
Leveraging an NSD via edge or GW to GCP's Cloud VPN gateway(s) provides an easy way to connect to the VPC that houses a GCVE private cloud and/or .  It is recommended to leverage the "Generic IKEv2 Router Based" NSD templates (though GCP also supports IKEv1) as BGP over IPSec is a best practice advised by both Google and VMware.  This also creates a dependency on running 4.3.1 on the VCE/VCG that the NSD is built from.  

As with other VMware cloud offerings that leverage NSX, it is important to note that the 169.254.0.0/28 and 100.64.0.0/10 subnets are utilized for communications amongst SR/DR as well as T1 and T0 routing constructs so the use of addresses in those ranges elsewhere in the enterprise should be avoided to prevent routing conflicts.  This is particularly pertinent when creating Cloud VPN connections and associated BGP peering as GCP requires the use of link local address space (169.254.0.0/16) for internal tunnel/BGP neighbor IPs so care must be taken to avoid using addresses in the 169.254.0.0/28 (169.254.0.0-169.254.0.15) space.

NSD's via edge should leverage redundant VPN tunnels to GCP Cloud VPN gateways from all internet facing interfaces.  In the GCP Cloud VPN configuration, select the "Create a pair of VPN Tunnels" High Availability configuration as depicted in the single private cloud diagram below.

For NSD via gateway, check the "Redundant VeloCloud Cloud VPN" option to enable maximum resiliency by creating dual redundant tunnels from both a primary and secondary cloud gateway, resulting in a total of 4 tunnels as depicted in the multiple private cloud/multiple VPC option below.

For regions with a single private cloud, the GCP Cloud VPN can be configured directly in the VPC that houses the GCVE Private Cloud as pictured below.
<figure markdown>
  ![Image title](/images/gcp/single-gcve.png){ width="800" }
  <figcaption></figcaption>
</figure>
In deployments with either multiple GCVE private clouds, or, a combination of GCVE along with native GCP workloads, GCP Cloud VPN can also be configured in a transit VPC and leverage [GCP private services access](https://cloud.google.com/vpc/docs/configure-private-services-access#creating-connection) for connectivity to the GCVE VPC(s) and VPC peering with associated native GCP workload VPCs.
<figure markdown>
  ![Image title](/images/gcp/multi-gcve.png){ width="800" }
  <figcaption></figcaption>
</figure>

### Virtual edges deployed to a customer owned VPC
When additional throughput/tunnel scale is required beyond what can be supported via the NSD options, and/or when end to end DMPO is desired, virtual edges can be deployed into a transit VPC that connects via GCP private services access to the GCVE VPC(s).

Similar to the Multi-VPC options previously noted, a single edge cluster can be deployed with LAN interfaces in a transit VPC to facilitate connectivity to the underlying GCVE VPCs.  The only difference in this model is that rather than traditional VPC peering, GCP private services access from the transit VPC is leveraged as depicted below.
<figure markdown>
  ![Image title](/images/gcp/multi-gcve-edge.png){ width="800" }
  <figcaption></figcaption>
</figure>

As with other designs, the use of NCC to peer edges with the Google Cloud Router and facilitate clustering for resiliency/additional throughput is recommended.