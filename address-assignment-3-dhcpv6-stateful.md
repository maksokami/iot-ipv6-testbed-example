# DHCPv6 stateful



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
