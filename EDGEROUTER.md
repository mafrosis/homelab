Edgerouter
==========

Personal setup plus a couple of tricks.


Global
----------

Disable the Ubiquiti call home:
```
set system analytics-handler send-analytics-report false
set system crash-handler send-crash-report false
```

Set the global domain:
```
set system domain-name eggs
```

Set the router timezone:
```
set system time-zone Australia/Melbourne
```

Disable Ubiquiti UNMS (UISP) management service:
```
set service unms disable
```


VLANs
----------

### Disable VLAN aware switching

VLAN tags are applied based on which wifi SSID traffic originates from, and the router does not tag
traffic for specific VLANs based on incoming port.
```
set interfaces switch switch0 switch-port vlan-aware disable
```

### Create VLAN VIF interfaces
```
set interfaces switch switch0 vif 20 address 192.168.20.1/24
set interfaces switch switch0 vif 20 description IOT
set interfaces switch switch0 vif 30 address 192.168.30.1/24
set interfaces switch switch0 vif 30 description Guest
```

### Create DHCP servers for each VLAN

Configure the a DHCP server for `switch0`, and create additional ones for each VLAN.
```
set service dhcp-server shared-network-name LAN subnet 192.168.1.0/24 start 192.168.1.100 stop 192.168.1.254
set service dhcp-server shared-network-name LAN subnet 192.168.1.0/24 default-router 192.168.1.1

set service dhcp-server shared-network-name IOT subnet 192.168.20.0/24 start 192.168.20.100 stop 192.168.20.254
set service dhcp-server shared-network-name IOT subnet 192.168.20.0/24 default-router 192.168.20.1

set service dhcp-server shared-network-name Guest subnet 192.168.30.0/24 start 192.168.30.100 stop 192.168.30.254
set service dhcp-server shared-network-name Guest subnet 192.168.30.0/24 default-router 192.168.30.1
```

### Set Unifi controller IP for LAN

This apparently helps with AP discovery etc.
```
set service dhcp-server shared-network-name LAN subnet 192.168.1.0/24 unifi-controller 192.168.1.198
```

### References

* https://help.ui.com/hc/en-us/articles/218889067-EdgeRouter-How-to-Create-a-Guest-LAN-Firewall-Rule

### CLI

```
$ show dhcp leases
IP address      Hardware Address   Lease expiration     Pool       Client Name
----------      ----------------   ----------------     ----       -----------
192.168.1.133   c6:94:83:xx:xx:47  2021/09/15 10:03:30  LAN        Pixel-5
192.168.1.166   e0:63:da:xx:xx:35  2021/09/14 13:28:36  LAN        KitchenAP
192.168.1.254   1e:85:ce:xx:xx:7d  2021/09/14 17:08:09  LAN        blackmi1214KD7V

$ clear dhcp lease ip 192.168.1.254
[ ok ] Stopping dnsmasq (via systemctl): dnsmasq.service.
[ ok ] Starting dnsmasq (via systemctl): dnsmasq.service.
```


DNS
----------

DNS forwarding to switch0, switch0.20, switch0.30

### Use DNSmasq to resolve local statically mapped hostnames
```
set service dhcp-server use-dnsmasq enable
```

### Set a large local DNS cache size
```
set service dns forwarding cache-size 500
```

### Listen on all VLANs for DNS traffic
```
delete service dns forwarding listen-on
set service dns forwarding listen-on switch0
set service dns forwarding listen-on switch0.20
set service dns forwarding listen-on switch0.30
```

### Set the upstream DNS name server to pihole

Tell the DNS forwarder to use the system DNS upstream, and set the edgreouter global upstream DNS
to the pihole.
```
delete service dns forwarding name-server
set service dns forwarding system
set system name-server 192.168.1.198
```

### Use public DNS for the Guest network
```
set service dhcp-server shared-network-name Guest subnet 192.168.30.0/24 dns-server 1.1.1.1
```

### References

* https://help.ui.com/hc/en-us/articles/115002673188-EdgeRouter-DHCP-Server-Using-Dnsmasq
* https://help.ui.com/hc/en-us/articles/115010913367-EdgeRouter-DNS-Forwarding-Setup-and-Options
* https://community.ui.com/questions/EdgeRouter-X-DNS-local-hosts-resolved-using-Dnsmasq/dd3b1d6a-b018-4c31-bda0-b5ddf464392d
* https://community.ui.com/questions/EdgeRouter-DNS-different-than-DHCP-DNS/581612f0-2b11-453c-8f25-51a83be204db

### CLI

```
$ show dns forwarding nameservers
-----------------------------------------------
   Nameservers configured for DNS forwarding
-----------------------------------------------
192.168.1.198 available via 'optionally configured'

-----------------------------------------------
 Nameservers NOT configured for DNS forwarding
-----------------------------------------------
203.134.24.70 available via 'ppp pppoe0'
203.134.26.70 available via 'ppp pppoe0'
```


Port Forwarding
----------

### Setup auto-firewall, WAN and LAN interfaces

```
set port-forward auto-firewall enable
set port-forward wan-interface eth0
set port-forward lan-interface eth1
```

### Turn off Hairpin NAT

This enables clients to use the external IP to access internal hosts. This isn't used; port forwarding
sends traffic onto the correct host.

```
set port-forward hairpin-nat disable
```

### Setup a port forward rule

```
set port-forward rule 1 description ca
set port-forward rule 1 forward-to address 192.168.1.198
set port-forward rule 1 forward-to port 8443
set port-forward rule 1 original-port 443
set port-forward rule 1 protocol tcp
```

### References

* https://help.ui.com/hc/en-us/articles/217367937-EdgeRouter-Port-Forwarding
* https://help.ui.com/hc/en-us/articles/204952134



mDNS Repeater for Avahi/Chromecast
----------

Enable the `repeater` on LAN and IOT interfaces. Do not use `reflector`, as that floods mDNS to all
interfaces including WAN.
```
set service mdns repeater interface switch0
set service mdns repeater interface switch0.20
```
