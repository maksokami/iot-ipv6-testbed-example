# DHCPv6 stateful
What's different compared to stateless DHCPv6:
- Clients are allocated unique IPv6 addresses (client don't have to use and show EUI addresses that uncover their MAC address and therefore manufacturere)
- Server/router keeps lease table to keep track of assigned addresses
- Server/router can do prefix delegration (not only tell the client how to configure WAN interface but tell a list of prefixes Client can assign to it's LAN side devices, in case the Client device wants to be a router)

![DHCPv6 stateful](https://www.cisco.com/c/dam/en/us/support/docs/ip/ip-version-6-ipv6/213272-troubleshoot-ipv6-dynamic-address-assign-09.png)

## Router (ISC DHCP server)
Install DHCP server
```
apt install -y isc-dhcp-server
```
Alter config /etc/dhcp/dhcpd6.conf. (Timers are set very short in this example - increase for production.
```
default-lease-time 120;
preferred-lifetime 360;
# 1/2 preferred lifetime
option dhcp-renewal-time 180;
# 3/4 preferred lifetime
option dhcp-rebinding-time 270;

# Enable RFC 5007 support
allow leasequery;


option dhcp6.name-servers fd:0:0:1::1;
option dhcp6.domain-search "test.local";

option dhcp6.info-refresh-time 600;

subnet6 fd:0:0:1::/64 {
	range6  fd:0:0:1:1::1   fd:0:0:1:2::1;
}

```

Create a completely separate IPv6-only service (In case we want to preserve and control IPv4 DHCP server separately)
```
sudo cp /etc/init.d/isc-dhcp-server /etc/init.d/isc-dhcp-server6
sudo cp /etc/default/isc-dhcp-server /etc/default/isc-dhcp-server6
```
Alter /etc/init.d/isc-dhcp-server6
```
#!/bin/sh
#
#

### BEGIN INIT INFO
# Provides:          isc-dhcp-server6
. . .

# Short-Description: DHCPv6 server

. . .

DHCPD_DEFAULT="${DHCPD_DEFAULT:-/etc/default/isc-dhcp-server6}"

. . .


NAME6=dhcpd6
DESC6="ISC DHCPv6 server"
# fallback to default config file
DHCPDv6_CONF=${DHCPD_CONF:-/etc/dhcp/dhcpd6.conf}

DHCPDv6_PID="${DHCPDv6_PID:-/var/run/dhcpd6.pid}"

. . .
```
Alter /etc/default/isc-dhcp-server6
```

DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

DHCPDv6_PID=/var/run/dhcpd6.pid

OPTIONS="-6"

INTERFACESv6="enp0s8"
```

Restart the service and verify
```
systemctl daemon-reload
systemctl stop isc-dhcp-server
systemctl disable isc-dhcp-server

systemctl restart isc-dhcp-server6
systemctl enable isc-dhcp-server6
systemctl status isc-dhcp-server6 --no-pager

ss -tulpn | grep 547
```

Leases will be stored in /var/lib/dhcp/dhcpd6.leases.



## Client
Alter /etc/network/interface

```
iface enp0s3 dhcp
 request_prefix 1
```

Verify
```
ip -6 addr show enp0s3

cat /etc/resolv.conf
```
