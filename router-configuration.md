
> Do not use 2001:db8:: addresses for actual deployments. They are reserved for documentation.

> 64:ff9b:: is reserved for address translation - keep it.

## NAT64 options:
- Tayga
- Jool (selected)
- map646
- WrapSix
- Ecdysis

Performance comparison: https://www.sciencedirect.com/science/article/pii/S0140366419311569 ; https://link.springer.com/article/10.1007/s11235-020-00681-x

https://www.hardill.me.uk/wordpress/2020/02/05/ipv6-only-network-with-ipv4-access-dns64-nat64/
https://apprize.best/linux/ubuntu/17.html
### Enable IPv6 (if disabled)
```sh
echo "net.ipv6.conf.all.disable_ipv6 = 0" | tee /etc/sysctl.d/disable-ipv6.conf
echo "net.ipv6.conf.default.disable_ipv6 = 0" | tee -a /etc/sysctl.d/disable-ipv6.conf
sysctl -w net.ipv6.conf.all.disable_ipv6=0
sysctl -w net.ipv6.conf.default.disable_ipv6=0
sysctl -w net.ipv6.route.flush=1
```

### IP forwarding 
IP forwarding is required to forward IPv6 packets to the NAT64:
```
echo "net.ipv4.ip_forward=1" | tee -a /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" | tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Configure interfaces
#### Router
Modify /etc/network/interface
```
# WAN ipv4
iface enp0s3 inet dhcp

#LAN ipv6
iface enp0s8 inet6 static
address df::1:0:0:0:1
netmask 64
```
```
systemctl restart networking
```
Alternatively
```
ip link set eth1 up
ip address add x.x.x.x/24 dev eth1

ip link set enp0s8 down
ip address add df::1:0:0:0:1/96 dev enp0s8
ip link set enp0s8 up
```
#### Clients
```
ip link set enp0s3 down
ip address add df::1:0:0:0:5/96 dev enp0s3
ip route add 64:ff9b::/96 via df::1:0:0:0:1
ip link set enp0s8 up
```
### Install jool
https://www.jool.mx/en/install.html

> Note that DKMS is kernel dependent

Compiling from source
```
apt install build-essential pkg-config
apt install linux-headers-$(uname -r)
apt install libnl-genl-3-dev libxtables-dev dkms
apt install tar git autoconf libtool wget curl apt-transport-https
cd /tmp

wget https://jool.mx/download/jool-4.1.5.tar.gz
tar -xzf jool-4.1.5.tar.gz

dkms install jool-4.1.5/
cd jool-4.1.5/
./configure
make
make install
```

Configure jool (iptables)
https://www.jool.mx/en/run-nat64.html

```
modprobe jool
jool instance add "nat64test" --iptables  --pool6 64:ff9b::/96
ip6tables -t mangle -A PREROUTING -j JOOL --instance "nat64test"
iptables  -t mangle -A PREROUTING -j JOOL --instance "nat64test"
```

### Install DNS
https://www.jool.mx/en/dns64.html

```
sudo apt update
sudo apt install -y bind9 bind9utils libcomerr2
# install if you want to use dig for testing:
sudo apt install dnsutils
```

Modify /etc/bind/named.conf.options
```
acl any6 {
  # This ACL matches all ipv6 addresses
  ::0/0;
};
acl translator {
	# Please list all the translator's addresses here.
	localhost;
};
acl dns64-good-clients {
	# Please list here the clients that should be allowed to query
	# the DNS64 service.
	localnets;
};

options {
	directory "/var/cache/bind";
	dnssec-validation auto;
        auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };

	# Make sure our nameserver is not abused by external users
	allow-query { dns64-good-clients; };

	# This enables DNS64.
	# "64:ff9b::/96" is a Jool's `pool6`.
	dns64 64:ff9b::/96 {
	        suffix ::
		# Allow only local clients to use DNS. Prevent request from translator, which will fail anyway
		clients { !translator; dns64-good-clients; };
		# This line is required to resolve all DNS to NAT64 generated address in 'pool6'. 
		# You can remove it if you have a true dual-stack (client has both pure ipv4 and ipv6 connectivity). When removed, DNS for sites with ipv6 will not be modified, and they will not be NATed
		exclude { any6; };
	};
};
```
```
service bind9 restart
```

## Stateful NAT64
Stateful NAT64 is selected, as the purpose is to provide internet connectivity to a local IPv6 network using minimum ipv4 addresses (similar to NAPT).

      
      
