show parancsok

spanning-tree portfast minden porton ahol nincs switch 
spanning-tree portfast default 





# rtr-telekom-core-01

### Konfiguracio
```
enable 
conf terminal
hostname rtr-telekom-core-01
!
interface Se0/0/0
ip address 195.228.6.1 255.255.255.252
no shutdown
exit
!
interface Se0/0/1
ip address 195.228.6.5 255.255.255.252
no shutdown
exit
!
interface Gig0/0
ip address 195.228.7.1 255.255.255.224
no shutdown
exit
! OSPF konfiguráció
router ospf 6
router-id 6.6.6.1
passive-interface Gig0/0
network 195.228.7.0	0.0.0.31 area 6
network 195.228.6.0	0.0.0.3 area 6
network 195.228.6.4	0.0.0.3 area 6
area 6 authentication message-digest
exit
!
int se0/0/0
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
!
int se0/0/1
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
! Titkosítás
service password-encryption
no ip domain lookup
!
enable secret cisco
!
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH
ip domain-name telekom.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```
### Show parancsok

```
rtr-telekom-core-01#show ip route

[...]

Gateway of last resort is not set

     195.228.6.0/24 is variably subnetted, 4 subnets, 2 masks
C       195.228.6.0/30 is directly connected, Serial0/0/0
L       195.228.6.1/32 is directly connected, Serial0/0/0
C       195.228.6.4/30 is directly connected, Serial0/0/1
L       195.228.6.5/32 is directly connected, Serial0/0/1

```

# rtr-sp-01
```

! első boot után azonnal aktiválni kell a securtity licencet
! el kell fogadni a license agreement-et, ki kell írni a konfigurációt
! majd újra kell indítani a router-t
ena
conf t
license boot module c2900 technology-package securityk9
end
wri mem
reload
!
! indulás után ellenőrizni kell a licence beállításokat
ena
sh ver
!
! konfiguráció építés
conf t
!
hostname rtr-sp-01
!
no ip cef
ipv6 unicast-routing
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 ip address 195.228.7.16 255.255.255.224
 ip nat outside
 duplex auto
 speed auto
 ipv6 nat
 no shutdown
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 ipv6 nat
 no shutdown
!
interface GigabitEthernet0/1.400
 encapsulation dot1Q 400
 ip address 10.7.1.1 255.255.255.0
 ip nat inside
!
interface GigabitEthernet0/1.430
 encapsulation dot1Q 430
 ip address 10.7.30.1 255.255.255.0
 ipv6 address FE80::DB4F:1:1 link-local
 ipv6 enable
ip nat inside
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
! NAT ACL with exceptions
ip access-list extended NAT_ACL
 deny ip 10.7.1.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.7.30.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.7.1.0 0.0.0.255 any
 permit ip 10.7.30.0 0.0.0.255 any
!
! dynamic NAT rules
ip nat pool PNATPOOLSP 195.228.7.16 195.228.7.16 netmask 255.255.255.255
ip nat inside source list NAT_ACL pool PNATPOOLSP overload
!
ip route 0.0.0.0 0.0.0.0 195.228.7.1 
!
!!!!!!!!!!!!!!! IPsec related objects !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
!
crypto ipsec transform-set IKEV1_TRSET esp-aes 256 esp-sha-hmac
!
crypto isakmp policy 100
 encr aes
 authentication pre-share
 group 5
 lifetime 28800
!
! DC oldali ASA
crypto isakmp key cisco address 195.228.5.15
!
! IPsec tunnel specifikus ACL
ip access-list extended IPSEC_DC_ACL
 permit ip 10.7.1.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.7.30.0 0.0.0.255 10.5.20.0 0.0.0.255
!
crypto map CMAP_DC 10 ipsec-isakmp 
 set peer 195.228.5.15
 set transform-set IKEV1_TRSET 
 match address IPSEC_DC_ACL
!
! interfész cryptomap összerendelés
interface GigabitEthernet0/0
 crypto map CMAP_DC
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```


# sw-sp-01

```
en 
conf t
hostname sw-sp-01
sdm prefer dual-ipv4-and-ipv6 default
do reload
y
!
en 
conf t
vlan 400
!
name SP_DESKTOP
exit
!
vlan 430
name SP_SWMAN
exit
!
vlan 440
name SP_NAT
exit
!
int vlan 430
ip address 10.7.30.192 255.255.255.0
ipv6 address FE80::DB4F:1:C0 link-local
!
int g0/1
switchport mode trunk
switchport trunk native vlan 440
switchport trunk allowed vlan 400,430
exit
!
int f0/1
switchport mode access 
switchport access vlan 400
spanning-tree portfast
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int fa0/2
switchport mode access 
switchport access vlan 430
spanning-tree portfast
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int range fa0/3-24, gig0/2
sh
exit
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```






# rtr-telekom-core-02

### Konfiguracio

```
enable
configure terminal
hostname rtr-telekom-core-02
!
interface Se0/0/0
ip address 195.228.6.2 255.255.255.252
no shutdown
exit
!
interface Se0/0/1
ip address 195.228.6.9 255.255.255.252
no shutdown
exit
!
interface GigabitEthernet0/0
ip address 195.228.3.1 255.255.255.0
ip address 195.228.3.1 255.255.255.224
no sh
exit
! OSPF konfiguráció
router ospf 6
router-id 6.6.6.2
passive-interface Gig0/0
network 195.228.6.0	0.0.0.3 area 6
network 195.228.6.8	0.0.0.3 area 6
network 195.228.3.0	0.0.0.31  area 6
area 6 authentication message-digest
exit
!
int se0/0/0
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
!
int se0/0/1
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
! Titkosítás
service password-encryption
no ip domain lookup
!
enable secret cisco
!
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH
ip domain-name telekom.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```

### Show parancsok

```
rtr-telekom-core-02#show ip route

[...]

Gateway of last resort is not set

     195.228.3.0/24 is variably subnetted, 2 subnets, 2 masks
C       195.228.3.0/27 is directly connected, GigabitEthernet0/0
L       195.228.3.1/32 is directly connected, GigabitEthernet0/0
     195.228.6.0/24 is variably subnetted, 4 subnets, 2 masks
C       195.228.6.0/30 is directly connected, Serial0/0/0
L       195.228.6.2/32 is directly connected, Serial0/0/0
C       195.228.6.8/30 is directly connected, Serial0/0/1
L       195.228.6.9/32 is directly connected, Serial0/0/1
```



# rtr-kol-01

### Konfiguracio

