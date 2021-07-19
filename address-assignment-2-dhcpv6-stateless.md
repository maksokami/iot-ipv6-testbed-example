# DHCPv6 stateless
Server/router sends RA and DHCPv6 information (network prefix, default gateway, DNS, NTP). However, unlike stateful DHCPv6 and classical DHCPv4, the server/router doesn't assign unique IP addresses on its own and doesn't keep lease table.

![DHCPv6 stateless](https://www.cisco.com/c/dam/en/us/support/docs/ip/ip-version-6-ipv6/213272-troubleshoot-ipv6-dynamic-address-assign-08.png)

## What to expect?
- Client will learn correct prefix from the Router automatically (taken from RA packet)
- Client will learn router's IP and configure it as default gateway automatically (taken from RA packet)
- Client will use EUI-64 to generate a unique IP address automatically
- DNS will be configure automatically via DHCPv6
- NTP will not be configured in this example 


## Router (RADVD)

```
apt install radvd
```
Create /etc/radvd.conf. Bind to Client facing interface with IPv6
```
interface enp0s8 {
  AdvSendAdvert on;
  MinRtrAdvInterval 3; 
  MaxRtrAdvInterval 10;
  AdvManagedFlag off;
  AdvOtherConfigFlag on;
  AdvLinkMTU 1500;
  prefix fd:0:0:1::/64 {
    AdvOnLink on; 
    AdvAutonomous on; 
    AdvRouterAddr off; 
  };
  # Router is the DNS
  RDNSS fd:0:0:1::1 {};
  # Configure lookup domain list
  DNSSL test.local {};
};
```
Restart the service
```
systemctl restart radvd
systemctl status radvd --no-pager
```

## Client
Install rdnssd to autoconfigure DNS using ICMPv6 Neighbor Discovery (RFC 5006)
```
apt update
apt install -y rdnssd

```
Alter /etc/network/interface

```
iface enp0s3 dhcp
 accept_ra 2
 request_prefix 1
 autoconf 1
```

Verify
```
ip -6 addr show enp0s3

cat /etc/resolv.conf
```

> **Why accept_ra 2?** 
> We consider for this lab that our IoT device/client required IP forwarding to be enabled. By default Linux device will disable SLAAC interface configuration if it considers itself a router (ip forwarding enabled). To enable SLAAC we need to choose option "2" for accept_ra to override 'if forwarding' behavior.
Read more here: https://strugglers.net/~andy/blog/2011/09/04/linux-ipv6-router-advertisements-and-forwarding/
