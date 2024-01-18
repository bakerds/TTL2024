# Network Design - Segmentation

## Why is segmentation important?

**Efficiency and reliability**

- Network segments are typically broadcast domains
	- A broadcast is message that must be delivered to every host in the broadcast domain
	- Broadcasts are used by ARP and DHCP, and other protocols
	- Putting too many hosts in a single layer 2 segment can create a large volume of broadcast traffic
	- Carving layer 2 segments limits broadcast domains to manageable sizes
- Switches must learn and store the MAC address of every device in the layer 2 segment
	- Switches have finite memory resources and can only learn a certain number of MAC addresses (16K-112K)
	- Creating layer 2 segments limits the number of MAC address each switch needs to learn
- Segmenting creates smaller failure domains
	- It is easier to troubleshoot an issue when you can easily identify the scope

**Security**

- Segmentation makes it easier to control communication between hosts
	- Rather than being able to communicate directly, hosts in different subnets must communicate through a router (and possibly a firewall), which may apply security policy.
- Segmentation helps avoid spoofing or impersonation
	- Hosts in one subnet cannot easily impersonate hosts in another subnet

**Organization and management**

- Segmentation helps to keep the network organized
- Grouping similar devices or users together makes it easier to apply the correct policies

## How to segment?

**Layer 2 – VLAN**

- VLAN allows a single switch to handle multiple separate layer 2 segments
- VLAN tagging is implemented as an additional header in the layer 2 frame
	- 12-bit header field, permitting 4094 VLAN IDs (0 and 4095 are reserved)
- Layer 2 frames may be tagged or untagged
	- A tagged frame has a VLAN ID in the range of 1 - 4094 in the header
	- An untagged frame has no VLAN ID in the frame header

**Layer 3 – IP subnetting**

- A unique IP subnet is defined and assigned to each layer 2 segment
- A router is required for communication between subnets

## What to segment?

Functions that should be segmented:

- Infrastructure
	- Network management
	- Server management
	- DMZ servers
	- Internal servers
	- Printers
	- Security cameras
	- Security devices / building access control
	- HVAC and building automation
	- Phones and phone systems
- Users
	- Administrative users – wired
	- Administrative users – wireless
	- Faculty/staff – wired
	- Faculty/staff – wireless
	- Students – wired
	- Students – wireless
	- Guests – wired
	- Guests – wireless

**Each building/campus should be segmented from the others**

## IP Address Management / Subnet Planning

**Subnet size**

- IPv4
	- Large enough to accommodate all unique devices that will connect to the subnet within 24-48 hours
	- Typically no smaller than /24 (253 host addresses)
	- Ideally no larger than /20 (4093 host addresses)
- IPv6
	- Always /64 (18 billion billion host addresses)

**Allocating address space**

TTL School District:

- 6 buildings (2 large, 4 small)
- 3,500 students

How much IP address space should be allocated to each building and to the district?

- IPv4: depends on building population
	- Per-building enrollment:
		- HS - 1400 - /18
		- MS - 700 - /18
		- EL1 - 350 - /19
		- EL2 - 350 - /19
		- EL3 - 350 - /19
		- EL4 - 350 - /19
	- Total needed: /16
- IPv6: /48 per building (standard allocation - enough for 64K subnets)
	- Total needed: /44 (enough for 16 buildings)

**Example IPAM**

| Building | IPv4 | IPv6 |
|----------|------|------|
| HS  | `10.95.0.0/18`   | `2620:1d5:9500::/48` |
| MS  | `10.95.64.0/18`  | `2620:1d5:9501::/48` |
| EL1 | `10.95.128.0/19` | `2620:1d5:9502::/48` |
| EL2 | `10.95.160.0/19` | `2620:1d5:9503::/48` |
| EL3 | `10.95.192.0/19` | `2620:1d5:9504::/48` |
| EL4 | `10.95.224.0/19` | `2620:1d5:9505::/48` |


**HS**

| VLAN ID | Name | IPv4 | IPv6 |
|---------|------|------|------|
|1100 | Network Management     | `10.95.1.0/24`  | `2620:1d5:9500:1100::/64` |
|1101 | Wireless Management    | `10.95.2.0/23`  | `2620:1d5:9500:1101::/64` |
|1102 | Hypervisor Management  | `10.95.4.0/24`  | `2620:1d5:9500:1102::/64` |
|1103 | Servers                | `10.95.5.0/24`  | `2620:1d5:9500:1103::/64` |
|1104 | DMZ                    | `10.95.6.0/24`  | `2620:1d5:9500:1104::/64` |
|1200 | Door Access Control    | `10.95.7.0/24`  | `2620:1d5:9500:1200::/64` |
|1201 | Security Cameras       | `10.95.8.0/23`  | `2620:1d5:9500:1201::/64` |
|1202 | HVAC Controls          | `10.95.10.0/24` | `2620:1d5:9500:1202::/64` |
|1203 | Phones                 | `10.95.12.0/23` | `2620:1d5:9500:1203::/64` |
|1300 | IT Staff               | `10.95.20.0/24` | `2620:1d5:9500:1300::/64` |
|1301 | Administrative Staff   | `10.95.21.0/24` | `2620:1d5:9500:1301::/64` |
|1302 | Staff Wired            | `10.95.22.0/23` | `2620:1d5:9500:1302::/64` |
|1303 | Student Wired          | `10.95.24.0/22` | `2620:1d5:9500:1303::/64` |
|1400 | Staff Wireless         | `10.95.28.0/22` | `2620:1d5:9500:1400::/64` |
|1401 | Student Wireless       | `10.95.32.0/20` | `2620:1d5:9500:1401::/64` |
|1402 | Guest Wireless         | `10.95.48.0/20` | `2620:1d5:9500:1402::/64` |

**MS**

| VLAN ID | Name | IPv4 | IPv6 |
|---------|------|------|------|
|1100 | Network Management     | `10.95.65.0/24`  | `2620:1d5:9501:1100::/64` |
|1101 | Wireless Management    | `10.95.66.0/23`  | `2620:1d5:9501:1101::/64` |
|1102 | Hypervisor Management  | `10.95.68.0/24`  | `2620:1d5:9501:1102::/64` |
|1103 | Servers                | `10.95.69.0/24`  | `2620:1d5:9501:1103::/64` |
|1104 | DMZ                    | `10.95.70.0/24`  | `2620:1d5:9501:1104::/64` |
|1200 | Door Access Control    | `10.95.71.0/24`  | `2620:1d5:9501:1200::/64` |
|1201 | Security Cameras       | `10.95.72.0/23`  | `2620:1d5:9501:1201::/64` |
|1202 | HVAC Controls          | `10.95.74.0/24`  | `2620:1d5:9501:1202::/64` |
|1203 | Phones                 | `10.95.76.0/23`  | `2620:1d5:9501:1203::/64` |
|1300 | IT Staff               | `10.95.84.0/24`  | `2620:1d5:9501:1300::/64` |
|1301 | Administrative Staff   | `10.95.85.0/24`  | `2620:1d5:9501:1301::/64` |
|1302 | Staff Wired            | `10.95.86.0/23`  | `2620:1d5:9501:1302::/64` |
|1303 | Student Wired          | `10.95.88.0/22`  | `2620:1d5:9501:1303::/64` |
|1400 | Staff Wireless         | `10.95.90.0/22`  | `2620:1d5:9501:1400::/64` |
|1401 | Student Wireless       | `10.95.96.0/20`  | `2620:1d5:9501:1401::/64` |
|1402 | Guest Wireless         | `10.95.112.0/20` | `2620:1d5:9501:1402::/64` |