```
! első boot után azonnal aktiválni kell a securtity licencet
! el kell fogadni a license agreement-et, ki kell írni a konfigurációt
! majd újra kell indítani a router-t
ena
conf t
license boot module c2900 technology-package securityk9
end
wri mem
reload
!
! indulás után ellenőrizni kell a licence beállításokat
enable
show version
!
! konfiguráció építés
configure terminal
hostname rtr-kol-01
!
interface GigabitEthernet0/0
ip address 195.228.3.16 255.255.255.224
no sh
exit
!
int gigabitEthernet 0/1
no sh
!
int gigabitEthernet 0/1.300
encapsulation dot1Q 300
description KOL_DIAK
ip address 10.3.0.1 255.255.255.0
ip helper-address 10.3.0.1
ip nat inside
no sh
exit
!
int gigabitEthernet 0/1.330
encapsulation dot1Q 330
description KOL_SWMAN
ip address 10.3.30.1 255.255.255.0
no sh
exit
!
ip dhcp excluded-address 10.3.0.0 10.3.0.63
ip dhcp excluded-address 10.3.0.128 10.3.0.255
ip dhcp pool DIAK
network 10.3.0.0 255.255.255.0
default-router 10.3.0.1
dns-server 10.5.20.16
exit
ip route 0.0.0.0 0.0.0.0 195.228.3.1
!
int gigabitethernet 0/0
ip nat outside
exit
!
!
int gigabitethernet 0/1
ip nat inside
exit
!
! NAT ACL with exceptions
ip access-list extended NAT_ACL
 deny ip 10.3.0.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.3.30.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.3.0.0 0.0.0.255 any
!
! dynamic NAT rules
ip nat pool PNATPOOLKOL 195.228.3.16 195.228.3.16 netmask 255.255.255.255
ip nat inside source list NAT_ACL pool PNATPOOLKOL overload
!
!
!!!!!!!!!!!!!!! IPsec related objects !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
!
crypto ipsec transform-set IKEV1_TRSET esp-aes 256 esp-sha-hmac
!
crypto isakmp policy 100
 encr aes
 authentication pre-share
 group 5
 lifetime 28800
!
! DC oldali ASA
crypto isakmp key cisco address 195.228.5.15
!
! IPsec tunnel specifikus ACL
ip access-list extended IPSEC_DC_ACL
 permit ip 10.3.0.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.3.30.0 0.0.0.255 10.5.20.0 0.0.0.255
!
crypto map CMAP_DC 10 ipsec-isakmp 
 set peer 195.228.5.15
 set transform-set IKEV1_TRSET 
 match address IPSEC_DC_ACL
!
! interfész cryptomap összerendelés
interface GigabitEthernet0/0
 crypto map CMAP_DC
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```

### Show parancsok

```
rtr-kol-01#show ip route

[...]

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
C       10.3.0.0/24 is directly connected, GigabitEthernet0/1.300
L       10.3.0.1/32 is directly connected, GigabitEthernet0/1.300
C       10.3.1.0/24 is directly connected, GigabitEthernet0/1.301
L       10.3.1.1/32 is directly connected, GigabitEthernet0/1.301
C       10.3.30.0/24 is directly connected, GigabitEthernet0/1.330
L       10.3.30.1/32 is directly connected, GigabitEthernet0/1.330
C       10.3.40.0/24 is directly connected, GigabitEthernet0/1.340
L       10.3.40.1/32 is directly connected, GigabitEthernet0/1.340
     195.228.3.0/24 is variably subnetted, 2 subnets, 2 masks
C       195.228.3.0/27 is directly connected, GigabitEthernet0/0
L       195.228.3.16/32 is directly connected, GigabitEthernet0/0
```

```
rtr-kol-01#show interfaces gigabitEthernet 0/1.300
GigabitEthernet0/1.300 is up, line protocol is up (connected)
    [...]
  Internet address is 10.3.0.1/24 
    [...]
  Encapsulation 802.1Q Virtual LAN, Vlan ID 300
    [...]   

rtr-kol-01#show interfaces gigabitEthernet 0/1.301
GigabitEthernet0/1.301 is up, line protocol is up (connected)
    [...]
  Internet address is 10.3.1.1/24
    [...]
  Encapsulation 802.1Q Virtual LAN, Vlan ID 301
    [...]

rtr-kol-01#show interfaces gigabitEthernet 0/1.330
GigabitEthernet0/1.330 is up, line protocol is up (connected)
    [...]
  Internet address is 10.3.30.1/24
    [...]
  Encapsulation 802.1Q Virtual LAN, Vlan ID 330
    [...]

rtr-kol-01#show interfaces gigabitEthernet 0/1.340
GigabitEthernet0/1.340 is up, line protocol is up (connected)
    [...]
  Internet address is 10.3.40.1/24
    [...]
  Encapsulation 802.1Q Virtual LAN, Vlan ID 340
    [...]

rtr-kol-01#show interfaces gigabitEthernet 0/0
GigabitEthernet0/0 is up, line protocol is up (connected)
    [...]
  Internet address is 195.228.3.16/27
    [...]
```



# sw-kol-01 

### Konfiguracio

```
enable
config terminal
hostname sw-kol-01
!
spanning-tree mode rapid-pvst
!
vlan 300 
name %KOL_DIAK%
exit
!
vlan 330 
name %KOL_SWMAN%
exit
!
vlan 340
name %KOL_NAT%
exit
!
int vlan 330 
ip address 10.3.30.192 255.255.255.0
no sh 
exit
!
ip default-gateway 10.3.30.1
!
int gig0/1
switchport mode trunk
switchport trunk native vlan 340
switchport trunk allowed vlan 300,330
exit
!
int range fa0/1-4
switchport mode access
switchport access vlan 300
int range fa0/5-24
sh
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```

### Show parancsok

```
sw-kol-01#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gig2/1, Gig3/1, Gig4/1
300  %KOL_TANAR%                      active    Gig1/1
301  %KOL_DIAK%                       active    
330  %KOL_SWMAN%                      active    
340  %KOL_NAT%                        active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active 
```



# rtr-telekom-core-03

### Konfiguracio

```
enable
config terminal
hostname rtr-telekom-core-03
!
interface Se0/0/1
ip address 195.228.6.6 255.255.255.252
no shutdown
exit
!
interface Se0/0/0
ip address 195.228.6.13 255.255.255.252
no shutdown
exit
!
interface GigabitEthernet0/0
ip address 195.228.2.1 255.255.255.224
no sh
exit
! OSPF konfiguráció
router ospf 6
router-id 6.6.6.3
passive-interface Gig0/0
network 195.228.6.4	0.0.0.3 area 6
network 195.228.6.12 0.0.0.3 area 6
network 195.228.2.0	0.0.0.31 area 6
area 6 authentication message-digest
exit
!
int  se0/0/0
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
!
int  se0/0/1
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
! Titkosítás
service password-encryption
no ip domain lookup
!
enable secret cisco
!
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH
ip domain-name telekom.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```
### Show parancsok

```
rtr-telekom-core-03#show ip route

[...]

Gateway of last resort is not set

     195.228.2.0/24 is variably subnetted, 2 subnets, 2 masks
C       195.228.2.0/27 is directly connected, GigabitEthernet0/0
L       195.228.2.1/32 is directly connected, GigabitEthernet0/0
     195.228.6.0/24 is variably subnetted, 4 subnets, 2 masks
C       195.228.6.4/30 is directly connected, Serial0/0/1
L       195.228.6.6/32 is directly connected, Serial0/0/1
C       195.228.6.12/30 is directly connected, Serial0/0/0
L       195.228.6.13/32 is directly connected, Serial0/0/0

```



# rtr-gim-01

### Konfiguracio

