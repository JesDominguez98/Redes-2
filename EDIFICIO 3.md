### EDIFICIO 3

enable
configure terminal
hostname Edificio3
banner motd #***Acceso Restringido! Solo personal autorizado!***#

## DIRECCIONAMIENTO IP
interface fa 0/0
ip address 130.168.10.1 255.255.255.0
no sh
exit

interface fa 0/1 
ip address 130.168.20.1 255.255.255.0
no sh
exit

interface se 0/0/0
ip address 10.0.0.2 255.255.255.252
no sh
exit

interface se 0/0/1
ip address 10.0.0.5 255.255.255.252
no sh
exit

interface se 0/1/0
ip address 10.0.0.18 255.255.255.252
no sh
exit

## DHCP
ip dhcp pool Oficina6
network 130.168.10.0 255.255.255.0
default-router 130.168.10.1
dns-server 8.8.8.5
ip dhcp excluded-address 130.168.10.1 130.168.10.10

ip dhcp pool Oficina5
network 130.168.20.0 255.255.255.0
default-router 130.168.20.1
dns-server 8.8.8.5
ip dhcp excluded-address 130.168.20.1 130.168.20.10

## ENRUTAMIENTO Y REDISTRIBUCION
router rip
version 2
network 130.168.10.0
network 130.168.20.0
network 10.0.0.0
network 10.0.0.4
network 10.0.0.16
redistribute eigrp 100
redistribute ospf 100

router ospf 100
network 130.168.10.0 0.0.0.255 area 0
network 130.168.20.0 0.0.0.255 area 0
network 10.0.0.0 0.0.0.3 area 0
network 10.0.0.4 0.0.0.3 area 0
network 10.0.0.16 0.0.0.3 area 0
passive-interface fa 0/0
passive-interface fa 0/1
redistribute rip subnets
redistribute eigrp 100 metric 65 subnets

router eigrp 100
network 130.168.10.0 0.0.0.3
network 130.168.20.0 0.0.0.3 
network 10.0.0.0 0.0.0.3 
network 10.0.0.4 0.0.0.3 
network 10.0.0.16 0.0.0.3
redistribute rip
redistribute ospf 100 metric 1658031 514560 255 255 1500

## DENEGAR COMUNICACION ENTRE OFICINAS
OFICINA 2 CON OFICINA 6
access-list 1 deny 192.168.20.0 0.0.0.255
access-list 1 permit any
interface fa 0/0
ip access-group 1 out

OFICINA 8 CON OFICINA 5
access-list 30 deny 130.168.30.0 0.0.0.255
access-list 30 permit any
interface fa 0/1
ip access-group 30 out

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
ip domain-name edificio3.com
crypto key generate rsa

# AAA
aaa new-model
tacacs-server host 193.168.10.5 key claveT
radius-server host 193.168.10.7 key claveR
aaa authentication enable default group tacacs+ group radius local
aaa authentication login default group tacacs+ group radius local