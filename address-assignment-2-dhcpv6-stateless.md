# DHCPv6 stateless
## Router

### Option 1 - RADVD 

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

### Option 2 - ISC server
Install DHCP server
```
apt install -y isc-dhcp-server
```
Alter config /etc/dhcp/dhcpd6.conf
```

```



## Client
Delete static IPv6 address
```
# Delete any static addresses
ip address delete fd:0:0:1::5/96 dev enp0s3
# Renew IPv6 address
dhclient -6 -r enp0s3
dhclient -6 enp0s3

# You will see address generated in fd:0:0:1 subnet from client's MAC address:
ip -6 addr show enp0s3
```
