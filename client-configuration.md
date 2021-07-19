# Client configuration
### Enable IPv6 (if disabled)
```sh
echo "net.ipv6.conf.all.disable_ipv6 = 0" | tee /etc/sysctl.d/disable-ipv6.conf
echo "net.ipv6.conf.default.disable_ipv6 = 0" | tee -a /etc/sysctl.d/disable-ipv6.conf
sysctl -w net.ipv6.conf.all.disable_ipv6=0
sysctl -w net.ipv6.conf.default.disable_ipv6=0
sysctl -w net.ipv6.route.flush=1
```

### Configure static IP and routes
```sh
ip link set enp0s3 down
ip address add fd:0:0:1::5/96 dev enp0s3
ip route add 64:ff9b::/96 via fd:0:0:1::1
ip link set enp0s3 up
ip -6 route add default via fd:0:0:1::1
```

### Configure DNS resolution
Alter /etc/resolv.conf. Router should be the client's DNS server.
```
nameserver fd:0:0:1::1
```

### Verify
You will be able to ping and acess ipv4 sites (or both ipv4 and ipv6 sites, if your lab runs with dual-stack).
IPv4 sites will resolve to the NAT pool starting from 64:ff9b::
```sh
ip addr
ip -6 route

# Test without DNS (ping google)
ping 64:ff9b::8.8.8.8

# Test with DNS
ping example.com
ping google.com
```
