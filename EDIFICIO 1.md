### EDIFICIO 1

enable
configure terminal
hostname Edificio1
banner motd #***Acceso Restringido! Solo personal autorizado!***#

## DIRECCIONAMIENTO IP
interface fa 0/0
ip address 192.168.10.1 255.255.255.0
no sh
exit

interface fa 0/1 
ip address 192.168.20.1 255.255.255.0
no sh
exit

interface se 0/0/0
ip address 10.0.0.10 255.255.255.252
no sh
exit

interface se 0/0/1
ip address 10.0.0.13 255.255.255.252
no sh
exit

interface se 0/1/0
ip address 10.0.0.17 255.255.255.252
no sh
exit

## DHCP
ip dhcp pool Oficina1
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1 
ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp pool Oficina2
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
ip dhcp excluded-address 192.168.20.1 192.168.20.10

## ENRUTAMIENTO

EIGRP
router eigrp 100
network 192.168.10.0 0.0.0.255
network 192.168.20.0 0.0.0.255
network 10.0.0.8 0.0.0.3
network 10.0.0.12 0.0.0.3
network 10.0.0.16 0.0.0.3
no auto-summary

router ospf 100
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 are 0
network 10.0.0.8 0.0.0.3 area 0
network 10.0.0.12 0.0.0.3 area 0
network 10.0.0.16 0.0.0.3 area 0
passive-interface fa 0/0
passive-interface fa 0/1

## REDISTRIBUCION
router eigrp 100
redistribute ospf 100 metric 1658031 514560 255 255 1500
exit
router ospf 100
redistribute eigrp 100 metric 65 subnets
network 10.0.0.12 0.0.0.3 area 0
network 10.0.0.8 0.0.0.3 area 0
network 10.0.0.16 0.0.0.3 area 0

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
ip domain-name edificio1.com
crypto key generate rsa

# AAA
aaa new-model
tacacs-server host 193.168.10.5 key claveT
radius-server host 193.168.10.7 key claveR
aaa authentication enable default group tacacs+ group radius local
aaa authentication login default group tacacs+ group radius local