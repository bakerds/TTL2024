# Routing Security — BGP, IRR, and RPKI

## Important terms:

- **Router**: a device that connects multiple networks at layer 3
- **Autonomous System (AS)**: a network or group of networks that belongs to a single organization and is connected to the Internet
- **Autonomous System Number (ASN)**: a 16-bit or 32-bit number that identifies an *autonomous system*
- **Border Gateway Protocol (BGP)**: the routing protocol used by Internet routers to exchange routing information between *autonomous systems*
- **Regional Internet Registry (RIR)**: an organization that manages the allocation and registration of Internet number resources such as IPv4 and IPv6 address space and AS numbers within a region of the world
- **American Registry for Internet Numbers (ARIN)**: the *regional Internet registry* for the United States, Canada, and many Caribbean and North Atlantic islands
- **Internet Routing Registry (IRR)**: a public database of routing information that can be used for matching networks to origin ASNs
- **Resource Public Key Infrastructure (RPKI)**: a cryptographic method of signing records that associate a BGP route announcement with the correct originating AS number


## Border Gateway Protocol: the routing protocol of the Internet

- BGP is the routing protocol used by all Internet service providers and many of their customers
- BGP routers may *originate* routes or prefixes for their own AS, or they may *announce* routes learned from another AS they are connected to
- An ISP router typically has the full Internet routing table in memory and knows one or more paths to every prefix announced to the Internet
- When there are multiple paths to a destination, the router uses a calculation based on BGP route attributes and locally administered policy to determine the best route


### What are IRR and RPKI for?

- IRR serves three purposes:
	- provides a means to publish your own routing intentions
	- provites the information necessary to build route filters
	- aids in troubleshooting and network management by providing information about other networks
- RPKI serves two purposes:
	- provides a cryptographic means to publish your own routing intentions
	- provides a cryptographic means for validating routes you receive


### Why are route filtering and validation important?

- The early Internet was built on trust between the connected networks
- Network operators are generally well-behaved and announce the correct routes
- Now the Internet has 5 billion users (about 70,000 active ASNs)
- Routing issues can cause financial losses, compromise of data, or even loss of life!
- Money and political gain are powerful incentives for some to tamper with Internet routing
	- Hackers and scammers
	- Unfriendly governments or adversaries
- Mistakes happen -- misconfigurations or software bugs can cause route leaks, loops, and traffic loss
- Many providers use IRR to build filters. You must maintain up-to-date IRR entries to ensure all backbone carriers will carry your routes!
	- *"But I don't have IRR, and my providers are carrying my routes just fine!"*
	- Most likely one or more of your providers has created IRR records on your behalf, a practice known as "proxy registration". You should maintain your own IRR records rather than rely on a carrier to get them right. The records may have been created be a carrier you no longer use.
- Many providers filter *RPKI invalid* routes but still accept *RPKI unknown* routes. You get the most protection against route hijacking by signing your route advertisements with RPKI so they are *RPKI valid*.




## IRR: Internet Routing Registry

The IRR is not a single database -- there are several routing registries to choose from:
- ARIN (most likely choice)
- RADb (paid service, mirrors others)
- Level3 (mirrors others)
- NTT (mirrors others)

The IRR can be queried using WHOIS: `whois -h rr.arin.net AS14773`

*Using Windows? Try Windows Subsystem for Linux. At an elevated command prompt, run `wsl --install`. After installation completes, launch Ubuntu from the Start menu and run `sudo apt install whois`.*


## Using the IRR

### Publish routing information about your networks

#### Object types

- route/route6: describes a prefix that you will originate
	- Maintained By: links the record to your ARIN organization
	- Prefix: the prefix you plan to announce
	- Origin: the ASN that will originate the prefix
	- Description: your organization's name and address, and any additional information about the prefix
- as-set: specifies a set of Autonomous System Numbers (ASNs) through which traffic can be routed. Useful if you have downstream ASNs (customers)
	- AS Set Name: the unique name of the as-set
	- Description: your organization's name and address, and any additional information about the as-set
	- Members: ASNs and as-set names that are included in the as-set
- aut-num: specifies an ASN and its routing policies
	- Maintained By: links the record to your ARIN organization
	- ASN: the ASN described by this object
	- AS Name: a human-readable nickname for the ASN
	- Description: your organization's name and address, and any additional information about the autonomous system
	- Routing Policy Specifications:
		- Import: IPv4 import policy
		- Export: IPv4 export policy
		- Default: IPv4 default routing policy
		- Mp Import: multi-protocol import policy (multi-protocol support added later for IPv6)
		- Mp Export: multi-protocol export policy
		- Mp Default: multi-protocol default routing policy
		- Remarks: additional comments or notes visible to the public
- route-set: a set of of IPv4 prefixes, IPv6 prefixes, and other route-sets that can be used in aut-num policy specifications
	- Route Set Name: the unique name of the route-set
	- Description: your organization's name and address, and any additional information about the route-set
	- Members: a list of IPv4 prefixes or route-sets to include
	- Mp Members: a list of IPv6 prefixes or route-sets to include
	- Members by Reference: a list of organizations whose route objects should be included in the route-set
	- Remarks: additional comments or notes visible to the public