```
! első boot után azonnal aktiválni kell a securtity licencet
! el kell fogadni a license agreement-et, ki kell írni a konfigurációt
! majd újra kell indítani a router-t
ena
conf t
license boot module c2900 technology-package securityk9
do wri mem
do reload
!
! indulás után ellenőrizni kell a licence beállításokat
ena
sh ver
!
! konfiguráció építés
config terminal
hostname rtr-gim-01
!
int g0/0
ip address 195.228.2.16 255.255.255.224 
no sh
exit
!
int g0/1
ip address 10.2.255.1 255.255.255.252
no sh
exit
!
int g0/2
ip address 10.2.255.5 255.255.255.252
no sh
exit
! OSPF konfiguráció
ip route 0.0.0.0 0.0.0.0 195.228.2.1
router ospf 2
router-id 2.2.2.1
passive-interface Gig0/0
default-information originate
network 195.228.2.0 0.0.0.31 area 2
network 10.2.255.0 0.0.0.3 area 2
network 10.2.255.4 0.0.0.3 area 2
area 2 authentication message-digest
exit
!
int range g0/1-2
ip ospf message-digest-key 2 md5 eiaA8Qsp1i
exit
!
int range g0/1-2
ip nat inside
exit
!
int g0/0
ip nat outside
exit
!
! NAT ACL with VPN exception
ip access-list extended NAT_ACL
 deny ip 10.2.0.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.1.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.3.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.4.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.5.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.10.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.20.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.2.21.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.0.0 0.0.0.255 any
 permit ip 10.2.1.0 0.0.0.255 any
 permit ip 10.2.3.0 0.0.0.255 any
 permit ip 10.2.4.0 0.0.0.255 any
 permit ip 10.2.5.0 0.0.0.255 any
 permit ip 10.2.10.0 0.0.0.255 any
 permit ip 10.2.20.0 0.0.0.255 any
!
! NAT rules
ip nat inside source list NAT_ACL pool PNATPOOLGIM overload
ip nat pool PNATPOOLGIM 195.228.2.16 195.228.2.16 netmask 255.255.255.255


!!!!!!!!!!!!!!! IPsec related objects !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
!
crypto ipsec transform-set IKEV1_TRSET esp-aes 256 esp-sha-hmac
!
crypto isakmp policy 100
 encr aes
 authentication pre-share
 group 5
 lifetime 28800
!
! DC oldali ASA
crypto isakmp key cisco address 195.228.5.15
!
! IPsec tunnel specifikus ACL
ip access-list extended IPSEC_DC_ACL
 permit ip 10.2.0.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.1.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.3.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.4.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.5.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.10.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.20.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.2.21.0 0.0.0.255 10.5.20.0 0.0.0.255
!
crypto map CMAP_DC 10 ipsec-isakmp 
 set peer 195.228.5.15
 set transform-set IKEV1_TRSET 
 match address IPSEC_DC_ACL
!
! interfész cryptomap összerendelés
interface GigabitEthernet0/0
 crypto map CMAP_DC
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```

### Show parancsok

```
rtr-gim-01#show ip route

[...]

Gateway of last resort is not set

     195.228.2.0/24 is variably subnetted, 2 subnets, 2 masks
C       195.228.2.0/27 is directly connected, GigabitEthernet0/0
L       195.228.2.16/32 is directly connected, GigabitEthernet0/0
```

# rtr-gim-02


```
en 
conf t
hostname rtr-gim-02
!
int g0/0
ip address 10.2.255.2 255.255.255.252
no sh
ex
!
int g0/2
ip address 10.2.255.9 255.255.255.252
no sh
exit
!
int g0/1
no sh
exit
!
int gig0/1.200
encapsulation dot1Q 200
description %GIM_RG%
ip address 10.2.0.2 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 200 ip 10.2.0.1 
standby 200 priority 150
standby 200 preempt
exit
!
int gig0/1.201
encapsulation dot1Q 201
description %GIM_VEZ%
ip address 10.2.1.2 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 201 ip 10.2.1.1 
standby 201 priority 150
standby 201 preempt
exit
!
int gig0/1.203
encapsulation dot1Q 203
description %GIM_TANAR%
ip address 10.2.3.2 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 203 ip 10.2.3.1 
standby 203 priority 150
standby 203 preempt
exit
!
int gig0/1.204
encapsulation dot1Q 204
description %GIM_PORTA%
ip address 10.2.4.2 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 204 ip 10.2.4.1 
standby 204 priority 150
standby 204 preempt
exit
!
int gig0/1.205
encapsulation dot1Q 205
description %GIM_GO%
ip address 10.2.5.2 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 205 ip 10.2.5.1 
standby 205 priority 150
standby 205 preempt
exit
!
int gig0/1.210
encapsulation dot1Q 210
description %GIM_WIFI%
ip address 10.2.10.2 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 210 ip 10.2.10.1 
standby 210 priority 150
standby 210 preempt
exit
!
int g0/3/0
no sh
exit
!
int gig0/3/0.220
encapsulation dot1Q 220
description %GIM_SERVER%
ip address 10.2.20.2 255.255.255.0
standby version 2
standby 220 ip 10.2.20.1 
standby 220 priority 150
standby 220 preempt
exit
!
int gig0/3/0.221
encapsulation dot1Q 221
description %GIM_SERVER_MGMT%
ip address 10.2.21.2 255.255.255.0
standby version 2
standby 221 ip 10.2.21.1 
standby 221 priority 150
standby 221 preempt
exit
!
! OSPF konfiguráció
router ospf 2
router-id 2.2.2.2
passive-interface Gig0/1
passive-interface Gig0/3/0
network 10.2.255.0 0.0.0.3 area 2
network 10.2.255.8 0.0.0.3 area 2
network 10.2.0.0 0.0.0.255 area 2
network 10.2.1.0 0.0.0.255 area 2
network 10.2.3.0 0.0.0.255 area 2
network 10.2.4.0 0.0.0.255 area 2
network 10.2.5.0 0.0.0.255 area 2
network 10.2.10.0 0.0.0.255 area 2
network 10.2.20.0 0.0.0.255 area 2
network 10.2.21.0 0.0.0.255 area 2
area 2 authentication message-digest
exit
!
int range gig0/0, gig0/2
ip ospf message-digest-key 2 md5 eiaA8Qsp1i
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```



# rtr-gim-03
```
en 
conf t
hostname rtr-gim-03
!
int g0/0
ip address 10.2.255.6 255.255.255.252
no sh
ex
!
int g0/2
ip address 10.2.255.10 255.255.255.252
no sh
!
int g0/1 
no sh
exit
!
int gig0/1.200
encapsulation dot1Q 200
description %GIM_RG%
ip address 10.2.0.3 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 200 ip 10.2.0.1 
standby 200 priority 120
exit
!
int gig0/1.201
encapsulation dot1Q 201
description %GIM_VEZ%
ip address 10.2.1.3 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 201 ip 10.2.1.1 
standby 201 priority 120
exit
!
int gig0/1.203
encapsulation dot1Q 203
description %GIM_TANAR%
ip address 10.2.3.3 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 203 ip 10.2.3.1 
standby 203 priority 120
exit
!
int gig0/1.204
encapsulation dot1Q 204
description %GIM_PORTA%
ip address 10.2.4.3 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 204 ip 10.2.4.1 
standby 204 priority 120
exit
!
int gig0/1.205
encapsulation dot1Q 205
description %GIM_GO%
ip address 10.2.5.3 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 205 ip 10.2.5.1 
standby 205 priority 120
exit
!
int gig0/1.210
encapsulation dot1Q 210
description %GIM_WIFI%
ip address 10.2.10.3 255.255.255.0
ip helper-address 10.2.20.16
standby version 2
standby 210 ip 10.2.10.1 
standby 210 priority 120
exit
!
int g0/3/0
no sh
exit
!
int Gig0/3/0.220
encapsulation dot1Q 220
description %GIM_SERVER%
ip address 10.2.20.3 255.255.255.0
standby version 2
standby 220 ip 10.2.20.1 
standby 220 priority 120
exit
!
int Gig0/3/0.221
encapsulation dot1Q 221
description %GIM_SERVER_MGMT%
ip address 10.2.21.3 255.255.255.0
standby version 2
standby 221 ip 10.2.21.1 
standby 221 priority 120
exit
! OSPF konfiguráció
router ospf 2
router-id 2.2.2.3
passive-interface Gig0/1
passive-interface Gig0/3/0
network 10.2.255.4 0.0.0.3 area 2
network 10.2.255.8 0.0.0.3 area 2
network 10.2.0.0 0.0.0.255 area 2
network 10.2.1.0 0.0.0.255 area 2
network 10.2.3.0 0.0.0.255 area 2
network 10.2.4.0 0.0.0.255 area 2
network 10.2.5.0 0.0.0.255 area 2
network 10.2.10.0 0.0.0.255 area 2
network 10.2.20.0 0.0.0.255 area 2
network 10.2.21.0 0.0.0.255 area 2
area 2 authentication message-digest
exit
!
int range gig0/0, gig0/2
ip ospf message-digest-key 2 md5 eiaA8Qsp1i
exit
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```

