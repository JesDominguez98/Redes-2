### EDIFICIO EXTERNO CONFIGURATION

## DIRECCIONAMIENTO IP
enable
configure terminal
hostname Sucursal
interface fa 0/0
ip address 197.168.10.1 255.255.255.0
no sh
exit
interface se 0/0/0
ip address 20.0.0.2 255.255.255.252
no sh
exit

## DHCP
ip dhcp pool datosS
network 197.168.10.0 255.255.255.0
default-router 197.168.10.1
ip dhcp excluded-address 197.168.10.1 197.168.10.11

## ENRUTAMIENTO
router ospf 100
network 20.0.0.0 0.0.0.3 area 0
network 197.168.10.0 0.0.0.255 area 0

ip route 197.168.10.0 255.255.255.0 10.0.0.1

## VPN CONFIGURATION
crypto isakmp policy 10
encryption aes
hash sha
authentication pre-share
group 2
exit

crypto isakmp key mykey address 20.0.0.1

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
exit


crypto map CMAP 10 ipsec-isakmp
set peer 20.0.0.1
set transform-set VPN-SET
match address 101
exit

access-list 101 permit ip 197.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 101 permit ip 197.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255

interface se 0/0/0
crypto map CMAP
exit