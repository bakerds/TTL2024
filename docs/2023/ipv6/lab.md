# Lab

## Lab objectives

- [ ] Connect to switch and become familiar with basic management operations
- [ ] Configure client ports in several VLANs and test IPv4 connectivity
- [ ] Add IPv6 connectivity to transit VLAN and ping the Internet
- [ ] Add IPv6 connectivity to existing client VLANs
- [ ] Create additional client VLANs with IPv4 and IPv6 connectivity
- [ ] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ ] Bonus: Connect an AP and create a wireless network using Mist

## Switch configuration

### Getting connected to the switch

=== "Windows"

	- Connect your serial console cable to the switch Console port
	- Determine the name of your serial adapter
		- Open Device Manager
		- Expand the **Ports (COM & LPT)** section
		- Take note of the serial port listed (**COM3**, for example)
	- Launch your favorite terminal emulator (**[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html){:target="_blank"}**, for example)
	- Choose **Serial**, enter your COM port name, and connect at **9600** baud

=== "macOS / Linux"

	- Connect your serial console cable to the switch Console port
	- Determine the name of your serial adapter
		- Open a terminal
		- Type `ls /dev/tty.*`
		- Look for a USB tty and note its name (`/dev/ttyUSB0`, for example)
	- Launch your favorite terminal emulator (**minicom** or **screen**, for example) and connect to the desired TTY at 9600 baud
		- `screen /dev/ttyUSB0 9600`
	
## Logging in to the switch CLI
- Hit enter to start a serial session
- Log in to the switch:
```
login: admin
Password: TTL2023!

Last login: Mon May 1 13:09:10

--- JUNOS 21.4R3-S3.4 Kernel 32-bit  JNPR-12.1-20230120.f3fd182_buil
{master:0}
admin@TTL-01>
```

## The JunOS CLI

Basic commands to run in operational mode (`>` prompt)

- `show configuration` — show entire running configuration
- `show interfaces terse` — show interface status
- `configure` — enter configuration mode

Basic commands to use in configuration mode (`#` prompt)

- `show` — show the configuration at the current hierarchy
- `set` — create an element in the configuration
- `delete` — delete an element from the configuration
- `commit` — commit changes from candidate config to running config
- `rollback 0` — revert candidate config back to running config (undo uncommitted changes)
- `rollback 1` - revert candidate config to previous running config (undo most recently committed changes)

!!! note "JunOS Config"
	`show` commands display the configuration in a json-like format. `set` and `delete` commands operate on an element in the configuration tree. Throughout the lag guide, config samples are displayed in both `show` and `set` formats:

	`show` format:

	```
	system {
		host-name TTL-01;
	}
	```
	`set` format:
	```
	set system host-name TTL-01
	```


!!! note "Commiting Changes"
	JunOS uses a two-stage configuration. When editing the configuration, changes are not applied until they are committed. This enables you to prepare config changes without disrupting the current function of the device, and then apply them all at once.

!!! note "CLI Help"
	Typing `?` anywhere in the JunOS CLI will enumerate possible commands or completions from your current position.

**Objectives completed:**

- [x] Connect to switch and become familiar with basic management operations
- [ ] Configure client ports in several VLANs and test IPv4 connectivity
- [ ] Add IPv6 connectivity to transit VLAN and ping the Internet
- [ ] Add IPv6 connectivity to existing client VLANs
- [ ] Create additional client VLANs with IPv4 and IPv6 connectivity
- [ ] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ ] Bonus: Connect an AP and create a wireless network using Mist


## The lab network

### Physical connections
Each "building" (lab group) switch connects to the core via its `ge-0/0/0` interface.

``` mermaid
---
title: Physical Topology
---
flowchart TD
  f(Firewall)---c(Core);
  c---b1(Building 1);
  c---b2(Building 2);
  c---b3(Building 3);
```

### Transit VLANs and routing
Each building has a separate transit VLAN with an IPv4 and IPv6 subnet used for routing. The building will communicate with the core using traffic tagged with its transit VLAN. In the transit subnets, the core is assigned the first host address, and the building switch is assigned the second host address.