# sw-gim-01
```
en 
conf t
hostname sw-gim-01
!
spanning-tree mode rapid-pvst
! VPT szerver
vtp version 2
vtp domain gim
vtp mode server
vtp password 2VqOjiFH
!
int gig0/1
switchport mode trunk
exit
!
vlan 200
name GIM_RG
!
vlan 201
name GIM_VEZ
!
vlan 202
name GIM_TEL
!
vlan 203
name GIM_TANAR
!
vlan 204
name GIM_PORTA
!
vlan 205
name GIM_GO
!
vlan 210
name GIM_WIFI
!
vlan 240
name GIM_NAT
!
int range fa0/18-19
switchport mode trunk
channel-group 1 mode active
exit
!
int range fa0/20-21
switchport mode trunk
channel-group 2 mode desirable
exit
!
int g0/1 
switchport mode trunk
exit
!
spanning-tree vlan 1-4095 priority 4096
!
int range fa0/2-12
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
switchport voice vlan 202
exit
!
int fa 0/1
switchport mode access
switchport access vlan 210
spanning-tree bpduguard enable
exit
!
int range fa 0/2-11
switchport access vlan 203
exit
!
int fa 0/12
switchport access vlan 201
exit
!
int range fa0/13-17
sh
exit
!
int vlan 200
ip address 10.2.0.192 255.255.255.0
no sh
exit
!
ip default-gateway 10.2.0.1
!
int range fa0/20-21, g0/1, fa0/18-19
switchport trunk native vlan 240
switchport trunk allowed vlan 200,201,202,203,204,205,210
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```



# sw-gim-02
```
en 
conf t
hostname sw-gim-02
!
spanning-tree mode rapid-pvst
!
int range fa0/18-19
switchport mode trunk
channel-group 1 mode passive
exit
!
int range fa0/23-24
switchport mode trunk
channel-group 3 mode active
exit
! VTP cliens
vtp version 2
vtp mode client
vtp domain gim
vtp password 2VqOjiFH
!
spanning-tree vlan 1-4095 priority 8192
!
int range fa 0/3-13, fa0/16                
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
switchport voice vlan 202
exit
!
int range fa 0/2, fa0/14
switchport mode access
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int range fa0/1, fa0/15
switchport mode access
spanning-tree bpduguard enable
switchport access vlan 210
exit
!
int range fa0/17, fa0/20-22, gig0/1-2
switchport mode access
spanning-tree bpduguard enable
sh
exit
!
int range fa0/2-12
switchport access vlan 203
exit
!
int range fa 0/13-14
switchport access vlan 201
exit
!
int fa 0/16
switchport access vlan 204
exit
!
int vlan 200
ip address 10.2.0.193 255.255.255.0
no sh
exit
!
ip default-gateway 10.2.0.1
!
int range fa0/18-19, fa0/23-24
switchport trunk native vlan 240
switchport trunk allowed vlan 200,201,202,203,204,205,210
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```




# sw-gim-03
```
en 
conf t
hostname sw-gim-03
!
spanning-tree mode rapid-pvst
!
int g0/1
switchport mode trunk
!
int range fa0/21-22
switchport mode trunk
channel-group 2 mode auto
exit
!
int range fa0/23-24
switchport mode trunk
channel-group 3 mode passive
exit
! VTP cliens
vtp mode client
vtp version 2
vtp domain gim
vtp password 2VqOjiFH
!
!
int range fa 0/2, fa0/4-9, fa0/11-20
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
switchport voice vlan 202
exit
!
int range  fa 0/3, fa0/10
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int fa 0/1
spanning-tree bpduguard enable
switchport mode access
switchport access vlan 210
exit
!
int fa 0/2
switchport access vlan 200 
exit
!
int fa 0/3
switchport access vlan 202 
exit
!
int range fa 0/4-10
switchport access vlan 205 
exit
!
int range fa 0/11-20
switchport access vlan 203 
exit
!
int gig0/1
switchport mode trunk
exit
!
int fa0/10
spanning-tree bpduguard enable
!
int vlan 200
ip address 10.2.0.194 255.255.255.0
no sh
exit
!
ip default-gateway 10.2.0.1
!
int range gig0/1, fa0/23-24, fa0/21-22
switchport trunk native vlan 240
switchport trunk allowed vlan 200,201,202,203,204,205,210
exit
!
int gig0/2
switchport mode access
spanning-tree bpduguard enable
sh
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# sw-gim-04

```
en
conf t
hostname sw-gim-04
!
vlan 220
name GIM_SERVER
exit
!
vlan 221
name GIM_SERVER_MGMT
!
vlan 241
name GIM_SERVER_NAT
exit
!
int range g0/1, G1/1, G2/1
no sh
switchport mode trunk
switchport trunk native vlan 241
exit
!
int vlan 221
ip address 10.2.21.192 255.255.255.0
no sh
exit
!
ip default-gateway 10.2.21.1
!
int g2/1
switchport mode access
switchport access vlan 220
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```

# ssw-gim-01
```
en 
conf t
hostname ssw-gim-01
!
int fa0/0
description %GIM_TEL%
ip address 10.2.2.1 255.255.255.0
no sh
exit
!
ip dhcp excluded-address 10.2.2.0 10.2.2.63
ip dhcp excluded-address 10.2.2.128 10.2.2.255
ip dhcp pool VOICE
network 10.2.2.0 255.255.255.0
default-router 10.2.2.1
option 150 ip 10.2.2.1
exit
!
telephony-service
max-dn 39
max-ephones 39
ip source-address  10.2.2.1 port 2000
auto assign 1 to 39
exit
!
ephone-dn 1
number 52201
ephone-dn 2
number 52202
ephone-dn 3
number 52203
ephone-dn 4
number 52204
ephone-dn 5
number 52205
ephone-dn 6
number 52206
ephone-dn 7
number 52207
ephone-dn 8
number 52208
ephone-dn 9
number 52209
ephone-dn 10
number 52210
ephone-dn 11
number 52211
ephone-dn 12
number 52212
ephone-dn 13
number 52213
ephone-dn 14
number 52214
ephone-dn 15
number 52215
ephone-dn 16
number 52216
ephone-dn 17
number 52217
ephone-dn 18
number 52218
ephone-dn 19
number 52219
ephone-dn 20
number 52220
ephone-dn 21
number 52221
ephone-dn 22
number 52222
ephone-dn 23
number 52223
ephone-dn 24
number 52224
ephone-dn 25
number 52225
ephone-dn 26
number 52226
ephone-dn 27
number 52227
ephone-dn 28
number 52228
ephone-dn 29
number 52229
ephone-dn 30
number 52230
ephone-dn 31
number 52231
ephone-dn 32
number 52232
ephone-dn 33
number 52233
ephone-dn 34
number 52234
ephone-dn 35
number 52235
ephone-dn 36
number 52236
ephone-dn 37
number 52237
ephone-dn 38
number 52238
ephone-dn 39
number 52239
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```



# srv-gim-01
```
ip cím: 10.2.20.16
mask: 255.255.255.0
default gateway: 10.2.20.1
dns address: 10.5.20.16
```

DHCP

| Pool Name              | Default Gateway | DNS server | Start IP Address | Subnet Mask | Maximum Users | TFTP server |
| :---------------- | :------: | :------: | :------: | :------: | :------: | :------: |
| GIM_RG | 10.2.0.1 | 10.5.20.16 | 10.2.0.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_VEZ | 10.2.1.1 | 10.5.20.16 | 10.2.1.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_TANAR | 10.2.3.1 | 10.5.20.16 | 10.2.3.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_PORTA | 10.2.4.1 | 10.5.20.16 | 10.2.4.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_GO | 10.2.5.1 | 10.5.20.16 | 10.2.5.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_WIFI | 10.2.10.1 | 10.5.20.16 | 10.2.10.64 | 255.255.255.0 | 64 | 10.2.20.16 |

TFTP, DNS




# rtr-telekom-core-04
```
en 
conf t
hostname rtr-telekom-core-04
interface Se0/0/1
ip address 195.228.6.10 255.255.255.252
no shutdown
exit
!
interface Se0/0/0
ip address 195.228.6.14 255.255.255.252
no shutdown
exit
!
interface Gig0/0
no sh
ip address 195.228.5.1 255.255.255.224
ex
!
int Gig 0/1
ip address 195.228.1.1 255.255.255.224
no sh 
exit
! OSPF konfiguráció
router ospf 6
router-id 6.6.6.4
passive-interface Gig0/0
passive-interface Gig0/1
network 195.228.6.8	0.0.0.3 area 6
network 195.228.6.12 0.0.0.3 area 6
network 195.228.5.0	0.0.0.31 area 6
network 195.228.1.0	0.0.0.31 area 6
area 6 authentication message-digest
exit
!
int  se0/0/0
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
!
int  se0/0/1
ip ospf message-digest-key 6 md5 lOLpFsanK7
exit
!
! Titkosítás
service password-encryption
no ip domain lookup
!
enable secret cisco
!
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH
ip domain-name telekom.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```


# fw-dc-01
```
en 
conf t

