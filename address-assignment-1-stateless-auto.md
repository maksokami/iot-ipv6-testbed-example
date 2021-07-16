# Stateless autoconfiguration (SLAAC)

## Router

```
apt install radvd
```
Create /etc/radvd.conf. Bind to Client facing interface with IPv6
```
interface enp0s8 {
  AdvSendAdvert on;
  MinRtrAdvInterval 3; 
  MaxRtrAdvInterval 10;
  prefix fd:0:0:1::/64 {
    AdvOnLink on; 
    AdvAutonomous on; 
    AdvRouterAddr on; 
  };
};
```
Restart the service
```
systemctl restart radvd
systemctl status radvd --no-pager
```

## Client
Alter /etc/network/interface
```
iface enp0s3 inet auto
  request prefix 1
```

Verify
```
ip -6 addr show enp0s3
```

-----

Network Prefix: Auto

Host Address: Client machine derives based on MAC (EUI-64)

Default gateway configuration: Auto (Taken from router advertisement RA packet)

DNS configuration: Manual
