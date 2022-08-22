---
title: Cloud Web Security
---

# Web Security Policies

With distributed users and a multi cloud adoption strategy adopted by most organizations, Cloud security is critical. Just as we would expect to protect our data and apps on premises, we must ensure that our apps and services in the cloud are secure as well as the users who are accessing and connecting to them. VMware Cloud Web Security is a cloud hosted service that protects users and infrastructure accessing SaaS and Internet applications from a changing landscape of internal and external threats, offers visibility and control, and ensures compliance.

<figure markdown>
  ![Image title](imgs/2-cws/figure1.png){ width="800" }
  <figcaption></figcaption>
</figure>

VMware Cloud Web Security (CWS) is delivered through a global network of VMware SASE Points-of-Presence (PoP) to ensure that users located anywhere and connecting over any device have secure, consistent, and optimal access to applications. Cloud Web Security simplifies management of security services and helps IT tighten the security posture while balancing user productivity.

To use VMware Cloud Web Security, a user must first create, configure a Security Policy, and then apply the policy. Security policies are created and edited on the New UI of the VMware SD-WAN Orchestrator.

To configure a Cloud Web Security (CWS) policy, a user must have one of the following roles:

* An Operator with a superuser or standard roles.
* A Partner user with a superuser or standard role.
* A Customer user with a superuser, standard, or security admin role.

To configure a Security Policy, in the Orchestrator once in the *New Orchestrator UI*, from the SD-WAN drop down menu select *Cloud Web Security*. Once the CWS page appears click on Configure on the top right-hand corner and click on New Policy. The screen capture below shows how to add a new policy to CWS.

<figure markdown>
  ![Image title](imgs/2-cws/figure2.png){ width="800" }
  <figcaption></figcaption>
</figure>


In the upcoming sections we will investigate the components of VMware CWS and how those elements are incorporated within *CWS Security Policy*


### Cloud Web Security Architecture

The components making up CWS are as shown in the figure below:

<figure markdown>
  ![Image title](imgs/2-cws/figure3.png){ width="800" }
  <figcaption></figcaption>
</figure>



The components for CWS include the following:

* **URL Filtering** - The first line of defense for users accessing the Internet. URL Filtering can block known security threats and restrict access to Websites based on company policies.
* **Content Filtering** - Further reduce threat vectors by controlling which type of files are allowed for download or upload in the company network.
* **Anti-Malware** - Scans documents and files download from the Web browser, checking for known viruses or malware and preventing either from spreading to your users.
* **Sandboxing** - Zero-day vulnerabilities are threats not yet known to a vendor and therefore have no patch. Sandboxing reduces the zero-day risk by detonating the file in the cloud and analyzing it for malicious behavior. If malicious behavior is detected, the file is kept from the user and the company network.
* **CASB** - Enables IT teams to get visibility into sanctioned and unsanctioned SaaS applications. This helps IT determine what activities users can undertake when they access these applications. For example, can an employee login, upload and download files from Drop Box, or are summer interns only allowed to upload documents to Drop Box
* **DLP** - Data is a company’s most valuable asset. CWS ensures the most sensitive data, such as credit card numbers or intellectual property, is not exfiltrated from the environment.
* **SSL Inspection** - Most web applications are SSL encrypted. To gain visibility and control over the traffic going to the Internet, SSL Inspection exposes the transaction to bolster policy enforcement capabilities. Traffic that should not be decrypted (e.g., health and financial institutions) can simply be bypassed from decryption.

This section deep dives into the various components of CWS, their functionalities, features, implementation, and best practices for deployment.

#### Rules Processing Order

The components of the Cloud Web Security are applied through a sequential architecture. Each component is processed in order before handing over to the next component. If traffic is blocked or dropped, then no further processing happens. Below is a general level flow of each CWS component.

<figure markdown>
  ![Image title](imgs/2-cws/figure4.png){ width="800" }
  <figcaption></figcaption>
</figure>


!!! Remember
	If there is a bypass rule specifying a specific domain or content category for SSL Decryption, then only URL filtering is applied. This is due to the fact that CASB, DLP, Content Filtering, and Content Inspection need unencrypted access to enforce policy.

The PAC file allows enterprise users to use CWS as an explicit proxy. This may be useful in cases where customers do not have an SD-WAN overlay or Secure Access connection to the SASE PoP. By using a PAC file, the browser traffic can be redirected to CWS directly and CWS policies applied to control access to Internet and SaaS applications. This is currently on the roadmap to be implemented in 2HCY22.

The total percentage of Internet traffic that is encrypted is about 80-90% and increasing annually. This traffic needs to be SSL decrypted before being sent further through CWS for processing and hence that is the first step in the process. CASB and DLP rules are evaluated next before they are passed on to URL Filtering, Content Filtering, and Inspection.

The details of how each component handles traffic is further explained in the next few sections.

#### Cloud Web Security Policies

Policies are the method by which access rules are applied to internet traffic that passes through CWS. Only HTTP (80) and HTTPS (443) traffic are evaluated by CWS. Traffic sent to CWS via other ports (i.e., non-browser traffic like FTP, SSH, DNS, etc.) are passed through transparently but not evaluated nor processed.

After setting up a CWS Policy, the policy can then be applied to the following:
* Secure Access service

<figure markdown>
  ![Image title](imgs/2-cws/figure5.png){ width="800" }
  <figcaption></figcaption>
</figure>

* SD-WAN Edge internet backhaul traffic via Business Policy

<figure markdown>
  ![Image title](imgs/2-cws/figure6.png){ width="800" }
  <figcaption></figcaption>
</figure>


* Explicit proxy using a PAC file configuration (Roadmap in future release)

Each CWS policy is made up of a set of rules. Rules can be defined within each component of Cloud Web Security (SSL Decryption, CASB, DLP, URL Filtering, Content Filtering, and Inspection). When configuring a component rule there may be variations, but each rule follows a general framework as below:

* Source (User, Group)
* Destination (Category, Domain, IP Address)
* File content (file type and format)
* Action to be taken (Block, Allow, Log)

<figure markdown>
  ![Image title](imgs/2-cws/figure7.png){ width="800" }
  <figcaption></figcaption>
</figure>


### Enabling Identity Based Policy

Most organizations provide Internet access through a demilitarized zone (DMZ). The DMZ firewall is set up to allow Web traffic to the Internet and tracks activity by the source of requests by IP address. That IP address becomes a loose basis for establishing user identity. The assumption in this approach to security is that an IP address assigned to a device directly correlates to the person accessing the Internet. This view of identity is too simplistic and lacks the context required to articulate effective security policy. When next-generation firewalls (NGFW) were introduced, they featured integrations with services like Active Directory (AD) to identify the person, not just the machine, that was accessing Internet resources. This model worked great when the data center was the center of gravity for all network traffic.

