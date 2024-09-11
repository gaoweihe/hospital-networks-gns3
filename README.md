# Hospital Networks 

c7200 

```
enable
configure terminal
interface g2/0
ip address dhcp
no shutdown
exit

interface g4/0
ip address 10.0.0.1 255.255.255.0
no shutdown
exit

ip dhcp pool lan
network 10.0.0.0 255.255.255.0
default-router 10.0.0.1
dns-server 8.8.8.8 1.1.1.1
lease 7
exit

interface g2/0
ip nat outside

interface g4/0
ip nat inside

access-list 1 permit 10.0.0.0 0.0.0.255
ip nat inside source list 1 interface g2/0 overload

exit

write memory
```

switch 

