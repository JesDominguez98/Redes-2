### EDIFICIO 4

enable
configure terminal
hostname Edificio1
banner motd #***Acceso Restringido! Solo personal autorizado!***#

## DIRECCIONAMIENTO IP
interface fa 0/0
ip address 130.168.40.1 255.255.255.0
no sh
exit

interface fa 0/1 
ip address 130.168.30.1 255.255.255.0
no sh
exit

interface se 0/0/0
ip address 10.0.0.1 255.255.255.252
no sh
exit

interface se 0/0/1
ip address 10.0.0.14 255.255.255.252
no sh
exit


## DHCP
ip dhcp pool Oficina7
network 130.168.40.0 255.255.255.0
default-router 130.168.40.1
dns-server 8.8.8.5
ip dhcp excluded-address 130.168.40.1 130.168.40.10

ip dhcp pool Oficina8
network 130.168.30.0 255.255.255.0
default-router 130.168.30.1
dns-server 8.8.8.5
ip dhcp excluded-address 130.168.30.1 130.168.30.10

## ENRUTAMIENTO
EIGRP
router eigrp 100
network 130.168.40.0 0.0.0.255
network 130.168.30.0 0.0.0.255
network 10.0.0.0 0.0.0.3
network 10.0.0.12 0.0.0.3
no auto-summary

BGP
router bgp 400
network 130.168.40.0 mask 255.255.255.0
network 130.168.30.0 mask 255.255.255.0
network 10.0.0.0 mask 255.255.255.252
network 10.0.0.12 mask 255.255.255.252
neighbor 10.0.0.2 remote-as 300

## REDISTRIBUCION
router eigrp 100
redistribute bgp 400 metric 1658031 514560 255 255 1500
exit
router bgp 400
redistribute eigrp 100
network 10.0.0.0 mask 255.255.255.252

## DENEGAR COMUNICACION ENTRE OFICINAS
OFICINA 4 CON OFICINA 7
access-list 1 deny 192.168.40.0 0.0.0.255
access-list 1 permit any
interface fa 0/0
ip access-group 1 out

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
ip domain-name edificio4.com
crypto key generate rsa

# AAA
aaa new-model
tacacs-server host 193.168.10.5 key claveT
radius-server host 193.168.10.7 key claveR
aaa authentication enable default group tacacs+ group radius local
aaa authentication login default group tacacs+ group radius local