Now most companies are highly distributed, their employees accessing Internet and SaaS applications from anywhere they can work. While services like AD are still very prevalent in enterprise networks, pulling Internet-bound traffic to a NGFW to check the user's identity and apply policy based on that identity is detrimental to user experience and, ultimately, workforce productivity. With an identity provider (IdP), organizations can integrate their existing user database with a platform designed to provide user verification services to CWS. When CWS is paired with an IdP, CWS policy enforcement becomes fully context-aware and provides precise logging of "who connected to what and when." This is functionally equivalent to the on- premises NGFW capability but is optimized for the highly distributed workforce of today's digital world.

<figure markdown>
  ![Image title](imgs/2-cws/figure8.png){ width="800" }
  <figcaption></figcaption>
</figure>


#### Integrating a SAML Provider

There are several IdPs on the market today, including VMware's Workspace ONE Access, that can be integrated with CWS. Integration with CWS is straightforward, and once it is set up, you will be able to create a CWS policy based on group and user attributes. To activate authentication services in CWS, it is as simple as navigating to the Authentication configuration page in the UI and filling in several fields. These fields include:

* SAML Server Internet Accessible? – The SAML server should be Internet accessible.
* SAML Provider - There are several defaults for popular IdPs such as Okta, ADFS, or Workspace ONE Access. CWS can also accommodate other types of IdPs but may require additional configuration from your provider to work correctly
* SAML 2.0 Endpoint - The URL of the endpoint that CWS will need to communicate with when establishing user identity.
* Service Identifier (Issuer) - An IdP is responsible for numerous services, not just CWS, and an individual service will need to be created in the IdP. This URL is used to reference that specific service.
* X.509 Certificate - An essential element for establishing a basis of trust with the IdP. This certificate is used to validate the identity of the IdP that CWS uses.

<figure markdown>
  ![Image title](imgs/2-cws/figure9.png){ width="800" }
  <figcaption></figcaption>
</figure>


#### Referencing Users and Groups in Policy

Once authentication services are enabled, the first time a user opens a Web browser and attempts to navigate to a site they will be prompted to enter credentials. After the user’s credentials are authenticated, the user is forwarded on to their original destination. All subsequent Web transactions are seamlessly authenticated by CWS, operating transparently to the user. This process allows you to tap into the user and group identity attributes found in the IdP.

While a detailed analysis of how an IdP operates is out of scope for this document, it is worth understanding how identity attributes (e.g., user and group) are populated in an IdP from an AD server. Most IdP vendors support automated import of group attributes using a connector that is installed on the AD server. This connector establishes a secure channel to the IdP and makes the AD attributes referenceable by the IdP.

In the examples shown below, you will see an IdP (Workspace ONE Access) with a user "**restricted.user**" assigned to a "**restricted**" group. When the user signs in, CWS is aware of both the username and group attributes assigned to that account. A policy can be crafted based on either of those values; the example below shows using the group attribute to prevent restricted users from accessing 'News and Media' content.

At the time of this writing, you must manually enter the username or group that is referenced in policy. This must match what is shown in the IdP. For example, the “**restricted**” group is typed exactly as it is shown in Workspace ONE Access.

<figure markdown>
  ![Image title](imgs/2-cws/figure10.png){ width="800" }
  <figcaption></figcaption>
</figure>


<figure markdown>
  ![Image title](imgs/2-cws/figure11.png){ width="800" }
  <figcaption></figcaption>
</figure>


<figure markdown>
  ![Image title](imgs/2-cws/figure12.png){ width="800" }
  <figcaption></figcaption>
</figure>


<figure markdown>
  ![Image title](imgs/2-cws/figure13.png){ width="800" }
  <figcaption></figcaption>
</figure>


#### Best Practices for Identity Based Enforcement

* CWS URL Filtering, Content Filtering, Content Inspection, CASB, and DLP are based on specifying the source of users and groups. If no user or group is selected, then it applies to all users. When creating policy always consider users as the most specific, groups the second most specific, and “all users and groups” as the least specific way to identify traffic sources, including anonymous users.
* Most of your identity-based policy should be built around groups. Groups are typically organized by common access criteria and can give you the flexibility needed to tailor CWS policy to different teams such as Talent Acquisition, Engineering, Accounting, and so on.
* Only use the individual user(s) when there is a clear need to do so. For example, a person on the Engineering team requires access to otherwise blocked social media to test their latest integration with that platform’s API.
* For agreed upon content categories and file types that should be blocked, regardless of the person’s role in the organization, it is appropriate to use the “all users and groups” setting.
* The goal of identity-based policy is to make location independent enforcement consistent no matter where your organization’s employees are accessing resources from.

### Gaining Visibility with SSL Inspection

End-to-end encryption is the de facto standard for communication on the Internet. Percentages are a moving target, but it is safe to say that nearly all your transactions on the Internet to various applications are encrypted. The Transport Layer Security (TLS) protocol provides users with confidentiality and integrity as they transact on the Web. Encryption provides confidentiality by ensuring no one can eavesdrop on a conversation between a client and application server. Message authentication ensures integrity by validating the communicating endpoints are indeed who they claim to be.

<figure markdown>
  ![Image title](imgs/2-cws/figure14.png){ width="800" }
  <figcaption></figcaption>
</figure>

Another critical component to establishing identity is the public key infrastructure (PKI). The PKI is used to distribute and validate certificates to servers and applications on the Web. Clients use these certificates to establish trusted and secure connections to the service they are accessing. While TLS and PKI are great at protecting legitimate traffic, the two can create security blind spots for organizations providing Internet services to their employees.

<figure markdown>
  ![Image title](imgs/2-cws/figure15.png){ width="800" }
  <figcaption></figcaption>
</figure>


From preventing procrastination to blocking threats on the Internet, enterprise security teams require visibility into the encrypted traffic traversing their network. In days past, and even today, network and security teams inserted purpose-built appliances in their network to intercept and decrypt TLS communications. This process is cumbersome and costly to an organization. With CWS, the ability to inspect TLS traffic is straightforward and requires minimal configuration to enable companywide. This section will examine the "SSL Inspection" feature available in CWS and explore it to a depth that will allow you to implement this capability successfully in your organization.

<figure markdown>
  ![Image title](imgs/2-cws/figure16.png){ width="800" }
  <figcaption></figcaption>
</figure>


!!! Remember
	Secure Socket Layer (SSL) and TLS are often used interchangeably. In the context of interception and decryption technologies this is okay. However, SSL and TLS are significantly different, with the former having been deprecated.

