# Stateless autoconfiguration (SLAAC)

## What to expect?
- Client will learn correct prefix from the Router automatically (taken from RA packet)
- Client will learn router's IP and configure it as default gateway automatically (taken from RA packet)
- Client will use EUI-64 to generate a unique IP address automatically
- DNS and NTP will not be configured automatically 

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
  AdvManagedFlag off;
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
iface enp0s3 auto
 accept_ra 2
```

Verify
```
ip -6 addr show enp0s3
```

> **Why accept_ra 2?** 
> We consider for this lab that our IoT device/client required IP forwarding to be enabled. By default Linux device will disable SLAAC interface configuration if it considers itself a router (ip forwarding enabled). To enable SLAAC we need to choose option "2" for accept_ra to override 'if forwarding' behavior.
Read more here: https://strugglers.net/~andy/blog/2011/09/04/linux-ipv6-router-advertisements-and-forwarding/