#### Example objects:

##### aut-num
IU13 WAN
```
$ whois -h rr.arin.net AS14773
aut-num:        AS14773
as-name:        LANLEB-IU13
descr:          Lancaster-Lebanon IU13 WAN Consortium
                1020 New Holland Avenue
                Lancaster PA 17601
                United States
member-of:      AS-IU13WAN
mp-export:      afi any.unicast to AS-ANY announce AS-IU13WAN
admin-c:        NETWO9356-ARIN
tech-c:         NETWO9356-ARIN
mnt-by:         MNT-LLIU1
created:        2023-02-02T18:08:17Z
last-modified:  2023-02-02T18:08:17Z
source:         ARIN
```

Hempfield School District
```
$ whois -h rr.arin.net AS395182
aut-num:        AS395182
as-name:        HEMPFIELD-SD
descr:          200 Church Street
                Landisville PA 17538
                United States
admin-c:        GRAHA156-ARIN
tech-c:         FARME185-ARIN
tech-c:         GRAHA156-ARIN
tech-c:         LICHT37-ARIN
mnt-by:         MNT-HSD-37
created:        2022-03-05T12:49:41Z
last-modified:  2022-03-05T12:49:41Z
source:         ARIN
```

##### as-set
IU13 WAN
```
$ whois -h rr.arin.net AS-IU13WAN
as-set:         AS-IU13WAN
descr:          Lancaster-Lebanon IU13 WAN Consortium
                1020 New Holland Avenue
                Lancaster PA 17601
                United States
members:        AS14773,AS395182,AS394781,AS398574
admin-c:        NETWO9356-ARIN
tech-c:         NETWO9356-ARIN
mnt-by:         MNT-LLIU1
created:        2022-03-07T21:58:53Z
last-modified:  2022-06-01T15:09:59Z
source:         ARIN
```

##### route/route6
```
$ whois -h rr.arin.net 206.82.16.0/20
route:          206.82.16.0/20
origin:         AS14773
descr:          Lancaster-Lebanon IU13 WAN Consortium
                1020 New Holland Avenue
                Lancaster PA 17601
                United States
admin-c:        NETWO9356-ARIN
tech-c:         NETWO9356-ARIN
mnt-by:         MNT-LLIU1
created:        2022-03-07T22:06:29Z
last-modified:  2022-03-07T22:06:29Z
source:         ARIN

$ whois -h rr.arin.net 2620:1d5::/32
route6:         2620:1d5::/32
origin:         AS14773
descr:          Lancaster-Lebanon IU13 WAN Consortium
                1020 New Holland Avenue
                Lancaster PA 17601
                United States
admin-c:        NETWO9356-ARIN
tech-c:         NETWO9356-ARIN
mnt-by:         MNT-LLIU1
created:        2022-10-03T15:48:56Z
last-modified:  2022-10-03T15:48:56Z
source:         ARIN
```

### Build route filters using IRR



---


---





#### Common scenarios for school districts connecting to the Internet:

- Static routing
	- The school leases a public IP address range from an ISP and uses a static route to send outbound traffic to the ISP
	- The ISP owns the public IP address range and originates a BGP advertisement to the Internet
	- Dual-homing is possible with NAT, but failover is bumpy and public-facing services are broken if the main provider is down
	
- BGP with provider aggregable (PA) address space
	- The school leases public IP address range from an ISP and uses BGP to advertise the prefix to the ISP
	- The ISP owns the public IP address range and likely originates a BGP advertisement for a *larger* prefix to the Internet
	- Dual-homing with two connections to the *same provider* is possible. Dual-homing with two *different* providers *may* be possible depending on prefix size and the policies of both ISPs
	
- BGP with provider independent (PI) address space
	- The school is assigned an ASN and public IP address range from an RIR and uses BGP to advertise the prefix to the ISP
	- The school originates the route from the school's ASN
	- Dual-homing with multiple providers is possible (and required to quality for an ASN and IP allocation)


#### Common scenarios for large organizations or small ISPs:

- Transit
	- The network operator purchases *full transit* connectivity from one or more larger providers. Full transit means a complete view of the Internet routing table and connectivity to 100% of the Internet
	- The network operator has its own ASN and public IP allocations which it advertises to upstream carriers, along with customer advertisements
- Transit and peering
	- In addition to transit, the operator may establish *peering* with other providers. Peering is the exchange of a limited set of routes and traffic with another network, often a content provider such as Akamai, Netflix, or Google
	- Peering is often available at no cost when the two networks are present in a common location
	- 




- Common types of BGP relationships:
	- Transit provider
		- Transit or "full transit" means the network operator purchases a complete view of the Internet routing table and connectivity to 100% of the Internet from a larger provider
		- The network operator has its own ASN and public IP allocations which it advertises to upstream carriers, along with customer advertisements
	- Peering
		- Peering is the exchange of a limited set of routes and traffic with another network, frequently a content provider such as Akamai, Netflix, or Google
		- Peering is often available at no cost when the two networks are present in a common location
		- Peering helps to reduce transit bandwidth demands
	- Customer
		- A customer peering is wh
