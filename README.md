# IoT ipv6 testbed example
This configuration helps to test ipv6 support on IoT device middleware.
Commands test on Debian9 VMs.

- **client-configuration.md**. Client (=IoT device) should be configured to use IPv6.
- **router-configuration.md**. Router (=intermediate device) serves as IPv6 gateway that does DNS64 and NAT64 translation.
- Private ipv6 block fd00::/8 is used in the examples (RFC 4193).
- 64:ff9b:: is reserved for address translation (RFC 8215) and used on the router.

Scenario 1: Most common. IPv4 only home network with internet access

| Client (IPv6) |  | Router (ipv6 and ipv4) | | WAN |
| ------------ | -- | -----| ---| --- | 
| fd:0:0:1::5/96 | <-> | LAN: fd:0:0:1::1/96 -NAT64- WAN: ipv4 home dhcp? | -> | Home ipv4 network with internet |
| Default gw: Router |   | Default ipv4 gw: Home network |  |  - | 
| Static route: 64:ff9b:: to Router  |   |  |  |   | 
| DNS: Router |   | DNS64 (default: all translated) |  |  - | 

Scenario 2: Dual-stack on the router (Home internet is dual-stack). 
In this case Router should only translate IPv4 DNS and leave IPv6 responces intact. Client will be able to access pure IPv6 resources using home internet.

| Client (IPv6) |  | Router (ipv6 and ipv4) | | WAN |
| ------------ | -- | ----- | ---| --- | 
| fd:0:0:1::5/96 | <-> | LAN: fd:0:0:1::1/96 -NAT64- WAN: ipv6 home dhcp? | -> | Home ipv6 network|
|   |   | WAN2 ipv4 home dhcp? | ->  |  Home ipv4 network | 
| Default gw: Router |   | Default ipv6 gw: Home ipv6 network |  |   | 
| Static route: 64:ff9b:: to Router  |   |  |  |   | 
|   |   | Default ipv4 gw: Home ipv4 network |   |   | 
| DNS: Router |   | DNS64 (enable dual-stack. see config comments) |  |  - | 

> Note: You can replace network fd00:: with your private home ipv6 network and connect client and router to the same switch instead of direct connection

RFCs:
[Unique Local IPv6 Unicast Addresses](https://datatracker.ietf.org/doc/html/rfc4193)
[IPv6 Addressing of IPv4/IPv6 Translators](https://datatracker.ietf.org/doc/html/rfc6052)
[Local-Use IPv4/IPv6 Translation Prefix](https://datatracker.ietf.org/doc/html/rfc8215)
