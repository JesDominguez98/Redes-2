### OUTSIDE CONFIGURATION

## DIRECCIONAMIENTO IP
enable
configure terminal
hostname OUTSIDE

interface fa 0/0
ip address 8.8.8.1 255.0.0.0
no sh
exit

interface fa 0/1
ip address 10.0.0.30 255.255.255.252
no sh
exit

## ENRUTAMIENTO
router ospf 100
network 8.8.8.0 0.255.255.255 area 0
network 10.0.0.28 0.0.0.3 area 0
exit
