```
ip link set enp0s3 down
ip address add df::1:0:0:0:5/96 dev enp0s3
ip route add 64:ff9b::/96 via df::1:0:0:0:1
ip link set enp0s3 up
ip -6 route add default via df::1:0:0:0:1
```
Verify
```
ip addr
ip -6 route
```
