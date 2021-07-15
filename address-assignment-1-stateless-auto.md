# Stateless autoconfiguration

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
Delete static IPv6 address
```
ip address delete fd:0:0:1::5/96 dev enp0s3

# You will see address generated in fd:0:0:1 subnet from client's MAC address:
ip -6 addr show enp0s3
```

DNS configuration: Manual
Default gateway configuration: Manual
