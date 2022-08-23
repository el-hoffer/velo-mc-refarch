---
title: Design Constants
---

#	Design constants

##	HA/Clustering support
<u>HA (Standard or Enhanced)</u>- Currently not supported in any of the major cloud providers due to a universal lack of support for multicast (which is leveraged in certain HA communications between edges in an HA pair) in the IaaS provider underlay networks.

<u>Clustering</u>- As of release 4.3 (which added support for the BGP over IPSec functionality needed to support AWS Transit Gateway VPN attachments) clustering is supported for redundancy/scaling use cases across all three of the IaaS providers covered in this document.  

## Throughput/prefix limitations
While the standard procedures for configuring clustering and the associated LAN side peering still apply in IaaS environments, there are additional considerations to be made (especially where clustering is used) due to limitations within the IaaS providers:

<u>Route scale</u>:  As of this writing, learning up to 1,000 routes per cluster member is supported in both [AWS](https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-quotas.html) and [Azure](https://docs.microsoft.com/en-us/azure/route-server/route-server-faq), while [Google’s Cloud Router only supports 300](https://cloud.google.com/vpc/docs/quota) learned routes total per region.  These limitations, combined with our current lack of support for route summarization, can present a challenge due to the requirement of each cluster member to advertise the specific prefixes from spoke edges associated with that particular cluster member.  All three providers do provide mechanisms by which customers can request an increase to the quota, but VMware of course cannot speak to what those providers might approve.

<u>Throughput limitations</u>:  Each cluster member in an IaaS environment will have its own native throughput limitations as noted in our own datasheets (which are in all cases meaningfully lower than the throughputs supported in the higher end hardware platforms we typically use when deploying clusters on-prem).  In the case of Microsoft, those numbers will potentially be even lower due to interoperability with their underlying Mellanox drivers that have yet to be incorporated into GA code as of this writing and the 5.0.1.0 edge release (see Issue# 36133).  Additionally, the AWS Transit Gateway architecture (wherein the Transit Gateway sits in the data path as opposed to providing control plane only like Azure and GCP) also has its own inherent limitation of 1.25Gbps per VPN attachment.  This may be addressed in a future release by adding support for Transit Gateway Connect attachments (which require GRE support today) and/or layer 3 ECMP in our edge code.

## Multi-VPC/VNET connectivity constructs
All 3 of the covered IaaS providers support a mechanism by which connectivity from one VPC or VNET can be fanned out to multiple workload VPC/VNETs where actual applications reside.  The nuances of AWS’s Transit Gateway and Cloud WAN, Microsoft’s vWAN and Route Server, and Google’s Network Connectivity Center (NCC) are covered in the individual IaaS provider and VMware Cloud service variant sections, but in all cases the general intention with these offerings is to both provide a way for VMware SDWAN edges to learn prefixes that exist in the workload subnets in the IaaS providers, and conversely dynamically advertise branch prefixes to the IaaS provider so that individual workload subnet routing tables can be populated appropriately.  In all cases, this is accomplished via BGP peering between the VMware SDWAN edge and the IaaS provider and is required in any scenarios where clustering is needed.

## Dedicated private connectivity
Dedicated private connectivity to the IaaS providers, while named differently (AWS = [Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html), Azure = [ExpressRoute](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction), and GCP = [Dedicated Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview)), all provide a method for customers to cross connect to the IaaS providers infrastructure in a colocation facility via cross connects from customer leased space or carrier circuits (either dedicated/customer owned circuits or NNIs with WAN services providers).

While this is very common in IaaS environments where high throughput/low latency is required, there is a common routing behavior in which prefixes learned by the IaaS provider’s SDN fabric from the dedicated connection will always be preferred over the same prefix learned from a VPN gateway and/or multi-VPC/VNET construct (i.e. Transit Gateway) regardless of traditional routing metrics without many, if any, options to influence the path selection .  This tends to produce a desired behavior in scenarios where the connection is used to provide connectivity to an on-prem datacenter and is the preferred path.  On the other hand, in situations where the SD-WAN overlay should be preferred like when a private connection is used to connect an MPLS underlay to the IaaS fabric, it can create a challenge as depicted in the real-world scenario below:  
<figure markdown>
  ![Image title](/images/overview/dedicated-conn.png){ width="800" }
  <figcaption></figcaption>
</figure>
In this case, the prefixes in the IaaS environment are learned by the branch via both the overlay and underlay, and similarly the IaaS provider’s SDN fabric learns the branch prefix from both the overlay and the ExpressRoute to the underlay.  As a result, asymmetric routing will occur since the branch edge prefers the route learned via the overlay, while the IaaS provider’s SDN fabric prefers the branch prefix learned via the underlay.  Due to the lack (as of this writing at least) of IaaS provider functionality allowing users to influence path selection, the only feasible solution in this scenario to meet both the hybrid overlay/underlay, as well as the desire to prefer the SDWAN overlay was to restrict branch advertisements to the MPLS underlay to a summary prefix encompassing branch subnets, with more specific routes being redistributed into the SDWAN overlay.

Additional measures like inbound route filters on the virtual edge in the IaaS provider’s environment may also be required to avoid having the site become transit for MPLS underlay routes redistributed to the edge by the IaaS provider (since to date none of the covered IaaS providers have an option to selectively set communities that we can filter on and/or designate as the uplink community).


## Multi-region interconnectivity guidance
All 3 of the covered IaaS providers have a mechanism to enable an edge in a specific region to handle ingress/egress for workloads in a different region.  While IaaS providers will often tout the perceived performance benefits of using their private backbone for transit, it’s important to weigh the benefits of this type of design against the potential added complexity, associated transit costs, and the realities of physics as it pertains to performance.

<u>Complexity</u>-  On the surface, a design where edge clusters in a couple of regions handle ingress and egress for SDWAN traffic to all regions may seem like it would simplify the overall design by reducing the number of potential preferred exit points exposed to the overlay as depicted below. 
<figure markdown>
  ![Image title](/images/overview/multi-region-2edge.png){ width="800" }
  <figcaption>This</figcaption>
</figure>
<figure markdown>
  ![Image title](/images/overview/multi-region-multi-edge.png){ width="800" }
  <figcaption>Vs this</figcaption>
</figure>
Indeed, there are certainly smaller and or less geographically diverse customer designs where a deployment like this will be simple, however, in cases where branches and/or IaaS provider regions will be far from one another it’s important to note that you won’t have much control over routing metrics exposed to your edge clusters.  As such, it’s important to consider the implications of adding a hop in the SDWAN edge tied to a given region that may or may not be optimally positioned to provide transit to a given branch.  For example, a branch in Sydney accessing an app in Vancouver via an edge in London is obviously suboptimal but may appear to the overlay to be the best path based on the BGP metrics presented to a given cloud edge since most providers view inter-region transit as a single hop in the AS-path.  In these cases, options (if any) to provide an automated, deterministic way to enforce optimal routing are limited and vary by IaaS provider.  As such, the onus is typically on the SDWAN overlay to correctly select an appropriate VPN exit point that will not only avoid adding excessive latency via sub-optimal routing, but also not create asymmetric routing scenarios.

<u>Transit costs</u>- In multi-region designs there may be some degree of savings to be realized by limiting the SDWAN edge deployment to a subset of the deployed IaaS regions.  It is, however, also important to understand the additional transit costs associated with such a design to be weighed against any savings realized by reducing the number of subscriptions and VM instances consumed.  As illustrated in the AWS based example below, there are three different cost elements associated with sending traffic between regions in this relatively common scenario with Transit Gateways peered between two regions.  Based on February 2022 pricing, for traffic from an SDWAN edge in Region 1 to a workload in Region 2, there’d be a cost of $0.02/GB for traffic to traverse the Region 1 TGW, then a cost of $0.01-$0.147/GB for the data transfer between regions, followed by another $0.02/GB to traverse the Region 2 TGW, resulting in an end to end cost of $0.05-$0.187/GB (on top of the internet egress charges that will apply in any scenario).  As such, an understanding of the amount of inter-region traffic and region-specific pricing is required to determine what, if any, savings might be achieved by avoiding the edge subscriptions and EC2 instance costs in Region 2 in lieu of the added transit costs.
<figure markdown>
  ![Image title](/images/overview/transit.png){ width="800" }
  <figcaption></figcaption>
</figure>