For the subnet `10.95.0.4/30`, the host addresses are `10.95.0.5` (core) and `10.95.0.6` (building).

For the subnet `2620:1d5:9500:1::/127`, the host addresses are `2620:1d5:9500:1::` (core) and `2620:1d5:9500:1::1` (building).

``` mermaid
---
title: Transit Links
---
flowchart TD
  f(Firewall)---c(Core);
  c-- Vlan 1301<br>10.95.0.4/30<br>2620:1d5:9500:1::/127 ---b1(Building 1);
  c-- Vlan 1302<br>10.95.0.8/30<br>2620:1d5:9500:2::/127 ---b2(Building 2);
  c-- Vlan 1303<br>10.95.0.12/30<br>2620:1d5:9500:3::/127 ---b3(Building 3);
```

### Address allocations
Each building has an assigned IPv4 and IPv6 prefix. Each prefix is routed to the corresponding building in the core. Client subnets will be chosen from each building's allocation.

``` mermaid
---
title: IP Allocations
---
flowchart TD
  f(Firewall)---c(Core);
  c---b1(Building 1<br>10.95.16.0/20<br>2620:1d5:9501::/48);
  c---b2(Building 2<br>10.95.32.0/20<br>2620:1d5:9502::/48);
  c---b3(Building 3<br>10.95.48.0/20<br>2620:1d5:9503::/48);
```

### Summary Table

|Group/<br>Building |Transit VLAN |IPv4 Transit |IPv6 Transit |IPv4 Allocation |IPv6 Allocation |
|---|-----|----------------|------------------------|-----------------|---------------------|
|1  |3001 |`10.95.0.4/30`  |`2620:1d5:9500:1::/127` |`10.95.16.0/20`  |`2620:1d5:9501::/48` |
|2  |3002 |`10.95.0.8/30`  |`2620:1d5:9500:2::/127` |`10.95.32.0/20`  |`2620:1d5:9502::/48` |
|3  |3003 |`10.95.0.12/30` |`2620:1d5:9500:3::/127` |`10.95.48.0/20`  |`2620:1d5:9503::/48` |
|4  |3004 |`10.95.0.16/30` |`2620:1d5:9500:4::/127` |`10.95.64.0/20`  |`2620:1d5:9504::/48` |
|5  |3005 |`10.95.0.20/30` |`2620:1d5:9500:5::/127` |`10.95.80.0/20`  |`2620:1d5:9505::/48` |
|6  |3006 |`10.95.0.24/30` |`2620:1d5:9500:6::/127` |`10.95.96.0/20`  |`2620:1d5:9506::/48` |
|7  |3007 |`10.95.0.28/30` |`2620:1d5:9500:7::/127` |`10.95.112.0/20` |`2620:1d5:9507::/48` |
|8  |3008 |`10.95.0.32/30` |`2620:1d5:9500:8::/127` |`10.95.128.0/20` |`2620:1d5:9508::/48` |
|9  |3009 |`10.95.0.36/30` |`2620:1d5:9500:9::/127` |`10.95.144.0/20` |`2620:1d5:9509::/48` |
|10 |3010 |`10.95.0.40/30` |`2620:1d5:9500:a::/127` |`10.95.160.0/20` |`2620:1d5:950a::/48` |
|11 |3011 |`10.95.0.44/30` |`2620:1d5:9500:b::/127` |`10.95.176.0/20` |`2620:1d5:950b::/48` |



<details>
<summary>Hint: Core static route config</summary>