#### SSL Termination Certificate

For CWS to provide a seamless end-user experience, the CWS root certificate must be distributed to all endpoints connecting to the Internet via CWS. Recall that it is up to the client to verify the server's identity by checking its certificate against its signer(s) in the certificate chain. When SSL Inspection is enabled, CWS will present itself as the server to the client and sign the server's certificate with the CWS CA certificate. If the CWS CA certificate is not present in the client's trusted root store , the user will encounter an error, and there will be an adverse impact on the user's quality of experience.

If the process of distributing the CWS CA certificate to clients sounds intimidating, it is not. To retrieve the certificate, navigate to your VMWare Orchestrator and follow these steps:

* Select 'Cloud Web Security'
* Click on 'SSL Termination'
* Click on 'Download Certificate'

<figure markdown>
  ![Image title](imgs/2-cws/figure17.png){ width="800" }
  <figcaption></figcaption>
</figure>


A dialog box will appear to provide you with an opportunity to name the certificate file and select the location you wish to save it.

<figure markdown>
  ![Image title](imgs/2-cws/figure18.png){ width="800" }
  <figcaption></figcaption>
</figure>



After saving the CWS CA certificate, inspect the certificate to confirm the thumbprint matches what is displayed on the VMware Orchestrator page. The example below is from a macOS device. Please note the area outlined in red has had the values removed. In your production instance, expect to see a 40-character alpha-numeric value for the SHA-1 fingerprint and a 64-character alpha-numeric value for the SHA-256 fingerprint. This check is critical as it ensures the CA certificate you have downloaded is not corrupt and will work with the SSL Inspection feature.

<figure markdown>
  ![Image title](imgs/2-cws/figure19.png){ width="800" }
  <figcaption></figcaption>
</figure>

Typically, you would want to test your SSL Inspection settings on a handful of machines before rolling this feature out to your entire production network. It is easiest to manually install the download certificate into your system's root certificate store for testing purposes. To do this, consult your operating system's manual. Below is an example from a macOS device.

<figure markdown>
  ![Image title](imgs/2-cws/figure20.png){ width="800" }
  <figcaption></figcaption>
</figure>

Once you are ready to implement the SSL Inspection feature in your production network, you will need to consider a distribution method for adding the CA certificate to the devices in your environment. Manual installation, typically done for localized testing, is not realistic in production. Instead, consider using a mobile device manager (MDM) or Windows System Center Configuration Manager (SCCM). These technologies will give you the ability to deploy tens of thousands of devices with minimal effort. It is worth noting that both MDM and AD are often under another technology group in large enterprise networks. Suppose you are leading this discussion from a network or security group. In that case, it is highly recommended you engage your end-user computing (EUC) early on in this process so they can meet your delivery timeline.

<figure markdown>
  ![Image title](imgs/2-cws/figure21.png){ width="800" }
  <figcaption></figcaption>
</figure>


#### SSL Inspection Policy

Setting up an SSL Inspection policy is relatively straightforward. By default, CWS will inspect all Web traffic destined to the Internet. The reasoning is that nearly all Web traffic is encrypted, and without intercepting that traffic, you cannot apply a meaningful security policy. By default, there is no need for you to configure inspection; that is done for you. However, you will need to configure your bypass rules. When you bypass SSL inspection, the reason is specific. There are protected categories of traffic such as banking or healthcare and applications that break when intercepted, such as WebSocket or applications with certificate pinning.

There is no such thing as a universal bypass list that applies to all enterprises. What is important to inspect and what is essential to leave private is a matter of agency. Instead, this section will focus on the methodology for creating effective bypass rules to apply to your SSL Inspection policy. If your organization already has a policy, you can focus more on implementing it in CWS.

<u>Bypass Rules with Content Categories</u>

A good rule to adhere to is to always start with the existing content categories in CWS. At the time of this writing, CWS has a total of 84 pre-defined content categories. When a content category is selected, it applies to all websites that fall under that category. For example, selecting 'News and Media' would apply to Forbes.com, Cnn.com, Foxnews.com, Npr.org, and so on. You can also choose multiple destination categories in a single rule, reducing configuration overhead.

<figure markdown>
  ![Image title](imgs/2-cws/figure22.png){ width="800" }
  <figcaption></figcaption>
</figure>


You are given four additional fields to complete on the next screen of the workflow

* Rule Name - Names are important. That is why so many organizations have a naming convention. Follow your organization's convention for naming a security rule. On the off chance, your company has no established guidance on this; consider calling it the action taken. The name 'Bypass News and Media' is chosen in the example below. Clear and concise over esoteric and verbose.
* Tags - Tags provide an option to add searchable metadata to a rule. This becomes significantly more useful as your configuration grows. Consider the example where the tag 'ssl-bypass' is set. As more bypass rules are created, applying this tag will allow you to identify all rules that bypass SSL inspection quickly.
* Reason - When it comes to SSL Inspection, an organization must be explicit in its intent. Company policy should dictate what should and should not be decrypted on the enterprise network. And that company policy should align with compliance relevant to its industry. In this example, a specific company rule is referenced. This clarifies to anyone auditing the bypass rule why the rule exists.
* Position - Rule processing occurs from top to bottom. As with any rules-based system, such as an NGFW or ACL, place your more specific rules at the top of the list and your least specific rules at the bottom. This example shows placing the content category rule at the bottom since it has a broad application

<figure markdown>
  ![Image title](imgs/2-cws/figure23.png){ width="800" }
  <figcaption></figcaption>
</figure>


!!! Remember
	If you are ever unsure how a website is categorized tools like https://www.brightcloud.com/tools/url-ip-lookup.php can be used to determine a URL’s or IP’s category.

<u>Bypass Rules with Destinations</u>

Content categories are meant to reduce configuration overhead for organizations. These pre-defined categories classify millions of destinations on the Internet better than any person could. But what happens when your organization has a policy or technical need to bypass SSL decryption for specific websites or SaaS applications? That is where Destination Types come into play. The types included are IP Address, IP Range, IP CIDR, and Host/Domain name. And these features provide your organization with the flexibility it needs to address policy or technical problems. This section will explore creating a bypass rule using the Host/Domain name option for Zoom's popular video-conferencing application.

