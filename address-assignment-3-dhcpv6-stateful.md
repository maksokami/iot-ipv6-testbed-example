# DHCPv6 stateful
What's different compared to stateless DHCPv6:
- Clients are allocated unique IPv6 addresses (client don't have to use and show EUI addresses that uncover their MAC address and therefore manufacturere)
- Server/router keeps lease table to keep track of assigned addresses
- Server/router can do prefix delegration (not only tell the client how to configure WAN interface but tell a list of prefixes Client can assign to it's LAN side devices, in case the Client device wants to be a router)

![DHCPv6 stateful](https://www.cisco.com/c/dam/en/us/support/docs/ip/ip-version-6-ipv6/213272-troubleshoot-ipv6-dynamic-address-assign-09.png)

## Client
Delete static IPv6 address
```
# Delete any static addresses
ip address delete fd:0:0:1::5/96 dev enp0s3
# Renew IPv6 address
dhclient -6 -r enp0s3
dhclient -6 enp0s3

# You will see a unique address in fd:0:0:1:: allocated by the server
ip -6 addr show enp0s3
```
Address: Auto (Server allocates unique address)

DNS configuration: Auto

Default gateway configuration: Auto