```
routing-options {
    rib inet6.0 {
        static {
            route ::/0 next-hop 2620:1d5:fff:9510::;
            route 2620:1d5:9501::/48 next-hop 2620:1d5:9500:1::1;
            route 2620:1d5:9502::/48 next-hop 2620:1d5:9500:2::1;
            route 2620:1d5:9503::/48 next-hop 2620:1d5:9500:3::1;
            route 2620:1d5:9504::/48 next-hop 2620:1d5:9500:4::1;
            route 2620:1d5:9505::/48 next-hop 2620:1d5:9500:5::1;
            route 2620:1d5:9506::/48 next-hop 2620:1d5:9500:6::1;
            route 2620:1d5:9507::/48 next-hop 2620:1d5:9500:7::1;
            route 2620:1d5:9508::/48 next-hop 2620:1d5:9500:8::1;
            route 2620:1d5:9509::/48 next-hop 2620:1d5:9500:9::1;
            route 2620:1d5:950a::/48 next-hop 2620:1d5:9500:a::1;
            route 2620:1d5:950b::/48 next-hop 2620:1d5:9500:b::1;
        }
    }
    static {
        route 0.0.0.0/0 next-hop 10.16.95.1;
        route 10.95.16.0/20 next-hop 10.95.0.6;
        route 10.95.32.0/20 next-hop 10.95.0.10;
        route 10.95.48.0/20 next-hop 10.95.0.14;
        route 10.95.64.0/20 next-hop 10.95.0.18;
        route 10.95.80.0/20 next-hop 10.95.0.22;
        route 10.95.96.0/20 next-hop 10.95.0.26;
        route 10.95.112.0/20 next-hop 10.95.0.30;
        route 10.95.128.0/20 next-hop 10.95.0.34;
        route 10.95.144.0/20 next-hop 10.95.0.38;
        route 10.95.160.0/20 next-hop 10.95.0.42;
        route 10.95.176.0/20 next-hop 10.95.0.46;
    }
}
```

</details>


### Starting config

Each building is pre-configured with the following VLANs already configured for IPv4 connectivity:

=== "Building 1"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.16.0/24` |  |
	|1101 | Wireless Management | `10.95.17.0/24` |  |
	|1300 | Staff Wired         | `10.95.18.0/24` |  |
	|1400 | Staff Wireless      | `10.95.19.0/24` |  |

=== "Building 2"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.32.0/24` |  |
	|1101 | Wireless Management | `10.95.33.0/24` |  |
	|1300 | Staff Wired         | `10.95.34.0/24` |  |
	|1400 | Staff Wireless      | `10.95.35.0/24` |  |

=== "Building 3"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.48.0/24` |  |
	|1101 | Wireless Management | `10.95.49.0/24` |  |
	|1300 | Staff Wired         | `10.95.50.0/24` |  |
	|1400 | Staff Wireless      | `10.95.51.0/24` |  |

=== "Building x"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.[16x].0/24`   |  |
	|1101 | Wireless Management | `10.95.[16x+1].0/24` |  |
	|1300 | Staff Wired         | `10.95.[16x+2].0/24` |  |
	|1400 | Staff Wireless      | `10.95.[16x+3].0/24` |  |

<details>
<summary>Hint: Building 11 Configuration</summary>

```
interfaces {
    irb {
        unit 1100 {
            family inet {
                address 10.95.176.1/24;
            }
        }
        unit 1101 {
            family inet {
                address 10.95.177.1/24;
            }
        }
        unit 1300 {
            family inet {
                address 10.95.178.1/24;
            }
        }
        unit 1400 {
            family inet {
                address 10.95.179.1/24;
            }
        }
        unit 3011 {
            family inet {
                address 10.95.0.46/30;
            }
        }
    }
}
forwarding-options {
    dhcp-relay {
        forward-only;
        server-group {
            TTL {
                10.14.14.100;
            }
        }
        group TTL {
            active-server-group TTL;
            interface irb.1100;
            interface irb.1101;
            interface irb.1300;
            interface irb.1400;
        }
    }
}
routing-options {
    static {
        route 0.0.0.0/0 next-hop 10.95.0.45;
    }
}
vlans {
    v1100 {
        description "Network Management";
        vlan-id 1100;
        l3-interface irb.1100;
    }
    v1101 {
        description "Wireless Management";
        vlan-id 1101;
        l3-interface irb.1101;
    }
    v1300 {
        description "Staff Wired";
        vlan-id 1300;
        l3-interface irb.1300;
    }
    v1400 {
        description "Staff Wireless";
        vlan-id 1400;
        l3-interface irb.1400;
    }
    v3011 {
        vlan-id 3011;
        l3-interface irb.3011;
    }
}

```
```
set interfaces irb unit 1100 family inet address 10.95.176.1/24
set interfaces irb unit 1101 family inet address 10.95.177.1/24
set interfaces irb unit 1300 family inet address 10.95.178.1/24
set interfaces irb unit 1400 family inet address 10.95.179.1/24
set interfaces irb unit 3001 family inet address 10.95.0.46/30
set forwarding-options dhcp-relay forward-only
set forwarding-options dhcp-relay server-group TTL 10.14.14.100
set forwarding-options dhcp-relay group TTL active-server-group TTL
set forwarding-options dhcp-relay group TTL interface irb.1100
set forwarding-options dhcp-relay group TTL interface irb.1101
set forwarding-options dhcp-relay group TTL interface irb.1300
set forwarding-options dhcp-relay group TTL interface irb.1400
set routing-options static route 0.0.0.0/0 next-hop 10.95.0.45
set vlans v1100 description "Network Management"
set vlans v1100 vlan-id 1100
set vlans v1100 l3-interface irb.1100
set vlans v1101 description "Wireless Management"
set vlans v1101 vlan-id 1101
set vlans v1101 l3-interface irb.1101
set vlans v1300 description "Staff Wired"
set vlans v1300 vlan-id 1300
set vlans v1300 l3-interface irb.1300
set vlans v1400 description "Staff Wireless"
set vlans v1400 vlan-id 1400
set vlans v1400 l3-interface irb.1400
set vlans v3011 vlan-id 3011
set vlans v3011 l3-interface irb.3011

```
</details>