!!! Remember
	VMware publishes a list of recommended domains for bypass on their [VMware Cloud Web Security Product Documentation page](https://docs.vmware.com/en/VMware-Cloud-Web-Security/5.0/VMware-Cloud-Web-Security-Configuration-Guide/GUID-23A23EE9-40E0-4F24-99AC-9D59A4E1C0F5.html)

Most SaaS providers, like Zoom, publish network firewall and proxy server settings for their applications. These support documents detail out what engineers need to allow through their perimeter so the application will function. And finding this information as easy typing in "Zoom SSL bypass" into your favorite search engine. There is a good chance it is the first or second result returned by the search engine. The vendor documentation can vary in terminology, but you generally need to search for recommended proxy or SSL settings. In the screenshot below, you can see that Zoom instructs you to exempt "zoom.us and *.zoom.us from proxy or SSL inspection."

<figure markdown>
  ![Image title](imgs/2-cws/figure24.png){ width="800" }
  <figcaption></figcaption>
</figure>

It is essential to recognize that CWS will interpret zoom.us as *.zoom.us. CWS automatically adds the wildcard to the parent domain to apply the rule to all subdomains. Therefore, in CWS, you will only need to add "zoom.us" for your bypass rule.

<figure markdown>
  ![Image title](imgs/2-cws/figure25.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure26.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure27.png){ width="800" }
  <figcaption></figcaption>
</figure>

Whether it is one entry or 40, creating an SSL Inspection is similar for each application. Alternatively, endpoints can be specified using their IPv4 address(es). However, using domains is preferred as IP addresses associated with an application may change over time. Still, there can be times it is necessary to use an IPv4 address instead of a domain.

The most challenging part of SSL Inspection is ensuring your configuration aligns with your company's information security and regulatory compliance policies. Then making sure the applications you support on your network are not adversely impacted. This task can easily be accomplished with a little bit of research and simple tools to extract the data you need.

<figure markdown>
  ![Image title](imgs/2-cws/figure28.png){ width="800" }
  <figcaption></figcaption>
</figure>

#### A QUIC Note

This SSL Inspection section focuses heavily on TLS over TCP, the dominant traffic type on the Web that transports HTTP. Techniques for intercepting this kind of traffic are well established and available in most security products today. However, the IETF recently standardized QUIC, an entirely new protocol for transporting HTTP/3 and not supported by security solutions today. QUIC represents a new paradigm in application transport technology, most notably its use of UDP and immediate use of encryption. The newness of QUIC (as a standardized protocol) and its unique approach to transport security prevent it from being intercepted and inspected.

While this is not to say this will always be the case, it is to recognize that all security vendors are faced with the same challenge today. However, applications that support QUIC are designed to fall back to a TCP/TLS should QUIC negotiation fail. At the time of this writing, the recommendation is first to determine if traffic destined to UDP port 443 is needed for any reason within your organization. Some applications use UDP/443 on the Internet, but it is most likely that your organization does not. After confirming UDP/443 is not required on the Internet, block it. In the VMware SASE solution, an ideal place to do this is on the SD-WAN Edge Firewall. These options will force the application to fall back to TCP/TLS, and your CWS SSL Inspection policy will work.

Finally, if QUIC is not a concern for your organization, you can still forward it to CWS. All QUIC traffic will bypass the proxy and forward directly to the Internet, resulting in zero impact to your end-users.

**Best Practices for SSL Inspection**

* Always keep in mind the CWS approach to SSL Inspection: Decrypt by default, bypass by exception. This could run counter to how your organization may implement SSL inspection currently.
* Use pre-defined content categories whenever possible. The most likely categories to be used are Financial Services and Health & Medicine to protect PII and consumer health information governed by HIPAA.
* When granular matching criteria is needed rely on domains over IP addresses. Domains are less likely to change than an IP or range of addresses.
* Take inventory of installed applications on end user systems (Orchestrator provides a complete list of Apps it has identified from the Edges) and consult the vendors’ product documentation to see if they tunnel non-Web traffic over HTTPS. If so, bypass entries will need to be created to avoid breaking the local application. A prime example is Outlook.
* Test your SSL Inspection policy before rolling it out to production. A good option is to use a corporate issued device, place it behind CWS, and validate applications and Web browsing works as expected. For anything that breaks, consult the vendor’s online documentation to determine what will need to be bypassed.
* Give your rules meaningful names, supply a reason for a given rule that ties back to your organization’s justification for it, and consider using tags to group together rules across policy that are used for achieving a specific outcome.


### Controlling Web Traffic with URL Filtering


Today, few companies allow unfettered Web access for their employees, contractors, and guests. Company reasoning may vary, but it is primarily due to the proliferation of bad actors that use the Web as a vector for compromise and inappropriate content that violates company policy. Businesses have relied on dedicated hardware appliances such as proxy servers and next-generation firewalls (NGFW) to filter content and URLs. Those organizations with experience with the legacy security stack will find CWS' URL Filtering capability familiar and should be able to map their existing policy into CWS with relative ease. And for those new to URL Filtering of any kind, the guidance below will prove invaluable for getting started.

<figure markdown>
  ![Image title](imgs/2-cws/figure29.png){ width="800" }
  <figcaption></figcaption>
</figure>

#### URL Filtering by the Numbers


Before you begin, know that URL Filtering in CWS is permissive by default, allowing website categories, threat categories, and content categories. CWS requires explicit configuration before blocking any traffic types or URLs can occur.

<figure markdown>
  ![Image title](imgs/2-cws/figure30.png){ width="800" }
  <figcaption></figcaption>
</figure>

Before adding rules, consider the approach you or your organization prefers to take on Web filtering. Is a "block by default, permit by exception" the guiding principle? Or does "block specifics, permit everything else" make the most sense? There are tradeoffs with either, one favoring tighter security over greater operational complexity and the other with less strict security but easier to manage. Regardless of methodology, the first rule worth considering is to Block All Threat Categories no matter who the user may be.

<figure markdown>
  ![Image title](imgs/2-cws/figure31.png){ width="800" }
  <figcaption></figcaption>
</figure>

Threat categories tap into intelligence feeds that classify malicious or harmful websites into one of the 11 pre-defined categories. While the possibility does exist for some legitimate traffic to get blocked by this filter, the benefits far outweigh the occasional block and can easily be corrected by adding an exclusion for that website in another rule.

As with threat categories, your organization should have an agreed-upon list of content categories that are not permissible to access irrespective of the employee. The list of categories will vary from company to company and should map back to specific company policy. An example is shown below for demonstration purposes.

<figure markdown>
  ![Image title](imgs/2-cws/figure32.png)
  <figcaption></figcaption>
</figure>

By now, you have removed most Web threats from the Internet for your organization and have enforced content access policy company wide. The following section will discuss more granular use of URL filtering.

#### Granular Control with URL Filtering

While the two preceding rules are enough to block thousands, if not hundreds of thousands of Websites, there will come a time when more refined access rules are required. The URL Lookup tool for validating CWS’s websites to their category mapping is: https://www.brightcloud.com/tools/url-ip-lookup.php. If a website runs a vulnerable service, such as PHP, it will trigger the Block All Threat Categories policy.

<figure markdown>
  ![Image title](imgs/2-cws/figure33.png)
  <figcaption></figcaption>
</figure>

However, this tool is helpful to security teams looking to update their CWS URL Filtering policy. Since only the security team needs access, the following rule example shows how to permit access to brightcloud.com only for security team users.

1. Based On: Domain
2. Select Source and Destination
	3. Source
		4.  Specify Groups: security
	5. Destinations
		6. Specify Domains: brightcloud.com
7. Action: Allow
8. Name, Reasons and Tags
	9. Rule Name: Exemptions for Security Team
	10. Position: Top of List


<figure markdown>
  ![Image title](imgs/2-cws/figure34.png){ width="800" }
  <figcaption></figcaption>
</figure>

The rule is simple but powerful. The threat category block is still in effect companywide with the exclusion of the organization's security team. By combining user groups with specific domains, you can create more granular rules that address the grey areas you often encounter in security. Finally, pay close attention to the rule's name: Exemptions for Security Team. There is no doubt this will not be the last required exemption for the group, and this rule can be updated to include more domain exclusions as they come up. The goal here is to be both granular and flexible.

#### Best Practices

* Start with defining threat and content category block rules that are applicable to the entire company. These rules will apply to all users and groups within your organization.
* When defining granular policy, use the group(s) attribute to avoid allowing unnecessary access to those in the company the rule is not applicable to.
* Consider naming rules based on the group and type. For example, “Exemptions for Security Team” becomes a container for additional domains or categories that can be allowed without having to create yet another rule.
* Limit the creation of user specific rules, both in number and the time they are needed. Ensure these rules are audited regularly to avoid being left in place when no longer required for use.
* If a content category is blocking a website that should otherwise be accessible by the entire company, create a top- level rule that excludes the required domain(s) for all users and groups. Continue to add or subtract domains to this rule as needed.
* When operating multiple segments (e.g., Corporate, IoT) create separate CWS policies and tailor the approach. The above best practices reflect a corporate user environment for human interaction. For IoT, the “block all, permit by exception only” approach may be more desirable from a security perspective since identity cannot be assigned to devices. 			

#### Restricting File Distribution with Content Filtering

VMware CWS can adhere to organizational policies to restrict users from uploading and downloading certain files to SaaS
applications or to any other site on the internet. This preventative security measure reduces the attack surface by allowing
only required types of file content to and from the internet. An organization may not want any executable files to be
downloaded on company devices; or it may limit uploading MS Projects to the Internet. The figure below is a visual
representation of restricting file distribution over the cloud.

<figure markdown>
  ![Image title](imgs/2-cws/figure35.png){ width="800" }
  <figcaption></figcaption>
</figure>


Note that Content Filtering on CWS is permissive by default allowing all files to be uploaded and downloaded from the Internet. The downloads undergo a virus scan, while the uploads are allowed without inspection. CWS requires explicit
policies to be configured for allowing and blocking upload and download of files. This can be achieved through selection of specific categories or exact file type; and identifying the source and destination to where the files need to be blocked from. At this time CWS default rules cannot be edited. As previously discussed, an organization needs to carefully evaluate its
security policy and operational requirements to determine if the default allowed rule fits, and now explicit block rules can be identified, or if block all rules need to be created and then explicit allowed rules are added.

In the following example we create explicit rules for blocking download of all scripts and executable files, except for Mac executables. Under the CWS tab, in the selected *Security Policy* screen select *Content Filtering*. The first step is to identify the *Transfer Type*, File Type, followed *source* and *destination*. The highlighted executables are selected for the transfer action of Download.


<figure markdown>
  ![Image title](imgs/2-cws/figure36.png){ width="800" }
  <figcaption></figcaption>
</figure>


<figure markdown>
  ![Image title](imgs/2-cws/figure37.png){ width="800" }
  <figcaption></figcaption>
</figure>

The next steps are to apply the restriction to the Action of BLOCK or ALLOW. followed by naming the policy. This is as shown in the screen captures from CWS below, where the download of the file type of executables is blocked.

<figure markdown>
  ![Image title](imgs/2-cws/figure38.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure39.png){ width="800" }
  <figcaption></figcaption>
</figure>

The policies should have meaningful names determined in the Designing and Planning phases by the Engineering team and following a naming convention that works with the company’s Corporate Information Security Office and other stakeholders. Upon configuration of the explicit download and upload rules, the CWS Content Filtering page looks like the screen capture below


<figure markdown>
  ![Image title](imgs/2-cws/figure40.png){ width="800" }
  <figcaption></figcaption>
</figure>

The policy should then be published by clicking on the *PUBLISH* button on the upper right-hand corner. Once published the user is ready to apply the security policy. It is important to identify in the design and planning phase identification of explicit “ALLOW” rules for specific teams like the IT department or Security teams managing remote users and their devices. These rules would be placed “Top of the List.” CWS Content Filtering allows security administrators to efficiently scale security policies limiting users from uploading, downloading when accessing a certain Websites over the internet.

### Preventing Viruses, Malware and Zero Days with Content Inspection

VMware CWS offers anti-virus, anti–malware and threat protection from Day 0 attacks with the Content Inspection feature built into CWS. CWS utilizes file hash check, antivirus scan and sandboxing for protecting against known and unknown threats on all scanned files entering and exiting the user towards any cloud. The threat protection check is as illustrated by the figure below.

<figure markdown>
  ![Image title](imgs/2-cws/figure41.png){ width="800" }
  <figcaption></figcaption>
</figure>

* When a file is received by CWS, it is sent for a File Hash Check (FHC). A file hash is a unique value and is compared against results from more than 50 AV engines. The result of a hash check can be clean, malicious, or unknown. If clean, the file is allowed onto the network. If malicious, the file is dropped. If unknown, the file will be either dropped or sent to the Anti-Virus Scan (AVS), depending on which options were selected.
* If a match is not found, the file contents are scanned in the AV engine with malware signatures/pattern to see if the file has malware. If no known malware signatures matches are found the file is forward for sandboxing.
* Sandboxing is a remote and isolated environment, where the files can be statically or dynamically analyzed. Static analysis analyzes the files for libraries and imported functions by scanning the files code for strings and linking methods utilized. Dynamic analysis is conducted by executing the files functionality and observing what impact it
has within that contained environment and if indeed the file is infected based on its behavior. Static analysis is faster; however, it may not be as proactive as dynamic analysis which is slower.
* A third approach to follow is a blend of static and dynamic analysis provisioning acceptable time frames and high- end threat protection. Sandboxing provides a method to identify Day 0 malware or other threats by actively catching suspicious activities preventing applications or operating systems from getting infected. CWS sandbox file limit is 100 MB by default, however there are settings available to allow for files larger than 100MB to be contained. Today sandboxing is done in a remote environment outside the SASE PoP (UK, USA, Japan and Germany).

#### Content Inspection Engine

The CWS Content Inspection Engine marks all traffic as clean by default. Today this setting cannot be changed, and policies have to be created to either Inspect the traffic, mark it as Infected or Mark as Clean. To configure this, select Content Inspection from the Security Policy screen under the CWS tab. Click on ADD RULE to select Transfer Type and select File Types on which the Content Inspection actions will be performed. In this example, certain Word processor file types are selected for the transfer type of download. This is as shown by the screen capture below

<figure markdown>
  ![Image title](imgs/2-cws/figure42.png){ width="800" }
  <figcaption></figcaption>
</figure>

In the next page of Select Source and Destination, all users and groups can be selected, or specific users and groups can be specified on which the rule will apply. In this example specific users and groups are specified with specific destination Domains including Google, DropBox, and domains in the category of *Computer and Internet Info* and *Computer and Internet Security*.

<figure markdown>
  ![Image title](imgs/2-cws/figure43.png){ width="800" }
  <figcaption></figcaption>
</figure>

The next step is to apply the Action on the source and destination for those specific users and groups. There are three actions that can be applied on transfer types. These are:

* Mark as Clean – Files (downloaded or uploaded based on transfer type) are automatically permitted onto the network
* Mark as Infected – Files are automatically blocked from the network from specified source and to specified destination
* Inspect – Matching files are inspected with three different checks previously explained including File Hash Check (FHS), Anti-virus Scan (AVS) and sandboxing. If the files fail the check, they are dropped otherwise they are permitted to the network.

The screen capture below shows the application of the Action.

<figure markdown>
  ![Image title](imgs/2-cws/figure44.png){ width="800" }
  <figcaption></figcaption>
</figure>


In the next *Name, Reasons and Tags* screen the user can configure the name of the policy to match its functionality. For the *Position* field, where the policy should be placed can be selected. In this example we place the policy *Top of the List* as by default all traffic is marked as clean. The user can then create another rule, create another Security Policy or click *Finish* and *Publish* this policy.


#### Protecting the Applications Outside Your Data Center with CASB

Software as a Service (SaaS) has redefined how organizations consume applications. Gone are the days when popular office application software came on a CD-ROM and was installed locally on a computer. And access to shared application resources was neatly tucked away behind a firewall. With the move to SaaS, applications, and the data they create are hosted in the vendors' cloud. The convenience and efficiencies that SaaS created accelerated adoption in the market and resulted in control, access, and enforcement no longer being in direct control of the enterprise. With the proliferation of SaaS applicat ions and the dissolution of security boundaries, the industry responded with Cloud Access Security Broker (CASB) solutions to return enterprise control, access, and enforcement. CWS provides an integrated and in-line CASB service that is easy to configure and helps businesses get control of their SaaS applications.


<figure markdown>
  ![Image title](imgs/2-cws/figure45.png){ width="800" }
  <figcaption></figcaption>
</figure>

#### Understanding SaaS Consumption

Before configuring any CASB policies for your organization, ask yourself, "Are the SaaS applications my company uses clearly defined? And are the policies that govern their use clear in technical restrictions I will need to implement?" If the answer to both questions is yes, you are ready to create policy, but if your situation is like most enterprises, you may not be aware of what is being accessed on your network. Assuming you have crafted a sound URL Filtering policy, and have it published, it could be best to give CWS an opportunity to monitor your company's SaaS application consumption to narrow down the scope of what you will need to allow, block, or provide granular restricted access to. This can be done by clicking on the **CASB option under policies**. You can learn more about each discovered SaaS application by clicking on its name.

<figure markdown>
  ![Image title](imgs/2-cws/figure46.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure47.png){ width="800" }
  <figcaption></figcaption>
</figure>


Once you have a firm grasp of the applications on your network, you are ready to begin creating your CASB policy. If you look closely at the image above, you will see that 280 domains have been associated with Facebook. Unlike URL Filtering policy, CASB performs application identification and has a much more complete view of all the domains associated with a given SaaS application. If you are ever unsure of which domains you should block, CASB is an effective tool, provided the application you are attempting to block is 1 of the 1,000+ available.


!!! Remember
	The best way to view the relationship between CASB and URL Filtering is in their granularity. To borrow painting terminology, CASB is best used for fine strokes and URL Filtering best suited to broad strokes.


#### Sanctioned and Unsanctioned Applications

This section will focus solely on the controls available in CASB and a good way to approach implementing security settings for your SaaS applications. Assume the following scenario:

1. An organization decided to give CWS a 30-day timeframe to identify the SaaS applications in use on their network.
2. After identifying the SaaS applications, the company discovered numerous apps being used that are not sanctioned by the company.
3. As a result of their analysis, security will configure CASB controls to permit users full access to O365 and restricted access to Facebook (browse and login only).
4. All other unsanctioned applications will be blocked using CASB controls. The determination to use CASB and not URL Filtering for blocking stems from the richness of the CASB app-id capability.
5. Security will continue to monitor SaaS consumption and ensure as more apps become sanctioned, their access restrictions are clearly defined and a new rule governing that specific application will be created.

For this scenario, and for the sake of brevity, the apps that have been deemed sanctioned are Facebook and O365. All other applications have been declared unsanctioned and, therefore, will be blocked from access on the network. Below is sample output from configuring the three rules described in the above scenario.

<figure markdown>
  ![Image title](imgs/2-cws/figure48.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure49.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure50.png){ width="800" }
  <figcaption></figcaption>
</figure>


The power to control SaaS applications with in-line CASB far exceeds security controls found in other technologies. While the preceding example is by no means complete, it serves to illustrate how CASB can be used to tightly control SaaS consumption on your network.


#### Best Practices

* Take inventory of the SaaS applications your organization uses. If you do not have a clear list, CWS will help you find the applications that users are accessing. Once you have a clear picture of SaaS usage on your network, identify those apps that are sanctioned and unsanctioned for use on the network.
* Consider the user groups as well. For example, a marketing or public relations team might need full access to social media (e.g., Facebook, LinkedIn, Twitter, etc.) whereas the technology and accounting departments are allowed to browse to and login to social media but are prevented from posting anything while on the work network.
* Create rules specific to an application. While it is possible to combine different apps into a single rule (e.g., PowerPoint and Box) you will see application control options that are common and different between apps. CWS will let you know which control applies to which, but this can quickly become cumbersome when dozens of different apps are referenced in the same rule.
* For all other SaaS apps that are not allowed on your network, should you choose to block, make use of CASBs pre- defined apps and create a block all rule. This should be placed at the bottom of the rules and as evaluation occurs from top down. A rule like this can provide a higher degree of fidelity for blocking than URL Filtering in the event an app uses multiple and different domains.


### Sending Traffic to CWS with VMware SD-WAN and Secure Access

Many organizations perform security processing for Web traffic in less than a handful of locations despite supporting a geographically dispersed user base. Redirecting remote access VPN (RAVPN) or legacy WAN traffic to the company's security stack for Internet access diminishes the end-user experience but is tolerated out of necessity for security. Additionally, companies need to enforce different policies for different types of traffic depending on what originated it (e.g., a corporate user, a network device, or an IoT endpoint). To achieve this level of granularity, engineers are forced to build and maintain complex hop-by-hop configurations on all devices in the forwarding path to ensure this works as expected. Suboptimal Internet access, processing delay from security products, and technical overhead make this approach unsustainable.

<figure markdown>
  ![Image title](imgs/2-cws/figure51.png){ width="800" }
  <figcaption></figcaption>
</figure>


<figure markdown>
  ![Image title](imgs/2-cws/figure52.png){ width="800" }
  <figcaption></figcaption>
</figure>


Every VMware SASE PoP consolidates access and security services within a geographically proximate location to its remote users and devices. Redirection of VMware SD-WAN and Secure Access (SA) traffic destined to the Internet to CWS is as simple as referencing the CWS policy in Business Policy or the SA configuration profile. Additionally, CWS is segment aware, meaning that tailored policy for corporate, network devices, and IoT endpoints can be crafted and easily deployed to those groups. This approach to security eliminates the latency impact caused by backhauling, simplifies redirecting Internet traffic to CWS, and gives greater flexibility when segmenting your critical resources without having to introduce a bunch of complexity to achieve it.


<figure markdown>
  ![Image title](imgs/2-cws/figure53.png){ width="800" }
  <figcaption></figcaption>
</figure>

#### Implementing CWS with VMware SD-WAN Business Policy

After your CWS policy is created and published for use, it becomes available in Business Policy for Internet Backhaul to a VMware Cloud Web Security Gateway (SASE PoP).

<figure markdown>
  ![Image title](imgs/2-cws/figure54.png){ width="800" }
  <figcaption></figcaption>
</figure>

There is no one-size-fits-all approach to applying CWS policy to your SD-WAN segment(s). Before applying your security policy, there are several questions worth asking:

* What are the site types?

Consider branches that have different Internet access requirements based on their site type. An example could be that one site type only uses thin clients and users access the Internet through the data center. However, other site types with fat clients could benefit from more direct Internet.

* Is the network segmented?

Network segmentation is typically used to partition devices based on access requirements. Consider a branch that has corporate, third-party, and IoT devices. Each group has different access requirements, and at no time should there be inter-group communication to cut down on potential lateral movement and further reduce the network attack surface.

* Do different network segments have different access requirements?

This question builds on the previous. In that example corporate, third-party, and IoT network segments are discussed. This further refines access permissions by asking if each segment has unique Internet access requirements. For example, corporate users will need more general access while IoT devices should receive the minimal Internet access they need to operate. Third party or contractor Internet access may fall somewhere between corporate and IoT.

* Is one CWS policy enough to meet security needs?

As policy becomes more prescriptive it becomes more likely you will need more than one CWS policy. When applied to the preceding question it becomes obvious that a Corporate, Third Party, and IoT CWS policy would make the most sense from a configuration management and auditing perspective. This will also ensure your policies are tailored to fit specific needs versus trying to address them in the rule set.

Once you have your policy or policies crafted, consider how you should apply these to your SD-WAN deployment. A general principle is to set the business policy at the profile level. This approach will ensure your CWS is activated on all devices that inherit the profile. However, if you are piloting your CWS policy before rolling it out organizational-wide, it is appropriate to use Edge Override for the sites selected for testing. Remember, there are situations where one-offs are necessary, but by and large SASE is more successful when deployed consistently and en masse.

In the example business policy rule for Corporate and IoT you will notice that Internet destinations are selected for Backhaul to a VMware Cloud Web Security Gateway to be processed by either the Corporate-Policy or IoT-Policy published in CWS. You can optionally refine the destination criteria to be more specific if needed. For example, combine Internet with Web-based applications so that only Web browsing is sent to CWS, and all other Internet-bound traffic will follow other rules in the business policy.

<figure markdown>
  ![Image title](imgs/2-cws/figure55.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure56.png){ width="800" }
  <figcaption></figcaption>
</figure>

<figure markdown>
  ![Image title](imgs/2-cws/figure57.png){ width="800" }
  <figcaption></figcaption>
</figure>


![Image title](imgs/2-cws/figure58.png "Figure 58"){ width="400" }   ![Image title](imgs/2-cws/figure59.png "Figure 59"){ width="450" }





Do keep in mind that if the SA client needs to communicate with the Workspace ONE UEM (when used with Workspace ONE Intelligent Hub) then an SSL bypass rule in CWS will be needed to prevent that communication from breaking. Simply go into your CWS policy and add a bypass for the awmdm.com domain as shown below.

<figure markdown>
  ![Image title](imgs/2-cws/figure60.png){ width="800" }
  <figcaption></figcaption>
</figure>

When you align your CWS policy between SD-WAN and SA, you achieve a consistent security posture and operating environment for your anywhere workspace workforce.

#### Best Practices for Sending Traffic to CWS

* Determine how many unique CWS policies are best suited for your environment. This requires you to examine your site types, network segments, and consider unique access restrictions you need to impose on different Internet consumers in your environment.
* Select a handful of SD-WAN sites and SA users to pilot your CWS deployment with. In this scenario use the Edge Override feature in Business Policy to apply your CWS settings. For SA users, only have the group you wish to test with use the SA policy you created.
* When your pilot phase is over and CWS is deployed to all sites use Profiles in the VMware SD-WAN Orchestrator for
the Edges on your network. For SA users, **Provision the SA service with the respective CWS policy. The SA service is also
associated with the Organization Group (OG) to which the SA users belong**. Deploy the SA tunnel client using Workspace ONE UEM to have those users start using CWS.
* Align your CWS implementation between SD-WAN and SA. The goal is to create a standard security posture and consistent operating environment for your entire workforce. This strategy is aimed at creating an anywhere workspace for your organization’s workforce.

### Data Loss Prevention (DLP)

As enterprises workloads start to move from on-prem to public cloud and with increasing enterprise user traffic starting to access Internet and SaaS applications, the security of the data being uploaded and downloaded is growing in importance. Sensitive data may be consciously or accidentally transferred into and out of enterprises without the knowledge of the enterprise admin and security teams.

<figure markdown>
  ![Image title](imgs/2-cws/figure61.png){ width="800" }
  <figcaption></figcaption>
</figure>


While CASB allows granular control of access to SaaS applications, these controls only affect the actions that an enterprise user can carry out on sanctioned or unsanctioned websites for example, login, upload and sharing of data. However, these control actions do not account for whether sensitive data is being transferred. With Data Loss Prevention (DLP), Personally Identifiable Information (PII data) and enterprise sensitive data are protected from being exfiltrated from the enterprise. DLP rules are setup with dictionaries which employ a pattern match to scan for this PII or sensitive data and can be configured to block or log the transfer attempt. An alert email can also be configured to be sent to a DLP auditor for review and analysis.

#### Dictionaries

The DLP engine comes with about 340 predefined dictionaries that have ready built pattern match algorithms for the most common types of Personally Identifiable Information (PII data) for example, Bank Account, Credit Card numbers, Passport numbers, Insurance, Social Security, Medical Records, Driver License numbers, etc. as well as region specific information.

<figure markdown>
  ![Image title](imgs/2-cws/figure62.png){ width="800" }
  <figcaption></figcaption>
</figure>


The predefined dictionaries also have a threshold which triggers a DLP violation when exceeded. The threshold setting is based on a sensitivity level with a smaller threshold number indicating a more aggressive pattern match sensitivity and a large threshold number for a less sensitive match. The predefined dictionaries are tuned to reduce false positives and missing data leakage and the recommendation is leave the threshold as-is without modifications.

If predefined dictionaries are not able to cover the type of sensitive data to be scanned, custom dictionaries can also be configured with a pattern match based on a string or regular expression. The regular expression match is based on Perl regular expression syntax. With either string or regular expression match, a repeat pattern count can be specified which would be the threshold trigger for the DLP violation.

#### DLP Auditors

When a DLP violation occurs, an alert email can be sent out to a DLP Auditor. The auditor is one or more email addresses configured to receive the alert email. In addition, the text or file upload that triggered the DLP violation can also be attached in the alert email in the original, zip or encrypted zip format for the auditor to review.

<figure markdown>
  ![Image title](imgs/2-cws/figure63.png){ width="800" }
  <figcaption></figcaption>
</figure>


<figure markdown>
  ![Image title](imgs/2-cws/figure64.png){ width="800" }
  <figcaption></figcaption>
</figure>



#### DLP Rules

DLP Rules are setup to scan the transfer of data based on the source, destination and dictionaries (predefined and/or custom). Based on the threshold set, a DLP violation can trigger the action to block or log the transfer of data, and an email alert can be configured to be sent to a DLP auditor as a result.

The source of a DLP rule can be a user or group if CWS has been setup to authenticate with an Identity Provider for example, Active Directory, Okta, Workspace ONE Access, etc. The default source is all users and groups.


<figure markdown>
  ![Image title](imgs/2-cws/figure65.png){ width="800" }
  <figcaption></figcaption>
</figure>




The content inspected can be in the form of text input and/or a file upload. If a file upload is chosen, the maximum file size can be specified beyond which the file would be dropped, and the auditor notified. A total of 36 file types are supported for inspection and a combination of file types can be selected. The default is for all file types to be inspected.

<figure markdown>
  ![Image title](imgs/2-cws/figure66.png){ width="800" }
  <figcaption></figcaption>
</figure>


By default, all domains and categories of traffic will be inspected. If an enterprise has custom domains which they want to inspect for sensitive data uploads, this can be specified here. Up to 84 different categories of websites can also be chosen depending on whether the enterprise wants to narrow the scope of inspection to specific categories.

<figure markdown>
  ![Image title](imgs/2-cws/figure67.png){ width="800" }
  <figcaption></figcaption>
</figure>


The dictionaries to be used for the pattern match can be selected. Note that predefined dictionaries have default sensitivity thresholds and custom dictionaries have configured repeat patterns which are used as the trigger for actions to be taken when the thresholds are met.

<figure markdown>
  ![Image title](imgs/2-cws/figure68.png){ width="800" }
  <figcaption></figcaption>
</figure>


There are 3 types of action that can take place when a DLP threshold is met. These are Block, Log and Skip Inspection. Based on the action taken, an alert email can also be sent out to the DLP auditor for review. By default both HTTP and HTTPS are protocols that are set to be inspected.

<figure markdown>
  ![Image title](imgs/2-cws/figure69.png){ width="800" }
  <figcaption></figcaption>
</figure>


Finally, a name is given to the DLP rule created. Tags and short description can be added. The text entered in the Notification field would be used as part of the email Subject field. For example, below, the alert email sent would show “DLP Violation” in the subject.

<figure markdown>
  ![Image title](imgs/2-cws/figure70.png){ width="800" }
  <figcaption></figcaption>
</figure>


#### Caveats

Though the current implementation of the DLP engine has support for up to 36 data formats, image files like jpeg, png, gif, etc. are not inspected. This is currently on the roadmap to be implemented in a later release.

Depending on the websites and applications, inspection of data could be part of a form post or a stream/binary request. In some instances, a 403 forbidden error is returned which would launch a DLP block page informing the user of the potential DLP violation taking place. However, there could be situations where a DLP block has taken place but no block page is shown and the text or file upload appears to be hanging or timing out. This is faced industry wide by other security vendors and some have taken the action to have an agent running on the enterprise user’s device that provides the notification. VMware is reviewing this and may add this agent functionality in a future release.


#### Best Practices

The predefined dictionaries pattern match and threshold have been configured and tuned to ensure minimal false positives or missed data is encountered when they are used. The recommendation is not to modify the thresholds. If modifications are required, then proper testing of the new threshold should be done to ensure it meets the enterprise requirements.

Custom dictionaries can be used for pattern matches that are specific to an enterprise for example, to match internal process, domains or enterprise data conventions. By combining this with the predefined dictionaries, a more holistic approach to the Data Loss Prevention can be adopted.

For better security, DLP alert emails sent to auditors should be encrypted. Rather than send this to a generic email inbox, the DLP auditor should be a person or team in the organization that should be tasked to regularly review and analyzed the alert emails and violations. A continuous feedback loop would be favorable to better reduce false positives and fine tune DLP rules to better handle the enterprise needs.
