# Introduction

## Important terms

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
    - provides the information necessary to build route filters
    - aids in troubleshooting and network management by providing information about other networks
- RPKI serves two purposes:
    - provides a cryptographic means to publish your own routing intentions
    - provides a cryptographic means for validating routes you receive


### Why are route filtering and validation important?

- The early Internet was built on trust between the connected networks. BGP has no built-in security, so **any** ASN can announce **any** prefix!
- Network operators are generally well-behaved and announce the correct routes
- Now the Internet has 5 billion users (about 70,000 active ASNs)
- Routing issues can cause financial losses, compromise of data, or even loss of life!
- Money and political gain are powerful incentives for some to tamper with Internet routing
    - Hackers and scammers
    - Unfriendly governments or adversaries
- Mistakes happen — misconfigurations or software bugs can cause route leaks, loops, and traffic loss. Remember [this outage](https://blog.cloudflare.com/how-verizon-and-a-bgp-optimizer-knocked-large-parts-of-the-internet-offline-today/){:target="_blank"} caused by a misconfiguration by DQE in 2019?
- Many providers use IRR to build filters. You must maintain up-to-date IRR entries to ensure all backbone carriers will carry your routes!
    - *"But I don't have IRR, and my providers are carrying my routes just fine!"*
    - Most likely one or more of your providers has created IRR records on your behalf, a practice known as "proxy registration". You should maintain your own IRR records rather than rely on a carrier to get them right. The records may have been created be a carrier you no longer use.
- Many providers filter *RPKI invalid* routes but still accept *RPKI unknown* routes. You get the most protection against route hijacking by signing your route advertisements with RPKI so they are *RPKI valid*.