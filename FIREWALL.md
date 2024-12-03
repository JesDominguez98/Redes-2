### FIREWALL CONFIGURATION

## DIRECCIONAMIENTO IP
enable
configure terminal
hostname FIREWALL
enable password redes2
username cisco password redes123

interface gi 1/1
no sh
nameif INSIDE
security-level 100
ip address 10.0.0.22 255.255.255.252
exit

interface gi 1/2
no sh
nameif DMZ
security-level 60
ip address 10.0.0.25 255.255.255.252
exit

interface gi 1/3
no sh
nameif OUTSIDE
security-level 0
ip address 10.0.0.29 255.255.255.252
exit

## ENRUTAMIENTO
router ospf 100
network 192.168.10.0 255.255.255.0 area 0
network 192.168.20.0 255.255.255.0 area 0
network 192.168.30.0 255.255.255.0 area 0
network 192.168.40.0 255.255.255.0 area 0
network 130.168.10.0 255.255.255.0 area 0
network 130.168.20.0 255.255.255.0 area 0
network 130.168.30.0 255.255.255.0 area 0
network 130.168.40.0 255.255.255.0 area 0
network 10.0.0.20 255.255.255.252 area 0
network 10.0.0.24 255.255.255.252 area 0
network 10.0.0.28 255.255.255.252 area 0
exit
route 0UTSIDE 0.0.0.0 0.0.0.0 10.0.0.30

## CONFIGURACION DMZ
access-list DMZ-ACCESO extended permit icmp any any
access-list DMZ-ACCESO extended permit tcp any any
access-list DMZ-ACCESO extended permit udp any any
access-group DMZ-ACCESO out interface INSIDE

## CONFIGURACION OUTSIDE
access-list INTERNET-ACCESS extended permit icmp any any
access-list INTERNET-ACCESS extended permit tcp any any
access-group INTERNET-ACCESS in interface OUTSIDE