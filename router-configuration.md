
> Do not use 2001:db8:: addresses for actual deployments. They are reserved for documentation.
> 64:ff9b:: is reserved for address translation. It best to keep it unaltered.

## NAT64 options:
- Tayga
- Jool (selected)
- map646
- WrapSix
- Ecdysis

Performance comparison: https://www.sciencedirect.com/science/article/pii/S0140366419311569 ; https://link.springer.com/article/10.1007/s11235-020-00681-x

https://www.hardill.me.uk/wordpress/2020/02/05/ipv6-only-network-with-ipv4-access-dns64-nat64/
https://apprize.best/linux/ubuntu/17.html


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
sudo apt install -y bind9 dnsutils
```

Modify /etc/bind/named.conf.options
```
acl translator {
	# Please list all the translator's addresses here.
	localhost;
};
acl dns64-good-clients {
	# Please list here the clients that should be allowed to query
	# the DNS64 service.
	# "localnets" is a convenient moniker for devices sharing a
	# network with our DNS64.
	localnets;
};

options {
	# Ubuntu BIND's default options.
	# Might need to tweak this if you use some other distribution.
	directory "/var/cache/bind";
	dnssec-validation auto;
        auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };

	# Make sure our nameserver is not abused by external
	# malicious users.
	allow-query { dns64-good-clients; };

	# This enables DNS64.
	# "64:ff9b::/96" has to be the same as Jool's `pool6`.
	dns64 df00:0:0:0001::/96 {
		# Though serving standard DNS to the translator device
		# is perfectly normal, we want to exclude it from DNS64.
		# Why? Well, one reason is that the translator is
		# already connected to both IP protocols, so its own
		# traffic doesn't need 64:ff9b for anything.
		# But a more important reason is that Jool can only
		# translate on PREROUTING [0]; it specifically excludes
		# local traffic. If the Jool device itself attempts to
		# communicate with 64:ff9b, it will fail.
		# Listing !translator before our good clients here
		# ensures the translator is excluded from DNS64, even
		# when it belongs to the client networks.
		clients { !translator; dns64-good-clients; };

		# Other options per prefix (if you need them) here.
		# More info here: https://kb.isc.org/article/AA-01031
	};
};
```
```
service bind9 restart
```

## Stateful NAT64
Stateful NAT64 is selected, as the purpose is to provide internet connectivity to a local IPv6 network using minimum ipv4 addresses (similar to NAPT).

      
      
