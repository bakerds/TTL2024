# Configuring IPv6

Tips:

- Allocate a /48 per building
- Use a /64 for every subnet
- Use a /127[^1] for transit links, but reserve the entire /64
- Use SLAAC for client configuration
- Provide DHCPv6 for scope options only (but also include them in the router advertisement)
- Start your deployment with transit links and client subnets

## Configuration samples

**JunOS**
```
interfaces {
	irb {
		unit 1300 {
			description "IT Staff";
			family inet {
				address 10.95.20.1/24;
			}
			family inet6 {
				address 2620:1d5:9500:1300::1/64 {
					subnet-router-anycast;
				}
			}
		}
	}
}
protocols {
    router-advertisement {
        interface irb.1300 {
			other-stateful-configuration;
			dns-server-address 2620:1d5:9500:1103::dc;
			dns-server-address 2620:1d5:9501:1103::dc;
            prefix 2620:1d5:9500:1300::/64;
        }
	}
}
forwarding-options {
    dhcp-relay {
        dhcpv6 {
            group TTL {
                interface irb.1300;
            }
            server-group {
                TTL {
                    2620:1d5:9500:1103::dc;
                    2620:1d5:9501:1103::dc;
                }
            }
            active-server-group TTL;
        }
        forward-only;
        server-group {
            TTL {
                10.95.5.2;
                10.95.69.2;
            }
        }
        group TTL {
            active-server-group TTL allow-server-change;
            interface irb.1300;
        }
    }
}

```

**Cisco IOS**
```
interface Vlan1300
 description "IT Staff"
 ip address 10.95.20.1 255.255.255.0
 ip helper-address 10.95.5.2
 ip helper-address 10.95.69.2
 ipv6 address 2620:1d5:9500:1300::/64 anycast
 ipv6 enable
 ipv6 nd other-config-flag
 ipv6 dhcp relay destination 2620:1d5:9500:1103::dc
 ipv6 dhcp relay destination 2620:1d5:9501:1103::dc
!
```

**Ruckus ICX**
```
interface ve 1300
 port-name IT_Staff
 ip address 10.95.20.1/24
 ip helper-address 1 10.95.5.2
 ip helper-address 2 10.95.69.2
 ipv6 address 2620:1d5:9500:1300::1/64
 ipv6 address 2620:1d5:9500:1300::/64 anycast
 ipv6 dhcp-relay destination 2620:1d5:9500:1103::dc
 ipv6 dhcp-relay destination 2620:1d5:9501:1103::dc
 ipv6 nd other-config-flag
!
```


[^1]:
	Some equipment doesn't support /127, so a /126 may be used if necessary