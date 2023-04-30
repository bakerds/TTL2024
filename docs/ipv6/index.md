# Introduction

## What is IPv6?

IPv6 is the next generation of the Internet Protocol after IPv4, first formalized in RFC 1883 published in 1995

## Comparison of IPv4 and IPv6

| | IPv4 | IPv6 |
|-|------|------|
|Address length | 32 bits | 128 bits |
|Number of unique addresses | 4.3 billion | 340 billion billion billion billion |
|Address notation | Decimal, with a `.` between bytes<br>`192.0.2.2` | Hexadecimal, with a `:` between two-byte words<br>`2001:0db8:0000:0000:0000:0000:0000:0123`<br>May be abbreviated/compressed[^1]<br>`2001:db8::123` |
|Subnet notation | Network ID and subnet mask<br>`192.0.2.0`, `255.255.255.0`<br>CIDR notation<br>`192.0.2.0/24` | Prefix/prefix length<br>`2001:db8::/64` |
| Subnetting | Variable Length Subnet Mask (VLSM):<br>Subnet size is based on expected number of clients | Standard subnet size:<br>/64 |
| Fragmentation | Packets may be fragmented in-flight by routers | Packets must be sized correctly by the sender |
| Number of header fields | 12 | 8 |
| Header Length | 20 bytes, variable | 40 bytes, plus optional extensions |
| Header checksum | Yes | No |
| Host configuration | Static, DHCP, or APIPA (link-local) | Static, DHCP, or SLAAC, and link-local |
| IP to MAC address resolution | Address Resolution Protocol (ARP) (broadcast) | Neighbor Discovery Protocol (NDP) (multicast) |

## Problems with IPv4

- Address exhaustion
	- Difficult/expensive to get IPv4 addresses
	- Need for NAT
- Internet routing table disaggregation
	- Large blocks are being split up and sold off as demand for IPv4 increases
- Forwarding inefficiencies
	- Packet checksum must be recalculated every time the packet is forwaded by a routers due to TTL decrement
	- Header length is variable, making it more complicated to identify the fields 
- Reliance on broadcast for ARP and DHCP

## Improvements with IPv6

- Virtually unlimited address space and subnet size
	- IPv6 addresses are free
	- No need for NAT! Native communication between unique IPv6 addresses
- No reason not to assign aggregable prefixes
- Forwarding efficiency
	- No packet checksum, and no calculation for the router to perform
	- Header length is fixed, so fields are always in the same place
- MAC address discovery and host configuration use multicast instead of broadcast


[^1]:
    Address compression rules: a single run of sequential zeroes can be replaced with a `::` and leading zeroes may be dropped in each address chunk
	
