# Stateless autoconfiguration

## Router

```
apt install radvd
```
Create /etc/radvd.conf. Bind to Client facing interface with IPv6
```
interface enp0s8 {
  prefix fd:0:0:1::/64 {
  };
};
```

## Client
Delete static IPv6 address
```
ip address delete fd:0:0:1::5/96 dev enp0s3

# You will see address generated in fd:0:0:1 subnet from client's MAC address:
ip -6 addr show enp0s3
```
