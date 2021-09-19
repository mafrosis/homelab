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