**All DHCPv4 scopes needed for the lab already exist on the DHCP server at 10.14.14.100**

### Configure access ports for user devices

```
interfaces {
    ge-0/0/2 {
        description "Network Management";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 1100;
                }
            }
        }
    }
    ge-0/0/3 {
        description "Wireless Management";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 1101;
                }
            }
        }
    }
    ge-0/0/4 {
        description "Staff Wired";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 1300;
                }
            }
        }
    }
}
```

```
set interfaces ge-0/0/2 description "Network Management"
set interfaces ge-0/0/2 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/2 unit 0 family ethernet-switching vlan members 1100
set interfaces ge-0/0/3 description "Wireless Management"
set interfaces ge-0/0/3 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/3 unit 0 family ethernet-switching vlan members 1101
set interfaces ge-0/0/4 description "Staff Wired"
set interfaces ge-0/0/4 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/4 unit 0 family ethernet-switching vlan members 1300
```

Connect a computer to `ge-0/0/2` and verify whether you get an IP address in the correct range and if you have IPv4 and/or IPv6 connectivity. (Be sure to turn off your Wi-Fi first!)

=== "Windows"
	- Launch `cmd.exe` and run `ipconfig`
	- Look for an IPv4 address in the expected range for Vlan 1100
	- Browse to [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"} and check for IPv4 and IPv6 Internet connectivity
	
=== "macOS / Linux"
	- Launch a terminal and run `ip a`
	- Look for an IPv4 address in the expected range for Vlan 1100
	- Browse to [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"} and check for IPv4 and IPv6 Internet connectivity

**Objectives completed:**

- [x] Connect to switch and become familiar with basic management operations
- [x] Configure client ports in several VLANs and test IPv4 connectivity
- [ ] Add IPv6 connectivity to transit VLAN and ping the Internet
- [ ] Add IPv6 connectivity to existing client VLANs
- [ ] Create additional client VLANs with IPv4 and IPv6 connectivity
- [ ] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ ] Bonus: Connect an AP and create a wireless network using Mist


### Add IPv6 to transit VLAN

Example config for building 11 below. Adjust as necessary for your building/group number, referring to the summary table above for the relevant information.

```
interfaces {
    irb {
        unit 3011 {
            family inet {
                address 10.95.0.46/30;              ## IPv4 transit address (already configured)
            }
            family inet6 {
                address 2620:1d5:9500:b::1/127;     ## IPv6 transit subnet
            }
        }
    }
}
routing-options {
    rib inet6.0 {
        static {
            route ::/0 next-hop 2620:1d5:9500:b::;  ## IPv6 default route
        }
    }
    static {
        route 0.0.0.0/0 next-hop 10.95.0.45;        ## IPv4 default route (already configured)
    }
}
```

```
set interfaces irb unit 3011 family inet6 address 2620:1d5:9500:b::1/127
set routing-options rib inet6.0 static route ::/0 next-hop 2620:1d5:9500:b::
```

After committing, test whether the switch can reach its gateway and the Internet:
```
{master:0}
admin@TTL-11> ping 2620:1d5:9500:b:: count 4
PING6(56=40+8+8 bytes) 2620:1d5:9500:1::1 --> 2620:1d5:9500:b::
16 bytes from 2620:1d5:9500:b::, icmp_seq=0 hlim=64 time=9.844 ms
16 bytes from 2620:1d5:9500:b::, icmp_seq=1 hlim=64 time=10.051 ms
16 bytes from 2620:1d5:9500:b::, icmp_seq=2 hlim=64 time=10.012 ms
16 bytes from 2620:1d5:9500:b::, icmp_seq=3 hlim=64 time=5.855 ms

--- 2620:1d5:9500:b:: ping6 statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max/std-dev = 5.855/8.941/10.051/1.783 ms

{master:0}
admin@TTL-11> ping 2600:: count 4
PING6(56=40+8+8 bytes) 2620:1d5:9500:1::1 --> 2600::
16 bytes from 2600::, icmp_seq=0 hlim=53 time=183.059 ms
16 bytes from 2600::, icmp_seq=1 hlim=53 time=183.607 ms
16 bytes from 2600::, icmp_seq=2 hlim=53 time=197.228 ms
16 bytes from 2600::, icmp_seq=3 hlim=53 time=197.185 ms

--- 2600:: ping6 statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max/std-dev = 183.059/190.270/197.228/6.939 ms
```

**Objectives completed:**

- [x] Connect to switch and become familiar with basic management operations
- [x] Configure client ports in several VLANs and test IPv4 connectivity
- [x] Add IPv6 connectivity to transit VLAN and ping the Internet
- [ ] Add IPv6 connectivity to existing client VLANs
- [ ] Create additional client VLANs with IPv4 and IPv6 connectivity
- [ ] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ ] Bonus: Connect an AP and create a wireless network using Mist