! default config cleanup
no dhcp auto_config outside 
no dhcp address 192.168.1.5-192.168.1.36 inside
no dhcpd enable inside 
int eth0/0
 no switchport access vlan 2

! base config
hostname fw-dc-01
!
interface Vlan1
 nameif INTERNET
 security-level 0
 ip address 195.228.5.15 255.255.255.224
!
interface Vlan2
 nameif DC_SERVER
 security-level 100
 ip address 10.5.20.1 255.255.255.0
!
interface Vlan530
 no forward interface Vlan2
 nameif DC_SWMAN
 security-level 0
 ip address 10.5.30.1 255.255.255.0
!
interface Ethernet0/0
 ! desc rtr-telekom-04/Gi0/0
 switchport access vlan 1
!
interface Ethernet0/1
 ! desc sw-dc-01/Fa0/1
 switchport access vlan 2
!
interface Ethernet0/2
 ! desc sw-dc-01/Fa0/2
 switchport access vlan 530
!
interface Ethernet0/3
 shutdown
!
interface Ethernet0/4
 shutdown
!
interface Ethernet0/5
 shutdown
!
interface Ethernet0/6
 shutdown
!
interface Ethernet0/7
 shutdown
!
route INTERNET 0.0.0.0 0.0.0.0 195.228.5.1 
!
class-map CMAP_BASE
 match default-inspection-traffic
!
policy-map PMAP_BASE
 class CMAP_BASE
  inspect dns 
  inspect ftp 
  inspect http 
  inspect icmp 
!
service-policy PMAP_BASE global
!
!!!!!!!!!!!!!!! IPsec related objects !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
! közös mindenhol
crypto ipsec ikev1 transform-set IKEV1_TRSET esp-aes-256 esp-sha-hmac
!
! ez is közös
crypto ikev1 policy 100
 encr aes
 authentication pre-share
 group 5
 lifetime 28800
!
object network DC_SERVER
 subnet 10.5.20.0 255.255.255.0
object network GIM_GO
 subnet 10.2.5.0 255.255.255.0
object network GIM_PORTA
 subnet 10.2.4.0 255.255.255.0
object network GIM_RG
 subnet 10.2.0.0 255.255.255.0
object network GIM_SERVER
 subnet 10.2.20.0 255.255.255.0
object network GIM_SERVER_MGMT
 subnet 10.2.21.0 255.255.255.0
object network GIM_TANAR
 subnet 10.2.3.0 255.255.255.0
object network GIM_VEZ
 subnet 10.2.1.0 255.255.255.0
object network GIM_WIFI
 subnet 10.2.10.0 255.255.255.0
object network KOL_SWMAN
 subnet 10.3.30.0 255.255.255.0
object network KOL_DIAK
 subnet 10.3.0.0 255.255.255.0
object network SP_DESKTOP
 subnet 10.7.1.0 255.255.255.0
object network SP_SWMAN
 subnet 10.7.30.0 255.255.255.0
object network TANK_ALK
 subnet 10.1.3.0 255.255.255.0
object network TANK_RG
 subnet 10.1.0.0 255.255.255.0
object network TANK_SERVER
 subnet 10.1.20.0 255.255.255.0
object network TANK_SWMAN
 subnet 10.1.30.0 255.255.255.0
object network TANK_VEZ
 subnet 10.1.1.0 255.255.255.0
object network TANK_WIFI
 subnet 10.1.10.0 255.255.255.0
!
object network rtr-tank-01.internet
 host 195.228.1.16
object network rtr-gim-01.internet
 host 195.228.2.16			
object network rtr-kol-01.internet
 host 195.228.3.16
object network rtr-sp-01.internet
 host 195.228.7.16
object network fw-dc-01.internet
 host 195.228.5.15
!
! Sportpálya specifikus IPsec tunnel ACL
access-list IPSEC_SP_ACL extended permit ip object DC_SERVER object SP_DESKTOP
access-list IPSEC_SP_ACL extended permit ip object DC_SERVER object SP_SWMAN
!
! Gimnázium specifikus IPsec tunnel ACL
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_GO
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_PORTA
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_RG
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_WIFI
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_SERVER
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_SERVER_MGMT
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_TANAR
access-list IPSEC_GIM_ACL extended permit ip object DC_SERVER object GIM_VEZ
!
! Tankerület specifikus IPsec tunnel ACL
access-list IPSEC_TANK_ACL extended permit ip object DC_SERVER object TANK_ALK
access-list IPSEC_TANK_ACL extended permit ip object DC_SERVER object TANK_RG
access-list IPSEC_TANK_ACL extended permit ip object DC_SERVER object TANK_WIFI
access-list IPSEC_TANK_ACL extended permit ip object DC_SERVER object TANK_SERVER
access-list IPSEC_TANK_ACL extended permit ip object DC_SERVER object TANK_SWMAN
access-list IPSEC_TANK_ACL extended permit ip object DC_SERVER object TANK_VEZ
!
! Kollégium specifikus IPsec tunnel ACL
access-list IPSEC_KOL_ACL extended permit ip object DC_SERVER object KOL_SWMAN
access-list IPSEC_KOL_ACL extended permit ip object DC_SERVER object KOL_DIAK
!
!
! Tankerület
crypto map CMAP 1 match address IPSEC_TANK_ACL
crypto map CMAP 1 set peer 195.228.1.16 
crypto map CMAP 1 set ikev1 transform-set IKEV1_TRSET 
! Gimnázium
crypto map CMAP 2 match address IPSEC_GIM_ACL
crypto map CMAP 2 set peer 195.228.2.16 
crypto map CMAP 2 set ikev1 transform-set IKEV1_TRSET 
! Kollégium
crypto map CMAP 3 match address IPSEC_KOL_ACL
crypto map CMAP 3 set peer 195.228.3.16 
crypto map CMAP 3 set ikev1 transform-set IKEV1_TRSET
! Sportpálya
crypto map CMAP 7 match address IPSEC_SP_ACL
crypto map CMAP 7 set peer 195.228.7.16 
crypto map CMAP 7 set ikev1 transform-set IKEV1_TRSET 
!
crypto map CMAP interface INTERNET
crypto ikev1 enable INTERNET
!
! viszonylatonkénti tunnel group
! TANK
tunnel-group 195.228.1.16 type ipsec-l2l
tunnel-group 195.228.1.16 ipsec-attributes
 ikev1 pre-shared-key cisco
