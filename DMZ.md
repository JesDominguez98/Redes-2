### DMZ CONFIGURATION

## DIRECCIONAMIENTO IP
enable
configure terminal
hostname DMZ

interface fa 0/0 
ip address 193.168.10.1 255.255.255.0
no sh
exit

interface fa 0/1
ip address 10.0.0.26 255.255.255.252
no sh
exit

## ENRUTAMIENTO
router ospf 100
network 193.168.10.0 0.0.0.255 area 0
network 10.0.0.24 0.0.0.3 area 0
exit