### Add IPv6 connectivity client VLANs

Configure each VLAN with the appropriate IPv6 prefix and enable router advertisements

=== "Building 1"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.16.0/24` | `2620:1d5:9501:1100::/64` |
	|1101 | Wireless Management | `10.95.17.0/24` | `2620:1d5:9501:1101::/64` |
	|1300 | Staff Wired         | `10.95.18.0/24` | `2620:1d5:9501:1300::/64` |
	|1400 | Staff Wireless      | `10.95.19.0/24` | `2620:1d5:9501:1400::/64` |

=== "Building 2"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.32.0/24` | `2620:1d5:9502:1100::/64` |
	|1101 | Wireless Management | `10.95.33.0/24` | `2620:1d5:9502:1101::/64` |
	|1300 | Staff Wired         | `10.95.34.0/24` | `2620:1d5:9502:1300::/64` |
	|1400 | Staff Wireless      | `10.95.35.0/24` | `2620:1d5:9502:1400::/64` |

=== "Building 3"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.48.0/24` | `2620:1d5:9503:1100::/64` |
	|1101 | Wireless Management | `10.95.49.0/24` | `2620:1d5:9503:1101::/64` |
	|1300 | Staff Wired         | `10.95.50.0/24` | `2620:1d5:9503:1300::/64` |
	|1400 | Staff Wireless      | `10.95.51.0/24` | `2620:1d5:9503:1400::/64` |

=== "Building x"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.[16x].0/24` | `2620:1d5:950[x]:1100::/64` |
	|1101 | Wireless Management | `10.95.[16x+1].0/24` | `2620:1d5:950[x]:1101::/64` |
	|1300 | Staff Wired         | `10.95.[16x+2].0/24` | `2620:1d5:950[x]:1300::/64` |
	|1400 | Staff Wireless      | `10.95.[16x+3].0/24` | `2620:1d5:950[x]:1400::/64` |