!
! GIM
tunnel-group 195.228.2.16 type ipsec-l2l
tunnel-group 195.228.2.16 ipsec-attributes
 ikev1 pre-shared-key cisco
!
! KOL
tunnel-group 195.228.3.16 type ipsec-l2l
tunnel-group 195.228.3.16 ipsec-attributes
 ikev1 pre-shared-key cisco
!
! SP
tunnel-group 195.228.7.16 type ipsec-l2l
tunnel-group 195.228.7.16 ipsec-attributes
 ikev1 pre-shared-key cisco


!!!!!!!!!!!!!!!!!!!!!!! tűzfal szabályok !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
! IPsec tunnelhez szükséges szabályok
access-list FW_INTERNET_ACL extended permit ip object rtr-tank-01.internet object fw-dc-01.internet
access-list FW_INTERNET_ACL extended permit ip object rtr-gim-01.internet object fw-dc-01.internet
access-list FW_INTERNET_ACL extended permit ip object rtr-kol-01.internet object fw-dc-01.internet
access-list FW_INTERNET_ACL extended permit ip object rtr-sp-01.internet object fw-dc-01.internet
!
! Tankerület
access-list FW_INTERNET_ACL extended permit ip object TANK_RG object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object TANK_VEZ object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object TANK_ALK object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object TANK_WIFI object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object TANK_SERVER object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object TANK_SWMAN object DC_SERVER 
!
! Gimnázium
access-list FW_INTERNET_ACL extended permit ip object GIM_RG object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_VEZ object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_TANAR object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_PORTA object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_GO object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_WIFI object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_SERVER object DC_SERVER
access-list FW_INTERNET_ACL extended permit ip object GIM_SERVER_MGMT object DC_SERVER  
!
! Kollégium
access-list FW_INTERNET_ACL extended permit ip object KOL_SWMAN object DC_SERVER 
access-list FW_INTERNET_ACL extended permit ip object KOL_DIAK object DC_SERVER 
!
! Sportpálya
access-list FW_INTERNET_ACL extended permit ip object SP_DESKTOP object DC_SERVER 
access-list FW_INTERNET_ACL extended permit ip object SP_SWMAN object DC_SERVER 
!
no access-list FW_INTERNET_ACL extended deny ip any any
access-list FW_INTERNET_ACL extended deny ip any any
!
!
access-group FW_INTERNET_ACL in interface INTERNET
!
!
! Titkosítás
enable password cisco
!
```

# sw-dc-01
```
en 
conf t
hostname sw-dc-01
!
vlan 520
name DC_SERVER
!
vlan 530
name DC_SWMAN
!
vlan 540
name DC_NATIV
exit
!
int range fa0/2-3
switchport mode access
switchport access vlan 520
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int fa0/1
switchport mode access
switchport access vlan 520
exit
!
int fa0/4
switchport mode access
switchport access vlan 530
exit
!
interface Vlan530
ip address 10.5.30.16 255.255.255.0
exit
ip default-gateway 10.5.30.1
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```



# srv-dc-01 - Windows server 10.5.20.16
Szolgáltatások : 
- Címtár_pl_ Active Directory

- DNS
| Név | Elérhetőség | Típus |
| ------------ | ------------ | ----------- |
| avalon.hu | 10.5.20.16 | A Record |
| www.avalon.hu | 10.5.20.17 | A Record |

- Fájl_és_nyomtató_megosztás

- Kliens_számítógépekre automatizált_szoftvertelepítés

- Automatizált_mentés

- Email szolgáltatás

| Felhasználó gép | Felhasználónév | Jelszó | Email szerver |
| -------------- | -------------- |  -------------- | ------------- |
|  pc-tank-rg-01  | tank-rg-01  | cisco | avalon.hu |
|  pc-tank-kanc-01  | tank-kanc-01  | cisco | avalon.hu |
|  pc-tank-kanc-02  | tank-kanc-02  | cisco | avalon.hu |
|  pc-tank-alk-01  | tank-alk-01  | cisco | avalon.hu |
|  pc-tank-alk-11  | tank-alk-11  | cisco | avalon.hu |
|  pc-tank-alk-12  | tank-alk-12  | cisco | avalon.hu |
|  pc-tank-alk-20  | tank-alk-20  | cisco | avalon.hu |
|  pc-tank-alk-30  | tank-alk-30  | cisco | avalon.hu |
|  sp-tank-01  | tank-sp-01  | cisco | avalon.hu |
|  pc-gim-rg-01  | gim-rg-01  | cisco | avalon.hu |
|  pc-gim-iga-01  | gim-iga-01  | cisco | avalon.hu |
|  pc-gim-iga-02  | gim-iga-02  | cisco | avalon.hu |
|  pc-gim-tan-01  | gim-tan-01  | cisco | avalon.hu |
|  pc-gim-tan-02  | gim-tan-02  | cisco | avalon.hu |
|  pc-gim-tan-11  | gim-tan-21  | cisco | avalon.hu |
|  pc-gim-tan-12  | gim-tan-22  | cisco | avalon.hu |
|  pc-gim-tan-21  | gim-tan-21  | cisco | avalon.hu |
|  pc-gim-tan-22  | gim-tan-22  | cisco | avalon.hu |
|  pc-gim-porta-22  | gim-porta-01  | cisco | avalon.hu |
|  pc-gim-go-01  | gim-go-01  | cisco | avalon.hu |
|  sp-gim-01  | gim-sp-01  | cisco | avalon.hu |
|  sp-kol-01  | kol-sp-01  | cisco | avalon.hu |
|  pc-sp-rg-01  | sp-rg-01  | cisco | avalon.hu |
|  pc-sp-01  | sp-pc-01  | cisco | avalon.hu |




# srv-dc-02 - Linux server 10.5.20.17
Szolgáltatások
- HTTP_HTTPS docker




# rtr-tank-01
```
! első boot után azonnal aktiválni kell a securtity licencet
! el kell fogadni a license agreement-et, ki kell írni a konfigurációt
! majd újra kell indítani a router-t
ena
conf t
license boot module c2900 technology-package securityk9
do wri mem
do reload
!
! indulás után ellenőrizni kell a licence beállításokat
ena
sh ver
!
! konfiguráció építés
conf t
!
hostname rtr-tank-01
!
int g0/1
ip address 195.228.1.16 255.255.255.224
no sh
exit
!
int Gig0/2
ip address 10.1.255.1 255.255.255.252
no sh
exit
!
ip route 0.0.0.0 0.0.0.0 195.228.1.1
! OSPF konfiguráció
router ospf 1
router-id 1.1.1.1
passive-interface Gig0/1
default-information originate
network 195.228.1.0 0.0.0.31 area 1
network 10.1.255.0 0.0.0.3 area 1
area 1 authentication message-digest
exit
!
int g0/2
ip ospf message-digest-key 1 md5 iOIasQ18nh
exit
!
int g0/1 
ip nat outside
exit
!
int g0/2
ip nat inside
exit
!
!
!
! NAT ACL with exceptions
ip access-list extended NAT_ACL
 deny ip 10.1.0.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.1.1.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.1.3.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.1.10.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.1.20.0 0.0.0.255 10.5.20.0 0.0.0.255
 deny ip 10.1.30.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.1.0.0 0.0.0.255 any
 permit ip 10.1.1.0 0.0.0.255 any
 permit ip 10.1.3.0 0.0.0.255 any
 permit ip 10.1.10.0 0.0.0.255 any
 permit ip 10.1.20.0 0.0.0.255 any
