# Routing Security — BGP, IRR, and RPKI

## BGP

### Border Gateway Protocol: the routing protocol of the Internet

#### Important terms:

- **Router**: a device that connects multiple networks at layer 3
- **Autonomous System (AS)**: a network or group of networks that belongs to a single organization and is connected to the Internet
- **Autonomous System Number (ASN)**: a 16-bit or 32-bit number that identifies an *autonomous system*
- **Border Gateway Protocol (BGP)**: the routing protocol used by Internet routers to exchange routing information between *autonomous systems*
- **Regional Internet Registry (RIR)**: an organization that manages the allocation and registration of Internet number resources such as IPv4 and IPv6 address space and AS numbers within a region of the world
- **American Registry for Internet Numbers (ARIN)**: the *regional Internet registry* for the United States, Canada, and many Caribbean and North Atlantic islands
- **Internet Routing Registry (IRR)**: a public database of routing information that can be used for matching networks to origin ASNs
- **Resource Public Key Infrastructure (RPKI)**: a cryptographic method of signing records that associate a BGP route announcement with the correct originating AS number

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


#### Common scenarios for ISPs or large organizations:

- Transit
- Transit and peering


Routers may *originate* routes or prefixes for their own AS, or they may *announce* routes learned from another AS they are connected to.