Example config for building 11 below. Adjust as necessary for your building/group number, referring to the summary table above for the relevant information.

```
interfaces {
    irb {
        unit 1100 {
            family inet {
                address 10.95.176.1/24;             ## IPv4 address (already configured)
            }
            family inet6 {
                address 2620:1d5:950b:1100::1/64 {  ## IPv6 address
					subnet-router-anycast;          ## Bind subnet router anycast address to receive router solicitations
				}
            }
        }
        unit 1101 {
            family inet {
                address 10.95.177.1/24;             ## IPv4 address (already configured)
            }
            family inet6 {
                address 2620:1d5:950b:1101::1/64 {  ## IPv6 address
					subnet-router-anycast;          ## Bind subnet router anycast address to receive router solicitations
				}
            }
        }
        unit 1300 {
            family inet {
                address 10.95.178.1/24;             ## IPv4 address (already configured)
            }
            family inet6 {
                address 2620:1d5:950b:1300::1/64 {  ## IPv6 address
					subnet-router-anycast;          ## Bind subnet router anycast address to receive router solicitations
				}
            }
        }
        unit 1400 {
            family inet {
                address 10.95.179.1/24;             ## IPv4 address (already configured)
            }
            family inet6 {
                address 2620:1d5:950b:1400::1/64 {  ## IPv6 address
					subnet-router-anycast;          ## Bind subnet router anycast address to receive router solicitations
				}
            }
        }
    }
}
protocols {
    router-advertisement {
        interface irb.1100 {
            prefix 2620:1d5:950b:1100::/64;
        }
        interface irb.1101 {
            prefix 2620:1d5:950b:1101::/64;
        }
        interface irb.1300 {
            prefix 2620:1d5:950b:1300::/64;
        }
        interface irb.1400 {
            prefix 2620:1d5:950b:1400::/64;
        }
    }
}
```

```
set interfaces irb unit 1100 family inet6 address 2620:1d5:950b:1100::1/64 subnet-router-anycast
set interfaces irb unit 1101 family inet6 address 2620:1d5:950b:1101::1/64 subnet-router-anycast
set interfaces irb unit 1300 family inet6 address 2620:1d5:950b:1300::1/64 subnet-router-anycast
set interfaces irb unit 1400 family inet6 address 2620:1d5:950b:1400::1/64 subnet-router-anycast
set protocols router-advertisement interface irb.1100 prefix 2620:1d5:950b:1100::/64
set protocols router-advertisement interface irb.1101 prefix 2620:1d5:950b:1101::/64
set protocols router-advertisement interface irb.1300 prefix 2620:1d5:950b:1300::/64
set protocols router-advertisement interface irb.1400 prefix 2620:1d5:950b:1400::/64
```

!!! note "Tip"
	`irb unit 1100` and `irb.1100` are equivalent


After committing, once again connect a computer to `ge-0/0/2` and verify whether you get both IPv4 and IPv6 addresses in the correct range and if you have IPv4 and IPv6 connectivity. (Be sure to turn off your Wi-Fi first!)