!
! dynamic NAT rules
ip nat pool PNATPOOLTANK 195.228.1.16 195.228.1.16 netmask 255.255.255.255
ip nat inside source list NAT_ACL pool PNATPOOLTANK overload
!
!!!!!!!!!!!!!!! IPsec related objects !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
!
crypto ipsec transform-set IKEV1_TRSET esp-aes 256 esp-sha-hmac
!
crypto isakmp policy 100
 encr aes
 authentication pre-share
 group 5
 lifetime 28800
!
! DC oldali ASA
crypto isakmp key cisco address 195.228.5.15
!
! IPsec tunnel specifikus ACL
ip access-list extended IPSEC_DC_ACL
 permit ip 10.1.0.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.1.1.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.1.3.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.1.10.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.1.20.0 0.0.0.255 10.5.20.0 0.0.0.255
 permit ip 10.1.30.0 0.0.0.255 10.5.20.0 0.0.0.255
!
crypto map CMAP_DC 10 ipsec-isakmp 
 set peer 195.228.5.15
 set transform-set IKEV1_TRSET 
 match address IPSEC_DC_ACL
!
! interfész cryptomap összerendelés
interface GigabitEthernet0/1
 crypto map CMAP_DC
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!

```


# rtr-tank-02
```
en 
conf t
hostname rtr-tank-02
!
int Gig0/2
ip address 10.1.255.2 255.255.255.252
no sh
exit
!
int Gig0/3/0
ip address 10.1.255.5 255.255.255.252
no sh
exit
!
int range Gig 0/0, gig0/1
no sh
exit
!
int Gig 0/0.100
description %TANK_RG%
encapsulation dot1q 100
ip address 10.1.0.2 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 100 ip 10.1.0.1 
standby 100 priority 150
standby 100 preempt
exit
!
int Gig 0/0.101
description %TANK_VEZ%
encapsulation dot1q 101
ip address 10.1.1.2 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 101 ip 10.1.1.1 
standby 101 priority 150
standby 101 preempt
exit
!
int Gig 0/1.103
description %TANK_ALK%
encapsulation dot1q 103
ip address 10.1.3.2 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 103 ip 10.1.3.1 
standby 103 priority 150
standby 103 preempt
exit
!
int Gig 0/1.110
description %TANK_WIFI%
encapsulation dot1q 110
ip address 10.1.10.2 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 110 ip 10.1.10.1 
standby 110 priority 150
standby 110 preempt
exit
!
int Gig 0/0.120
description %TANK_RG%
encapsulation dot1q 120
ip address 10.1.20.2 255.255.255.0
standby version 2
standby 120 ip 10.1.20.1 
standby 120 priority 150
standby 120 preempt
exit
!
int Gig 0/1.130
description %TANK_SWMAN%
encapsulation dot1q 130
ip address 10.1.30.2 255.255.255.0
standby version 2
standby 130 ip 10.1.30.1 
standby 130 priority 150
standby 130 preempt
exit
! OSPF konfiguráció
router ospf 1
router-id 1.1.1.2
passive-interface Gig0/1
passive-interface Gig0/0
network 10.1.255.0 0.0.0.3 area 1
network 10.1.255.4 0.0.0.3 area 1
network 10.1.0.0 0.0.0.255 area 1
network 10.1.1.0 0.0.0.255 area 1
network 10.1.3.0 0.0.0.255 area 1
network 10.1.10.0 0.0.0.255 area 1
network 10.1.20.0 0.0.0.255 area 1
network 10.1.30.0 0.0.0.255 area 1
area 1 authentication message-digest
exit
!
int range g0/2, Gig0/3/0
ip ospf message-digest-key 1 md5 iOIasQ18nh
exit
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# rtr-tank-03
```
en 
conf t
hostname rtr-tank-03
!
int Gig0/3/0
ip address 10.1.255.6 255.255.255.252
no sh
exit
!
int range g0/1, g0/0
no sh
exit
!
int Gig 0/0.100
description %TANK_RG%
encapsulation dot1q 100
ip address 10.1.0.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 100 ip 10.1.0.1 
standby 100 priority 120
exit
!
int Gig 0/0.101
description %TANK_VEZ%
encapsulation dot1q 101
ip address 10.1.1.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 101 ip 10.1.1.1 
standby 101 priority 120
exit
!
int Gig 0/0.120
description %TANK_SERVER%
encapsulation dot1q 120
ip address 10.1.20.3 255.255.255.0
standby version 2
standby 120 ip 10.1.20.1 
standby 120 priority 120
exit
!
int Gig 0/1.103
description %TANK_ALK%
encapsulation dot1q 103
ip address 10.1.3.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 103 ip 10.1.3.1 
standby 103 priority 120
exit
!
int Gig 0/1.110
description %TANK_WIFI%
encapsulation dot1q 110
ip address 10.1.10.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 110 ip 10.1.10.1 
standby 110 priority 120
exit
!
int Gig 0/1.130
description %TANK_SWMAN%
encapsulation dot1q 130
ip address 10.1.30.3 255.255.255.0
standby version 2
standby 130 ip 10.1.30.1 
standby 130 priority 120
exit
!
! OSPF konfiguráció
router ospf 1
router-id 1.1.1.3
passive-interface Gig0/1
passive-interface Gig0/0
network 10.1.255.0 0.0.0.3 area 1
network 10.1.255.4 0.0.0.3 area 1
network 10.1.0.0 0.0.0.255 area 1
network 10.1.1.0 0.0.0.255 area 1
network 10.1.3.0 0.0.0.255 area 1
network 10.1.10.0 0.0.0.255 area 1
network 10.1.20.0 0.0.0.255 area 1
network 10.1.30.0 0.0.0.255 area 1
area 1 authentication message-digest
exit
!
int range g0/2, Gig0/3/0
ip ospf message-digest-key 1 md5 iOIasQ18nh
exit
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# ssw-tank-01 
```
en
conf t
hostname ssw-tank-01
int fa0/0
ip address 10.1.4.1 255.255.255.0
no sh
exit
!
ip dhcp excluded-address 10.1.4.0 10.1.4.63
ip dhcp excluded-address 10.1.4.128 10.1.4.255
!
ip dhcp pool ALKTEL
network 10.1.4.0 255.255.255.0
default-router 10.1.4.1
option 150 ip 10.1.4.1
exit
!
telephony-service
max-dn 4
max-ephones 4
ip source-address  10.1.4.1 port 2000
auto assign 1 to 4
exit
!
ephone-dn 1
number 51401
ephone-dn 2
number 51402
ephone-dn 3
number 51403
ephone-dn 4
number 51404
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```

# ssw-tank-02
```
en 
conf t
hostname ssw-gim-02
int fa0/0
ip address 10.1.2.1 255.255.255.0
no sh
exit
!
ip dhcp excluded-address 10.1.2.0 10.1.2.63
ip dhcp excluded-address 10.1.2.128 10.1.2.255
ip dhcp pool VRTEL
network 10.1.2.0 255.255.255.0
default-router 10.1.2.1
option 150 ip 10.1.2.1
exit
!
telephony-service
max-dn 3
max-ephones 3
ip source-address  10.1.2.1 port 2000
auto assign 1 to 3
exit
!
ephone-dn 1
number 51201
ephone-dn 2
number 51202
ephone-dn 3
number 51203
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```



# sw-tank-01
```
en 
conf t
hostname sw-tank-01
! VTP szerver
vtp version 2
vtp domain tanka
vtp mode server
vtp password JXTBOyTW
!
vlan 103
name TANK_ALK
exit
!
vlan 104 
name TANK_ALKTEL
exit
!
vlan 110
name TANK_WIFI
exit
!
vlan 130
name TANK_SWMAN
exit
!
vlan 140
name TANK_NATALK
exit
!
int range fa0/1-3
switchport mode trunk
exit
!
spanning-tree vlan 1-4095 priority 4096
!
int Fa 0/4
switchport access vlan 110
spanning-tree bpduguard enable
exit
!
int  fa 0/6
switchport access vlan 103
switchport voice vlan 104
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
!
int range fa0/5, fa0/7-15
switchport access vlan 103
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
!
int range fa 0/16-24
spanning-tree bpduguard enable
sh
exit
!
int vlan 130
ip address 10.1.30.192 255.255.255.0
no sh
exit
ip default-gateway 10.1.30.1
!
int range fa0/1-3
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,110,130
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```



# sw-tank-02
```
en 
conf t
hostname sw-tank-02
! VTP client
vtp version 2
vtp mode client
vtp domain tanka
vtp password JXTBOyTW
!
int range fa 0/1-3
switchport mode trunk
exit
!
spanning-tree vlan 1-4095 priority  8192
spanning-tree mode rapid-pvst
!
int fa0/4 
switchport mode access
switchport access vlan 103
switchport voice vlan 104
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
!
int fa 0/5
switchport access vlan 110
!
int range fa0/6-24 
spanning-tree bpduguard enable
sh
exit
!
int range fa 0/6-24, gig0/1-2
sh
exit
!
int vlan 130
ip address 10.1.30.193 255.255.255.0
no sh
exit
!
ip default-gateway 10.1.30.1
!
int range fa 0/1-3
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,110,130
exit
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# sw-tank-03
```
en 
conf t
hostname sw-tank-03
! VTP client
vtp version 2
vtp mode client
vtp domain tanka
vtp password JXTBOyTW
!
spanning-tree mode rapid-pvst
!
int range fa0/1-3, Gig0/1
switchport mode trunk
exit
!
int fa 0/4
switchport mode access
switchport access vlan 103
switchport voice vlan 104
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
!
int range fa 0/5-14
switchport mode access
switchport access vlan 103
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int Gig 0/2
sh
!
int vlan 130
ip address 10.1.30.194 255.255.255.0
no sh
exit
!
ip default-gateway 10.1.30.1
!
int range fa0/1-3, Gig0/1
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,110,130
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# sw-tank-04
```
en 
conf t
hostname sw-tank-04
!
spanning-tree mode rapid-pvst
! vtp client
vtp version 2
vtp mode client
vtp domain tanka
vtp password JXTBOyTW
!
int range fa 0/1-3, Gig 0/1
switchport mode trunk
exit
!
int fa 0/4
switchport mode access
switchport access vlan 110
exit
!
int fa 0/5
switchport mode access
switchport access vlan 103
switchport voice vlan 104
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
!
int range fa 0/6-14
switchport mode access
switchport access vlan 103
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int range fa 0/15-24
sh
exit
!
int vlan 130
ip address 10.1.30.195 255.255.255.0
no sh
exit
!
ip default-gateway 10.1.30.1
int range fa 0/1-3,  Gig 0/1
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,110,130
exit
!
int Gig 0/2
switchport mode access
switchport access vlan 104
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# sw-tank-05
```
en 
conf t
hostname sw-tank-05
!
spanning-tree mode rapid-pvst
! VTP server
vtp version 2
vtp domain tankv
vtp mode server
vtp password IvYkhCjy
!
vlan 100
name TANK_RG
exit
!
vlan 101
name TANK_VEZ
exit
!
vlan 102
name TANK_VRTEL
exit
!
vlan 120
name TANK_SERVER
exit
!
vlan 150
name TANK_NATVEZRG
exit
!
int range fa0/1, Gig0/1-2
switchport mode trunk
exit
!
spanning-tree vlan 1-4095 priority 4096 
int fa 0/2 
switchport mode access
switchport access vlan 100
switchport voice vlan 102
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
!
int Gig 0/2
switchport mode access
switchport access vlan 120
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int range fa 0/3-24
switchport mode access
spanning-tree bpduguard enable
sh
!
int vlan 100
ip address 10.1.0.192 255.255.255.0
no sh
exit
!
ip default-gateway 10.1.0.1
!
int range fa0/1, Gig0/1
switchport trunk native vlan 150
switchport trunk allowed vlan 100,101,102,120
exit
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```

