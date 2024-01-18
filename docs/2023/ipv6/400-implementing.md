# Implementing IPv6

## Transition

- IPv6 is not backward-compatible with IPv4 - the two protocols cannot natively communicate
- Transition mechanisms allow IPv6-only devices to communicate with IPv4-only devices and vice-versa
	- 6in4 and Teredo tunneling - tunneling IPv6 traffic across IPv4-only intrastructure
	- 6rd and 6to4 - stateless tunneling between special endpoints
	- NAT64 - allows IPv6 clients to reach IPv4 servers through a NAT64 gateway router
		- DNS64 - works in conjunction with NAT64 to help IPv6 clients reach IPv4 servers using an IPv6 address
		- 464XLAT[^1] - allows IPv6 clients to reach IPv4 servers using stateless translation at the client and NAT64 router (T-Mobile)
- There are limitations with every transition technology

## Dual-Stack

**For the forseeable future, networks should run both IPv4 and IPv6**

- Keeping IPv4
	- IPv4 works well inside the LAN
	- Natively reach networks/websites that are still IPv4-only
	- Some devices only support IPv4 (IOT)
- Adding IPv6
	- IPv6 is more efficient over the Internet
	- Natively reach networks/websites that are IPv6-enabled
	- All modern operating systems support and *prefer* IPv6
	- Adding IPv6 will dramatically reduce the amount of IPv4 traffic leaving the network


[^1]:
    Widely deployed in mobile LTE networks. This is currently the best choice if you need to run a single protocol on a client access network.