=== "Windows"
	- Launch `cmd.exe` and run `ipconfig`
	- Look for an IPv4 address in the expected range for Vlan 1100
	- Look for an IPv6 address in the expected range for Vlan 1100
	- Browse to [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"} and check for IPv4 and IPv6 Internet connectivity
	
=== "macOS / Linux"
	- Launch a terminal and run `ip a`
	- Look for an IPv4 address in the expected range for Vlan 1100
	- Look for an IPv6 address in the expected range for Vlan 1100
		- No IPv6 addresses displayed? Try `ip -6 a`
	- Browse to [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"} and check for IPv4 and IPv6 Internet connectivity


**Objectives completed:**

- [x] Connect to switch and become familiar with basic management operations
- [x] Configure client ports in several VLANs and test IPv4 connectivity
- [x] Add IPv6 connectivity to transit VLAN and ping the Internet
- [x] Add IPv6 connectivity to existing client VLANs
- [ ] Create additional client VLANs with IPv4 and IPv6 connectivity
- [ ] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ ] Bonus: Connect an AP and create a wireless network using Mist


### Create two additional client VLANs with IPv4 and IPv6 connectivity

Create VLANs 1401-1402 as shown below, configure IRB interfaces for IPv4 and IPv6, and configure a client port in each VLAN for testing

=== "Building 1"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.16.0/24` | `2620:1d5:9501:1100::/64` |
	|1101 | Wireless Management | `10.95.17.0/24` | `2620:1d5:9501:1101::/64` |
	|1300 | Staff Wired         | `10.95.18.0/24` | `2620:1d5:9501:1300::/64` |
	|1400 | Staff Wireless      | `10.95.19.0/24` | `2620:1d5:9501:1400::/64` |
	|{==1401==} | {==Student Wireless==} | `10.95.20.0/24` | `2620:1d5:9501:1401::/64` |
	|{==1402==} | {==Guest Wireless==}   | `10.95.21.0/24` | `2620:1d5:9501:1402::/64` |

=== "Building 2"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.32.0/24` | `2620:1d5:9502:1100::/64` |
	|1101 | Wireless Management | `10.95.33.0/24` | `2620:1d5:9502:1101::/64` |
	|1300 | Staff Wired         | `10.95.34.0/24` | `2620:1d5:9502:1300::/64` |
	|1400 | Staff Wireless      | `10.95.35.0/24` | `2620:1d5:9502:1400::/64` |
	|{==1401==} | {==Student Wireless==} | `10.95.36.0/24` | `2620:1d5:9502:1401::/64` |
	|{==1402==} | {==Guest Wireless==}   | `10.95.37.0/24` | `2620:1d5:9502:1402::/64` |

=== "Building 3"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.48.0/24` | `2620:1d5:9503:1100::/64` |
	|1101 | Wireless Management | `10.95.49.0/24` | `2620:1d5:9503:1101::/64` |
	|1300 | Staff Wired         | `10.95.50.0/24` | `2620:1d5:9503:1300::/64` |
	|1400 | Staff Wireless      | `10.95.51.0/24` | `2620:1d5:9503:1400::/64` |
	|{==1401==} | {==Student Wireless==} | `10.95.52.0/24` | `2620:1d5:9503:1401::/64` |
	|{==1402==} | {==Guest Wireless==}   | `10.95.53.0/24` | `2620:1d5:9503:1402::/64` |

=== "Building x"

	| VLAN | Name | IPv4 | IPv6 |
	|--|--|--|--|
	|1100 | Network Management  | `10.95.[16x].0/24`   | `2620:1d5:950[x]:1100::/64` |
	|1101 | Wireless Management | `10.95.[16x+1].0/24` | `2620:1d5:950[x]:1101::/64` |
	|1300 | Staff Wired         | `10.95.[16x+2].0/24` | `2620:1d5:950[x]:1300::/64` |
	|1400 | Staff Wireless      | `10.95.[16x+3].0/24` | `2620:1d5:950[x]:1400::/64` |
	|{==1401==} | {==Student Wireless==} | `10.95.[16x+4].0/24` | `2620:1d5:950[x]:1401::/64` |
	|{==1402==} | {==Guest Wireless==}   | `10.95.[16x+5].0/24` | `2620:1d5:950[x]:1402::/64` |


Example config for building 11 below. Adjust as necessary for your building/group number, referring to the table above for the relevant information.

#### Create VLANs

```
vlans {
    v1401 {                    ## Name of the VLAN
        vlan-id 1401;          ## VLAN ID
        l3-interface irb.1401; ## Create a layer 3 routing interface for this VLAN
    }
    v1402 {                    ## Name of the VLAN
        vlan-id 1402;          ## VLAN ID
        l3-interface irb.1402; ## Create a layer 3 routing interface for this VLAN
    }
}
```

```
set vlans v1401 vlan-id 1401
set vlans v1402 vlan-id 1402
set vlans v1401 l3-interface irb.1401
set vlans v1402 l3-interface irb.1402
```

!!! Warning "Note"
	In `set vlans v1401 l3-interface irb.1401`, `l3-interface` is spelled with a lowercase L, as in "layer 3 interface"


#### Create layer 3 routing interfaces

```
interfaces {
    irb {
        unit 1401 {
            family inet {
                address 10.95.180.1/24;
            }
            family inet6 {
                address 2620:1d5:950b:1401::1/64 {
					subnet-router-anycast;
				}
            }
        }
        unit 1402 {
            family inet {
                address 10.95.181.1/24;
            }
            family inet6 {
                address 2620:1d5:950b:1402::1/64 {
					subnet-router-anycast;
				}
            }
        }
    }
}
```

```
set interfaces irb unit 1401 family inet address 10.95.180.1/24
set interfaces irb unit 1402 family inet address 10.95.181.1/24
set interfaces irb unit 1401 family inet6 address 2620:1d5:950b:1401::1/64 subnet-router-anycast
set interfaces irb unit 1402 family inet6 address 2620:1d5:950b:1402::1/64 subnet-router-anycast
```

#### Add DHCP relay for IPv4
```
forwarding-options {
    dhcp-relay {                       ## Already configured
        forward-only;                  ## Already configured
        server-group {                 ## Already configured
            TTL {                      ## Already configured
                10.14.14.100;          ## Already configured
            }                          
        }                              
        group TTL {                    ## Already configured
            active-server-group TTL;   ## Already configured
            interface irb.1401;
            interface irb.1402;
        }
    }
}

```
```
set forwarding-options dhcp-relay group TTL interface irb.1401
set forwarding-options dhcp-relay group TTL interface irb.1402
```

#### Configure a client port and test

Refer to previous objective "Configure access ports for user devices" for example commands.

After configuring a port, connect your client machine and visit [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"} to confirm IPv4 and IPv6 connectivity and verify IPv6 address is in the expected range.


**Objectives completed:**

- [x] Connect to switch and become familiar with basic management operations
- [x] Configure client ports in several VLANs and test IPv4 connectivity
- [x] Add IPv6 connectivity to transit VLAN and ping the Internet
- [x] Add IPv6 connectivity to existing client VLANs
- [x] Create additional client VLANs with IPv4 and IPv6 connectivity
- [x] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ ] Bonus: Connect an AP and create a wireless network using Mist


#### Connect an AP and create a wireless network using Mist

Configure a port for a Mist AP using the configuration below, then ask the presenter for an AP. Once the AP is online, configure an SSID using VLAN tagging in the Mist UI.

```
interfaces {
    ge-0/0/4 {
        native-vlan-id 1101;
        unit 0 {
            family ethernet-switching {
                interface-mode trunk;
                vlan {
                    members [ v1101 v1400 v1401 v1402 ];
                }
            }
        }
    }
}
```
```
set interfaces ge-0/0/4 native-vlan-id 1101
set interfaces ge-0/0/4.0 family ethernet-switching interface-mode trunk 
set interfaces ge-0/0/4.0 family ethernet-switching vlan members [ v1101 v1400 v1401 v1402 ]
```

**Objectives completed:**

- [x] Connect to switch and become familiar with basic management operations
- [x] Configure client ports in several VLANs and test IPv4 connectivity
- [x] Add IPv6 connectivity to transit VLAN and ping the Internet
- [x] Add IPv6 connectivity to existing client VLANs
- [x] Create additional client VLANs with IPv4 and IPv6 connectivity
- [x] Verify that both protocols are working by visiting [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [x] Bonus: Connect an AP and create a wireless network using Mist

### Tools to take home

Sites to visit:

- [test-ipv6.iu13.net](https://test-ipv6.iu13.net){:target="_blank"}
- [ipv6.google.com](https://ipv6.google.com){:target="_blank"}
- [Google IPv6 Statistics](https://www.google.com/intl/en/ipv6/statistics.html){:target="_blank"}

Check out the browser extension *IPvFoo* to get visual feedback on each website about which resources are loaded over IPv4 or IPv6

- [Chrome Extension](https://chrome.google.com/webstore/detail/ipvfoo/ecanpcehffngcegjmadlcijfolapggal?hl=en){:target="_blank"}
- [Firefox Extension](https://addons.mozilla.org/en-US/firefox/addon/ipvfoo-pmarks/){:target="_blank"}
