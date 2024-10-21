---
v: 3
docname: draft-chown-address-accountability-02
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
category: info
area: "Operations"
workgroup: "IPv6 Operations"

venue:
  group: "IPv6 Operations"
  type: "Working Group"
  mail: "v6ops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/v6ops/"
  github: "timchown/address-accountability"
  latest: https://timchown.github.io/address-accountability/draft-chown-address-accountability.html

pi:
  toc: 'yes'
  tocdepth: '3'
  symrefs: 'yes'
  sortrefs: 'yes'
  compact: 'yes'
  subcompact: 'no'
  rfcprocack: 'yes'

title: IPv6 Address Accountability Considerations
area: Operations
workgroup: "IPv6 Operations"
kw: IPv6

author:
- name: Tim Chown
  org: Jisc
  street: 4 Portwall Lane
  city: Bristol
  code: BS1 6NB
  country: United Kingdom
  email: tim.chown@jisc.ac.uk

normative:
  RFC1918:
  RFC2131:
  RFC4649:
  RFC4862:
  RFC8191:
  I-D.ietf-dhc-addr-notification:
informative:
  RFC2663:
  RFC7593:

--- abstract

   Hosts in IPv4 networks typically acquire addresses by use of DHCP,
   and retain that address and only that address while the DHCP lease
   remains valid.  In IPv6 networks, hosts may use DHCPv6, but may
   instead autoconfigure their own global address(es), and potentially use
   many privacy addresses over time.  This behaviour places an
   additional burden on network operators who require address
   accountability for their users and devices.  There has been some
   discussion of this issue on various mail lists; this text attempts to
   capture the issues to encourage further discussion.

--- middle

# Introduction

Administrators of IPv4 networks are used to an address accountability
model where devices acquire a single IPv4 address using DHCP {{RFC2131}}
and then use that address while the DHCP lease is valid.  

The IPv4 address may be a global address, or a private address {{RFC1918}}
used in conjunction with Network Address Translation (NAT) {{RFC2663}}.
The address obtained by DHCP is typically tied to a specific device 
MAC address.
   
This model allows an administrator to track back an IP address to a user or
device, in the event of some incident or fault requiring investigation.
   
While by no means foolproof, this model, which may include use of DHCP 
option 82, is one that IPv4 network administrators are generally 
comfortable with.

In IPv6 networks, where hosts may use SLAAC {{RFC4862}} and Privacy
Addresses {{RFC8191}}, it is quite possible that a host may use
multiple IPv6 addresses over time, possibly changing addresses used
frequently, or using multiple addresses concurrently.  Where privacy
addresses are used, a host may choose to generate and start using a
new privacy address at any time, and will also typically generate a
new privacy address after rebooting.  Clients may use different IPv6
addresses per application, while servers may have multiple addresses
configured, one per service offered.

There are many reasons why address stability is desirable, e.g.,  DNS
mappings, ACLs using IP addresses, and logging.  However, such
stability may not typically exist in IPv6 client networks,
particularly where clients are not managed by the network provider, e.g.,
in campus 'Bring Your Own Device' (BYOD) deployments.

It is also worth noting that in an IPv4 network, it is more difficult
for a user to pick and use an address manually without clashing with
an existing device on the network, while in IPv6 networks picking an
unused address is simple to do without an address clash.  


# Address accountability approaches

The issue of address accountability for IPv6 networks, and thus also 
for dual-stack IPv4-IPv6 networks, is one that has been raised many
times in various discussion fora. This document attempts to capture
the various solutions proposed, noting the advantages and disadvantages
of each approach.

At this stage of the draft's evolution, no single approach is recommended. 
The best solution may vary depending on the scenario and tools available.

The existing approaches to address accountability fall into the 
following categories.

## Switch-router polling

