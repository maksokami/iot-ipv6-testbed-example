```
ip link set enp0s3 down
ip address add df:0:0:1::5/96 dev enp0s3
ip route add 64:ff9b::/96 via df:0:0:1::1
ip link set enp0s3 up
ip -6 route add default via df:0:0:1::1
```
Verify
```
ip addr
ip -6 route
```
