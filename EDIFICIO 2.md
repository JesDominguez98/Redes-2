### EDIFICIO 2

enable
configure terminal
hostname Edificio1
banner motd #***Acceso Restringido! Solo personal autorizado!***#

## DIRECCIONAMIENTO IP
interface fa 0/0
ip address 192.168.30.1 255.255.255.0
no sh
exit

interface fa 0/1 
ip address 192.168.40.1 255.255.255.0
no sh
exit

interface se 0/0/0
ip address 10.0.0.9 255.255.255.252
no sh
exit

interface se 0/0/1
ip address 10.0.0.6 255.255.255.252
no sh
exit

interface fa 1/0
ip address 10.0.0.21 255.255.255.252
no sh
exit


## DHCP
ip dhcp pool Oficina3
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1 
dns-server 8.8.8.5
ip dhcp excluded-address 192.168.30.1 192.168.30.10

ip dhcp pool Oficina4
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 8.8.8.5
ip dhcp excluded-address 192.168.40.1 192.168.40.10

## ENRUTAMIENTO
OSPF
router ospf 100
network 192.168.40.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 10.0.0.8 0.0.0.3 area 0
network 10.0.0.4 0.0.0.3 area 0
network 10.0.0.20 0.0.0.3 area 0
passive-interface fa 0/0
passive-interface fa 0/1


RIP
router rip
version 2
network 192.168.30.0 
network 192.168.40.0
network 10.0.0.4
network 10.0.0.8
network 10.0.0.20

## REDISTRIBUCION
router ospf 100
redistribute rip subnets
exit
router rip 
version 2
redistribute ospf 100 metric 2
exit


## DENEGAR COMUNICACION ENTRE OFICINAS
OFICINA 1 CON OFICINA 3
access-list 1 deny 192.168.10.0 0.0.0.255
access-list 1 permit any 
interface fa 0/0
ip access-group 1 out

## CONFIGURAR VPN
enable
configure terminal
interface se 0/1/0
ip address 20.0.0.1 255.255.255.252
no sh

router ospf 100
network 20.0.0.0 0.0.0.3 area 0
exit

crypto isakmp policy 10
encryption aes
hash sha
authentication pre-share
group 2
exit

crypto isakmp key mykey address 20.0.0.2

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac


crypto map VPN-MAP 10 ipsec-isakmp
set peer 20.0.0.2
set transform-set VPN-SET
match address 101
exit

access-list 101 permit ip 192.168.40.0 0.0.0.255 197.168.10.0 0.0.0.255
access-list 101 permit ip 192.168.30.0 0.0.0.255 197.168.10.0 0.0.0.255

interface se 0/1/0
crypto map VPN-MAP
exit

## SEGURIDAD
# LOCAL
enable secret pass
service password-encryption
username admin privilege 15 secret redes123

line console 0
password consolePass
login local
exit

# TELNET
line vty 0 4
login local
transport input ssh
exit

# SSH
ip domain-name edificio2.com
crypto key generate rsa

# AAA
aaa new-model
tacacs-server host 193.168.10.5 key claveT
radius-server host 193.168.10.7 key claveR
aaa authentication enable default group tacacs+ group radius local
aaa authentication login default group tacacs+ group radius local