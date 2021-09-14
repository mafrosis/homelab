
Setup global eggs
PPPoE config
PoE config
Create VLANs
DNS forwarding to switch0, switch0.20, switch0.30
Create DHCP servers


DNS
---------

### Use DNSmasq to resolve local statically mapped hostnames
```
set service dhcp-server use-dnsmasq enable
```

### Set the upstream DNS name server to pihole
```
delete service dns forwarding name-server
set service dns forwarding name-server 192.168.1.198
```

### Hand out the router as DNS resolver to clients
```
set service dhcp-server shared-network-name LAN subnet 192.168.1.0/24 dns-server 192.168.1.1
set service dhcp-server shared-network-name IOT subnet 192.168.20.0/24 dns-server 192.168.1.1
set service dhcp-server shared-network-name Guest subnet 192.168.30.0/24 dns-server 192.168.1.1
```

### Remove any global nameserver config
```
delete system name-server
```


### References

* https://help.ui.com/hc/en-us/articles/115002673188-EdgeRouter-DHCP-Server-Using-Dnsmasq
* https://community.ui.com/questions/EdgeRouter-X-DNS-local-hosts-resolved-using-Dnsmasq/dd3b1d6a-b018-4c31-bda0-b5ddf464392d#answer/3509ac8e-ec64-4a0f-9e01-0eb8b57d95a8