By polling network switch and router devices for IPv4 Address Resolution 
Protocol (ARP) tables and IPv6 Neighbour Discovery (ND) tables, and 
   correlating the results with switch port MAC
   tables, it should be possible to determine which IP addresses are in
   use at any specific point in time and which addresses are being used
   on which switch ports (and thus users or devices).

   This is the approach that has been adopted by tools such as 
   NAV (https://nav.uninett.no) and Netdot (https://nav.uninett.no), and 
   that will be found in many other (open source) tools.  It is sometimes
   referred to as "ND cache scraping".

   The downside of this approach is the load that may be placed on
   devices by frequent Simple Network Management Protocol (SNMP) 
   or other polling. The polling frequency
   needs to be rapid enough to ensure that cached ND/ARP data on devices
   is not expired between polling intervals, i.e., the ND/ARP data should
   not be expired more frequently than the device is polled.

## Record all ND traffic

   If all ND traffic observed on a link can be captured, it should be
   possible for IPv6 address usage to be recorded.  This would require
   appropriate capability on a device on any given subnet, e.g. as is
   currently achieved for RAmond (https://ramond.sourceforge.net) 
   or NDPmon (https://sourceforge.net/projects/ndpmon), or a reporting 
   mechanism for the subnet router, such as syslog.  
   
   There may also be mechanisms such as a (filtered)
   Remote Switch Port Analyser (RSPAN) that may be suitable.

   A benefit of this approach is that collecting all ND traffic would
   allow additional accounting and fault detection to be undertaken,
   e.g. rogue RA detection, or DAD DoS detection.

The downside may be the significant volume of traffic to be held,
if a lengthy history is desired.

## Force use of DHCPv6 only

   One approach to accountability is to attempt to force devices to only
   use DHCPv6, rather than SLAAC, which would in principle give the same address
   accountability model as exists with DHCP for IPv4 today. 
   {{RFC4649}} for DHCPv6 appears to give at least some of the functionality 
   of DHCP option 82.

   While it is possible to craft IPv6 Router Advertisements that give
   hints to hosts that DHCPv6 should be used, i.e., the'M' bit is set, there is
   no obligation on the host to honour that hint.  However, if the
   Autonomous (A) flag in the Prefix Information option is unset (as
   discussed in section 5.5.3 of RFC 4862), the Prefix Information
   option should be ignored.  In such cases a user running the device will need to
   determine the on-link prefix if they wish to manually configure their
   own address.

Not all common operating systems support DHCPv6 for host addressing,
Android being the most notable exception. Larger enterprises that are 
enforcing use of DHCPv6 appear to be ones running dual-stack, so in those
scenarios clients that do not support DHCPv6 will be IPv4-only.

## Using 802.1X

It is now quite common practice for research and education sites - 
university or college campuses and research organisations - to use 802.1X
for network authentication as part of the international
eduroam {{RFC7593}} (https://www.eduroam.org) federated authentication system, as
deployed at thousands of educational sites across over 70 countries.
Use of 802.1X gives the network operator knowledge about their own users
authenticating, but would require coordination with remote peers for
details of visiting users admitted via eduroam.

802.1X can also be used for wired network access control.

## Use a host-based registration protocol

A current I-D, {{I-D.ietf-dhc-addr-notification}}, presents a 
mechanism for hosts to opt in to registering 
their self-generated or statically-configured addresses to a DHCPv6
server.

## Using a prefix per host

By using a prefix per host, the accountability model shifts from 
identifying address(es) used by a host, to the prefix from which it
is using addresses, whether for itself (e.g., for containers) or
for providing tethering.

## Use SAVI mechanisms

   Discussion of appropriateness of SAVI mechanisms to be added here.
   (In principle, SAVI mechanisms work by observing NDP and DHCP
   messages, allowing bindings to be set up and recorded.)

# Privacy Considerations

   This draft discusses mechanisms for a site or organisation to manage
   address accountability where IPv6 has been deployed.  In most
   networks there is a requirement to be able to identify which users
   have been using which addresses or devices at a given point in time.
   This draft was written in response to requests for improved
   accountability for IPv6 traffic in university campus sites, but
   the same rationale is likely to apply elsewhere.

   While the sources of data that may be used for such purposes (e.g.
   state on routers or switches) is generally not available to general
   users of the network, it is available to administrators of the
   network.  The use of privacy mechanisms, e.g.,  RFC 8191, gives the
   greatest benefit when the addresses are being observed by external
   third parties.


# Conclusions

   This text is an initial draft attempting to capture the issues
   related to IPv6 address accountability models.  If an all-DHCPv6
   model is not viable, IPv6 network administrators will need to deploy
   management and monitoring tools to allow them to account for hosts
   that will have multiple IPv6 addresses that may also change rapidly
   over time.

   Some of the approaches described do not depend on a specific type of
   address management being used, and will thus work with other
   addressing methods if they emerge in the future.

   Feedback on the issues discussed here is welcomed.


# Security Considerations

There are no extra security consideration for this document.


# IANA Considerations

This document has no IANA actions.

# Acknowledgments
{:numbered="no"}

The author would like to thank the following people for comments on
   this text: Mark Smith and James Woodyatt.
   
--- back