# sw-tank-06
```
en 
conf t
hostname sw-tank-06
!
spanning-tree mode rapid-pvst
! VTP client
vtp version 2
vtp mode client
vtp domain tankv
vtp password IvYkhCjy
!
int range fa0/1, Gig0/1
switchport mode trunk
exit
!
spanning-tree vlan 1-4095 priority 8192
int Gig 0/2
switchport mode access
switchport access vlan 102
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
!
int range fa 0/2-3
switchport access vlan 101
switchport voice vlan 102
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
!
int range fa0/4-24
switchport mode access
spanning-tree bpduguard enable
sh
exit
!
int range Gig0/1, fa0/1
switchport trunk native vlan 150
switchport trunk allowed vlan 100,101,102,120
exit
!
int vlan 100
ip address 10.1.0.193 255.255.255.0
no sh
exit
!
ip default-gateway 10.1.0.1
!
!
! Titkosítás
service password-encryption
no ip domain lookup
enable secret cisco
line console 0
password cisco
login
exit
!
banner motd #Az engedely nelkuli hozzaferes tiltott!#
! SSH konfiguráció
ip domain-name avalon.hu
crypto key generate rsa
1024
username admin privilege 15 secret cisco
line vty 0 4
transport input ssh
login local
exit
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
```


# srv-tank-01
```
ip address 10.1.20.16
mask: 255.255.255.0
default gateway: 10.1.20.1
dns address: 10.5.20.16
```

| Pool Name              | Default Gateway | DNS server | Start IP Address | Subnet Mask | Maximum Users | TFTP server |
| :---------------- | :------: | :------: | :------: | :------: | :------: | :------: |
| TANK_RG | 10.1.0.1 | 10.5.20.16 | 10.1.0.64 | 255.255.255.0 | 64 | 10.1.20.16 |
| TANK_VEZ | 10.1.1.1 | 10.5.20.16| 10.1.1.64 | 255.255.255.0 | 64 | 10.1.20.16 |
| TANK_ALK | 10.1.3.1 | 10.5.20.16 | 10.1.3.64 | 255.255.255.0 | 64 | 10.1.20.16 |
| TANK_WIFI | 10.1.10.1 | 10.5.20.16 | 10.1.10.64 | 255.255.255.0 | 64 | 10.1.20.16 |




