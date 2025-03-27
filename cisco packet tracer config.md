ki kell rakni a telefon központot (nem lehet külön valaminek át kell rajta mennie) jÓ AHOL VAN 
Le kell tesztelni a szervereket
Nem kell szórni ospf-fel a voice hálózatokat DONE
po1 re cseréljem le a etherchannelt DONE
ahol ap vagy telefon van ne legyen bpdguard hanem guard root legyen minden portján ki kell adni DONE
spanning-tree portfast minden porton ahol nincs switch 
spanning-tree portfast default 
Nem kell nativ vlan de biztonság miatt csinálok minden trunk porton engedélyezni kell DONE
Ha mindegyik switch ismeri a tegelést akkor nem kel natív vlan de ha régebbi eszközökezt DONE



EZEN A ROUTEREN IPV6 config kell és SNAT is az IPv6-ra

# rtr-telekom-core-01

### Konfiguracio
```
enable 
conf terminal
hostname rtr-telekom-core-01

interface Se0/0/0
ip address 195.228.6.1 255.255.255.252
no shutdown
exit

interface Se0/0/1
ip address 195.228.6.5 255.255.255.252
no shutdown
exit

ipv6 unicast-routing
interface Gig0/0
ipv6 enable
no shutdown
exit

router ospf 6
router-id 6.6.6.1
passive-interface Gig0/0
network 195.228.6.0	0.0.0.3 area 6
network 195.228.6.4	0.0.0.3 area 6

network ```IPV6``` area 6


ip ospf authentication message-digest
exit
int range se0/0/0-1
ip ospf message-digest-key 1 md5 lOLpFsanK7
exit

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
en 
conf t
hostname rtr-sp-01


# sw-sp-01
en 
conf t
hostname sw-sp-01
sdm prefer dual-ipv4-and-ipv6 default
do reload
y

en 
conf t
vlan 400
name SP_DESKTOP
exit
vlan 430
name SP_SWMAN
exit
vlan 440
name SP_NAT
exit
int vlan 430
ipv6 address  FIXME
int g0/1
switchport mode trunk
switchport trunk native vlan 440
switchport trunk allowed vlan 400,430
exit
int range f0/1-2
switchport mode access 
switchport access vlan 400





# rtr-telekom-core-02

### Konfiguracio

```
enable
configure terminal
hostname rtr-telekom-core-02

interface Se0/0/0
ip address 195.228.6.2 255.255.255.252
no shutdown
exit

interface Se0/0/1
ip address 195.228.6.9 255.255.255.252
no shutdown
exit

interface GigabitEthernet0/0
ip address 195.228.3.1 255.255.255.0
ip address 195.228.3.1 255.255.255.224
no sh
exit

router ospf 6
router-id 6.6.6.2
passive-interface Gig0/0
network 195.228.6.0	0.0.0.3 area 6
network 195.228.6.8	0.0.0.3 area 6
network 195.228.3.0	0.0.0.31  area 6
ip ospf authentication message-digest
exit
int range se0/0/0-1
ip ospf message-digest-key 1 md5 lOLpFsanK7
exit


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

IP default gateway a switcheknek

# rtr-kol-01

### Konfiguracio

```
enable
configure terminal
hostname rtr-kol-01

interface GigabitEthernet0/0
ip address 195.228.3.16 255.255.255.224
no sh
exit

int GigabitEthernet 0/1
no sh
exit

int gigabitEthernet 0/1.300
description %KOL_TANAR%
encapsulation dot1Q 300
ip address 10.3.0.1 255.255.255.0
exit

int gigabitEthernet 0/1.301
description %KOL_DIAK%
encapsulation dot1Q 301
ip address 10.3.1.1 255.255.255.0
exit

int gigabitEthernet 0/1.330
description %KOL_SWMAN%
encapsulation dot1Q 330
ip address 10.3.30.1 255.255.255.0
exit


int gigabitethernet 0/0
ip nat outside
exit


int gigabitethernet 0/1
ip nat inside
exit

access-list 3 permit 10.3.0.0 0.0.0.255
access-list 3 permit 10.3.1.0 0.0.0.255
access-list 3 permit 10.3.30.0 0.0.0.255
ip nat pool PNATPOOLKOL 195.228.3.4 195.228.3.8 netmask 255.255.255.224
ip nat inside source list 3 pool PNATPOOLKOL overload

```
  ^
  I
FIXME nem találtam ki milyen nat kell
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

ENNÉL KI KELL CSERÉLNI A PORTOKAT FIXME

# sw-kol-01

### Konfiguracio

```
enable
config terminal
hostname sw-kol-01
spanning-tree mode rapid-pvst

vlan 300 
name %KOL_TANAR%
exit

vlan 301 
name %KOL_DIAK%
exit

vlan 330 
name %KOL_SWMAN%
exit

vlan 340
name %KOL_NAT%
exit

int vlan 330 
ip address 10.3.30.192 255.255.255.0
no sh 
exit

int range gig0/1, gig6/1, gig5/1, gig7/1, gig8/1, gig9/1
switchport mode trunk
switchport trunk native vlan 340
switchport trunk allowed vlan 300,301,330
exit
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


# wlc-kol-01 FIXME
```
Username:cis$coIT
Password:cis$coIT
System Name: wlc-kol-01-mgmt
Country : Netherlands
Timezone: Amsterdam, Berlin, Rome, Vienna
Management IP address: 10.3.30.193
Subnet mask: 255.255.255.0
Default Gateway: 10.3.30.1
Management VLAN ID: 330
Employee Network:
    WPA2 Personal
    Passphrase:cis$coIT
    Confirm Passphrase: cisco1234
```

# rtr-telekom-core-03

### Konfiguracio

```
enable
config terminal
hostname rtr-telekom-core-03

interface Se0/0/1
ip address 195.228.6.6 255.255.255.252
no shutdown
exit

interface Se0/0/0
ip address 195.228.6.13 255.255.255.252
no shutdown
exit

interface GigabitEthernet0/0
ip address 195.228.2.1 255.255.255.224
no sh
exit

router ospf 6
router-id 6.6.6.3
passive-interface Gig0/0
network 195.228.6.4	0.0.0.3 area 6
network 195.228.6.12 0.0.0.3 area 6
network 195.228.2.0	0.0.0.31 area 6
ip ospf authentication message-digest
exit

int range se0/0/0-1
ip ospf message-digest-key 1 md5 lOLpFsanK7
exit

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
enable
config terminal
hostname rtr-gim-01

int g0/0
ip address 195.228.2.16 255.255.255.224 
no sh
exit

int g0/1
ip address 10.2.255.1 255.255.255.252
no sh
exit

int g0/2
ip address 10.2.255.5 255.255.255.252
no sh
exit

ip route 0.0.0.0 0.0.0.0 195.228.2.1
router ospf 2
router-id 2.2.2.1
passive-interface Gig0/0
default-information originate
network 195.228.2.0 0.0.0.31 area 2
network 10.2.255.0 0.0.0.3 area 2
network 10.2.255.4 0.0.0.3 area 2
ip ospf authentication message-digest
exit

int range g0/1-2
ip ospf message-digest-key 1 md5 eiaA8Qsp1i
exit

int range g0/1-2
ip nat inside
exit

int g0/0
ip nat outside
exit

access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.2.1.0 0.0.0.255
access-list 2 permit 10.2.3.0 0.0.0.255
access-list 2 permit 10.2.4.0 0.0.0.255
access-list 2 permit 10.2.5.0 0.0.0.255
access-list 2 permit 10.2.20.0 0.0.0.255
access-list 2 permit 10.2.21.0 0.0.0.255
ip nat pool PNATPOOLGIM 195.228.3.4 195.228.3.8 netmask 255.255.255.224
ip nat inside source list 2 pool PNATPOOLGIM overload


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

Ki kell venni az 220 ki kell venni áttettem de subinterfacen nem működik ki kell törölni a 220 vlant

# rtr-gim-02


```
en 
conf t
hostname rtr-gim-02
int g0/0
ip address 10.2.255.2 255.255.255.252
no sh
ex
int g0/2
ip address 10.2.255.9 255.255.255.252
no sh
exit

int gig0/1.200
encapsulation dot1Q 200
description %GIM_RG%
ip address 10.2.0.2 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 200 ip 10.2.0.1 
standby 200 priority 150
standby 200 preempt
exit

int gig0/0/1.201
encapsulation dot1Q 201
description %GIM_VEZ%
ip address 10.2.1.2 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 201 ip 10.2.1.1 
standby 201 priority 150
standby 201 preempt
exit

int gig0/0/1.203
encapsulation dot1Q 203
description %GIM_TANAR%
ip address 10.2.3.2 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 203 ip 10.2.3.1 
standby 203 priority 150
standby 203 preempt
exit

int gig0/0/1.204
encapsulation dot1Q 204
description %GIM_PORTA%
ip address 10.2.4.2 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 204 ip 10.2.4.1 
standby 204 priority 150
standby 204 preempt
exit

int gig0/0/1.205
encapsulation dot1Q 205
description %GIM_GO%
ip address 10.2.5.2 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 205 ip 10.2.5.1 
standby 205 priority 150
standby 205 preempt
exit

int gig0/3/0.220
encapsulation dot1Q 220
description %GIM_SERVER%
ip address 10.2.20.2 255.255.255.0
standby version 2
standby 220 ip 10.2.20.1 
standby 220 priority 150
standby 220 preempt
exit

int gig0/3/0.221
encapsulation dot1Q 221
description %GIM_SERVER_MGMT%
ip address 10.2.21.2 255.255.255.0
standby version 2
standby 221 ip 10.2.21.1 
standby 221 priority 150
standby 221 preempt
exit


router ospf 2
router-id 2.2.2.2
passive-interface Gig0/1, Gig0/3/0
network 10.2.255.0 0.0.0.3 area 2
network 10.2.255.8 0.0.0.3 area 2
network 10.2.0.0 0.0.0.255 area 2
network 10.2.1.0 0.0.0.255 area 2
network 10.2.3.0 0.0.0.255 area 2
network 10.2.4.0 0.0.0.255 area 2
network 10.2.5.0 0.0.0.255 area 2
network 10.2.20.0 0.0.0.255 area 2
network 10.2.21.0 0.0.0.255 area 2
ip ospf authentication message-digest
exit
int range gig0/0, gig0/2
ip ospf message-digest-key 1 md5 eiaA8Qsp1i
exit

```



# rtr-gim-03
```
en 
conf t
hostname rtr-gim-03
int g0/0
ip address 10.2.255.6 255.255.255.252
no sh
ex
int g0/2
ip address 10.2.255.10 255.255.255.252
no sh
int g0/1 
no sh
exit
int gig0/0/1.200
encapsulation dot1Q 200
description %GIM_RG%
ip address 10.2.0.3 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 200 ip 10.2.0.1 
standby 200 priority 120
exit

int gig0/0/1.201
encapsulation dot1Q 201
description %GIM_VEZ%
ip address 10.2.1.3 255.255.255.0
ip helper-address FIXME
standby version 2
standby 201 ip 10.2.1.1 
standby 201 priority 120
exit

int gig0/0/1.203
encapsulation dot1Q 203
description %GIM_TANAR%
ip address 10.2.3.3 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 203 ip 10.2.3.1 
standby 203 priority 120
exit

int gig0/0/1.204
encapsulation dot1Q 204
description %GIM_PORTA%
ip address 10.2.4.3 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 204 ip 10.2.4.1 
standby 204 priority 120
exit

int gig0/0/1.205
encapsulation dot1Q 205
description %GIM_GO%
ip address 10.2.5.3 255.255.255.0
ip helper-address 10.2.20.1
standby version 2
standby 205 ip 10.2.5.1 
standby 205 priority 120
exit


int Gig0/3/0.220
encapsulation dot1Q 220
description %GIM_SERVER%
ip address 10.2.20.2 255.255.255.0
standby version 2
standby 220 ip 10.2.20.1 
standby 220 priority 120
exit
int Gig0/3/0.221
encapsulation dot1Q 221
description %GIM_SERVER_MGMT%
ip address 10.2.21.2 255.255.255.0
standby version 2
standby 221 ip 10.2.21.1 
standby 221 priority 120
exit

router ospf 2
router-id 2.2.2.3
passive-interface Gig0/1, Gig0/3/0
network 10.2.255.0 0.0.0.3 area 2
network 10.2.255.8 0.0.0.3 area 2
network 10.2.0.0 0.0.0.255 area 2
network 10.2.1.0 0.0.0.255 area 2
network 10.2.3.0 0.0.0.255 area 2
network 10.2.4.0 0.0.0.255 area 2
network 10.2.5.0 0.0.0.255 area 2
network 10.2.20.0 0.0.0.255 area 2
network 10.2.21.0 0.0.0.255 area 2
ip ospf authentication message-digest
exit
int range gig0/0, gig0/2
ip ospf message-digest-key 1 md5 eiaA8Qsp1i
exit
```

# sw-gim-01
```
en 
conf t
hostname sw-gim-01
spanning-tree mode rapid-pvst
vtp version 2
vtp domain gim
vtp mode server
vtp password 2VqOjiFH
int gig0/1
switchport mode trunk
exit
vlan 200
name GIM_RG
vlan 201
name GIM_VEZ
vlan 202
name GIM_TEL
vlan 203
name GIM_TANAR
vlan 204
name GIM_PORTA
vlan 205
name GIM_GO
vlan 240
name GIM_NAT
int range fa0/18-19
switchport mode trunk
channel-group 1 mode active
exit
int range fa0/20-21
switchport mode trunk
channel-group 2 mode desirable
exit
int g0/1 
switchport mode trunk
exit
spanning-tree vlan 1-4095 priority 4096

int range fa0/2-12
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
switchport voice vlan 202
exit
int fa 0/1
switchport mode access
switchport access vlan 203
spanning-tree bpduguard enable
exit
int range fa 0/2-11
switchport access vlan 203
exit
int fa 0/12
switchport access vlan 201
exit
int range fa0/13-17
sh
exit
int vlan 200
ip address 10.2.0.192 255.255.255.0
no sh
exit

int range fa0/20-21, g0/1, fa0/18-19
switchport trunk native vlan 240
switchport trunk allowed vlan 200,201,202,203,204,205
exit
```



# sw-gim-02
```
en 
conf t
hostname sw-gim-02
spanning-tree mode rapid-pvst
int range fa0/18-19
switchport mode trunk
channel-group 1 mode passive
exit
int range fa0/23-24
switchport mode trunk
channel-group 3 mode active
exit
vtp version 2
vtp mode client
vtp domain gim
vtp password 2VqOjiFH
spanning-tree vlan 1-4095 priority 8192

int range fa 0/3-13, fa0/16                
spanning-tree bpduguard enable
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
switchport voice vlan 202
exit
int range fa 0/2, fa0/15
spanning-tree bpduguard enable
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
int range fa0/1, fa0/16
switchport mode access
spanning-tree bpduguard enable
switchport access vlan 203
exit
int range fa0/17, fa0/20-22, gig0/1-2
switchport mode access
spanning-tree bpduguard enable
sh
exit
int range fa0/2-12
switchport access vlan 203
exit
int range fa 0/13-14
switchport access vlan 201
exit
int fa 0/16
switchport access vlan 204
exit
int range fa0/1, fa0/15
switchport access vlan 203
exit
int vlan 200
ip address 10.2.0.193 255.255.255.0
no sh
exit
int range fa0/18-19, fa0/23-24
switchport trunk native vlan 240
switchport trunk allowed vlan 200,201,202,203,204,205
exit
```




# sw-gim-03
```
en 
conf t
hostname sw-gim-03
spanning-tree mode rapid-pvst
int g0/1
switchport mode trunk
int range fa0/21-22
switchport mode trunk
channel-group 2 mode auto
exit
int range fa0/23-24
switchport mode trunk
channel-group 3 mode passive
exit
vtp mode client
vtp version 2
vtp domain gim
vtp password 2VqOjiFH


int range fa 0/2, fa0/4-9, fa0/11-20
spanning-tree bpduguard enable
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
switchport voice vlan 202
exit
int range  fa 0/3, fa0/10
spanning-tree bpduguard enable
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
int fa 0/1
spanning-tree bpduguard enable
switchport mode access
switchport access vlan 203
exit
int fa 0/2
switchport access vlan 200 
exit
int fa 0/3
switchport access vlan 202 
exit
int range fa 0/5-10
switchport access vlan 205 
exit
int range fa 0/11-20
switchport access vlan 203 
exit
int gig0/1
switchport mode trunk
exit


int vlan 200
ip address 10.2.0.194 255.255.255.0
no sh
exit
int range gig0/1, fa0/23-24, fa0/21-22
switchport trunk native vlan 240
switchport trunk allowed vlan 200,201,202,203,204,205,240
exit
int gig0/2
switchport mode access
spanning-tree bpduguard enable
sh

```


# sw-gim-04

```
en
conf t
hostname sw-gim-04

vlan 220
name GIM_SERVER
exit

vlan 221
name GIM_SERVER_MGMT

vlan 241
name GIM_SERVER_NAT
exit

int range g0/1, G1/1, G2/1
no sh
switchport mode trunk
switchport trunk native vlan 241
switcport trunk allowed vlan none
switcport trunk allowed vlan add 220, 221
exit

int vlan 221
ip address 10.2.21.192 255.255.255.0
no sh
exit

int g2/1
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
```


MEG KELL CSINÁLNI AZ IP TELEFONT

# ssw-gim-01
```
en 
conf t
hostname ssw-gim-01
int fa0/0
ip address 10.2.2.1 255.255.255.0
no sh
exit

ip dhcp excluded-address 10.1.2.0 10.1.2.63
ip dhcp excluded-address 10.1.2.128 10.1.2.255
ip dhcp pool VOICE
network 10.1.2.0 255.255.255.0
default-router 10.1.2.1
option 150 ip 10.1.2.1
exit

telephony-service
max-dn 39
max-ephones 39
ip source-address  10.1.2.1 port 2000
auto assign 1 to 39
exit

ephone-dn 1
number 54001
ephone-dn 2
number 54002
ephone-dn 3
number 54003
ephone-dn 4
number 54004
ephone-dn 5
number 54005
ephone-dn 6
number 54006
ephone-dn 7
number 54007
ephone-dn 8
number 54008
ephone-dn 9
number 54009
ephone-dn 10
number 54010
ephone-dn 11
number 54011

```



# srv-gim-01
```
ip cím: 10.2.20.16
mask: 255.255.255.0
default gateway: 10.2.20.1
```

DHCP

| Pool Name              | Default Gateway | DNS server | Start IP Address | Subnet Mask | Maximum Users | TFTP server |
| :---------------- | :------: | :------: | :------: | :------: | :------: | :------: |
| GIM_RG | 10.2.0.1 | 10.2.20.16 | 10.2.0.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_VEZ | 10.2.1.1 | 10.2.20.16 | 10.2.1.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_TANAR | 10.2.3.1 | 10.2.20.16 | 10.2.3.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_PORTA | 10.2.4.1 | 10.2.20.16 | 10.2.4.64 | 255.255.255.0 | 64 | 10.2.20.16 |
| GIM_GO | 10.2.5.1 | 10.2.20.16 | 10.2.5.64 | 255.255.255.0 | 64 | 10.2.20.16 |

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

interface Se0/0/0
ip address 195.228.6.14 255.255.255.252
no shutdown
exit

interface Gig0/0
no sh
ip address 195.228.5.1 255.255.255.224
ex

int Gig 0/1
ip address 195.228.1.1 255.255.255.224
no sh 
exit

router ospf 6
router-id 6.6.6.4
passive-interface Gig0/0
passive-interface Gig0/1
network 195.228.6.8	0.0.0.3 area 6
network 195.228.6.12 0.0.0.3 area 6
network 195.228.5.0	0.0.0.31 area 6
network 195.228.1.0	0.0.0.31 area 6
ip ospf authentication message-digest
exit

int range se0/0/0-1
ip ospf message-digest-key 1 md5 lOLpFsanK7
exit
```



SZERVER TELEPHELY MÉG SEMMI

# fw-dc-01
en 
conf t
hostname fw-dc-01

# sw-dc-01
en 
conf t
hostname sw-dc-01


# srv-dc-01 - Windows server
Szolgáltatások : 
- Címtár_pl_ Active Directory

- DNS


- Fájl_és_nyomtató_megosztás

- Kliens_számítógépekre automatizált_szoftvertelepítés

- Automatizált_mentés





# srv-dc-02 - Linux server
Szolgáltatások
- HTTP_HTTPS docker




# rtr-tank-01
```
en 
conf t
hostname rtr-tank-01

int g0/1
ip address 192.228.1.16 255.255.255.224
no sh
exit

int Gig0/2
ip address 10.1.255.1 255.255.255.252
no sh
exit

ip route 0.0.0.0 0.0.0.0 195.228.1.1

router ospf 1
router-id 1.1.1.1
passive-interface Gig0/1
default-information originate
network 195.228.1.0 0.0.0.31 area 1
network 10.1.255.0 0.0.0.3 area 1
ip ospf authentication message-digest
exit

int g0/2
ip ospf message-digest-key 1 md5 iOIasQ18nh
exit

int g0/1 
ip nat outside
exit

int g0/2
ip nat inside
exit

access-list 1 permit 10.1.0.0 0.0.0.255
access-list 1 permit 10.1.1.0 0.0.0.255
access-list 1 permit 10.1.3.0 0.0.0.255
access-list 1 permit 10.1.20.0 0.0.0.255
access-list 1 permit 10.1.30.0 0.0.0.255
ip nat pool PNATPOOLTANK 195.228.1.4 195.228.1.8 netmask 255.255.255.224
ip nat inside source list 1 pool PNATPOOLTANK overload

```


# rtr-tank-02
```
en 
conf t
hostname rtr-tank-02

int Gig0/2
ip address 10.1.255.2 255.255.255.252
no sh
exit

int Gig0/3/0
ip address 10.1.255.5 255.255.255.252
no sh
exit

int Gig 0/0
no sh
exit

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


int Gig 0/0.120
description %TANK_RG%
encapsulation dot1q 120
ip address 10.1.20.2 255.255.255.0
standby version 2
standby 120 ip 10.1.20.1 
standby 120 priority 150
standby 120 preempt
exit

int Gig 0/1.130
description %TANK_SWMAN%
encapsulation dot1q 130
ip address 10.1.30.2 255.255.255.0
standby version 2
standby 130 ip 10.1.30.1 
standby 130 priority 150
standby 130 preempt
exit

router ospf 1
router-id 1.1.1.2
passive-interface Gig0/1
passive-interface Gig0/0
network 10.1.255.0 0.0.0.3 area 1
network 10.1.255.4 0.0.0.3 area 1
network 10.1.0.0 0.0.0.255 area 1
network 10.1.1.0 0.0.0.255 area 1
network 10.1.3.0 0.0.0.255 area 1
network 10.1.20.0 0.0.0.255 area 1
network 10.1.30.0 0.0.0.255 area 1
ip ospf authentication message-digest
exit

int range g0/2, Gig0/3/0
ip ospf message-digest-key 1 md5 iOIasQ18nh
exit

```


# rtr-tank-03
```
en 
conf t
hostname rtr-tank-03

int Gig0/3/0
ip address 10.1.255.6 255.255.255.252
no sh
exit

int Gig 0/0.100
description %TANK_RG%
encapsulation dot1q 100
ip address 10.1.0.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 100 ip 10.1.0.1 
standby 100 priority 120
exit

int Gig 0/0.101
description %TANK_VEZ%
encapsulation dot1q 101
ip address 10.1.1.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 101 ip 10.1.1.1 
standby 101 priority 120
exit

int Gig 0/0.120
description %TANK_SERVER%
encapsulation dot1q 120
ip address 10.1.20.2 255.255.255.0
standby version 2
standby 120 ip 10.1.20.1 
standby 120 priority 120
exit

int Gig 0/1.103
description %TANK_ALK%
encapsulation dot1q 103
ip address 10.1.3.3 255.255.255.0
ip helper-address 10.1.20.16
standby version 2
standby 103 ip 10.1.3.1 
standby 103 priority 120
exit

int Gig 0/1.130
description %TANK_SWMAN%
encapsulation dot1q 130
ip address 10.1.30.3 255.255.255.0
standby version 2
standby 130 ip 10.1.30.1 
standby 130 priority 120
exit


router ospf 1
router-id 1.1.1.3
passive-interface Gig0/1
passive-interface Gig0/0
network 10.1.255.0 0.0.0.3 area 1
network 10.1.255.4 0.0.0.3 area 1
network 10.1.0.0 0.0.0.255 area 1
network 10.1.1.0 0.0.0.255 area 1
network 10.1.3.0 0.0.0.255 area 1
network 10.1.20.0 0.0.0.255 area 1
network 10.1.30.0 0.0.0.255 area 1
ip ospf authentication message-digest
exit

int range g0/2, Gig0/3/0
ip ospf message-digest-key 1 md5 iOIasQ18nh
exit
```


# ssw-tank-01 FIXME
en
conf t
hostname ssw-tank-01
int fa0/0
ip address 10.1.4.1 255.255.255.0
no sh
exit
int fa0/1
ip address 10.1.2.1 255.255.255.0
no sh
exit



ip dhcp excluded-address 10.1.4.0 10.1.4.63
ip dhcp excluded-address 10.1.4.128 10.1.4.255
ip dhcp excluded-address 10.1.2.0 10.1.2.63
ip dhcp excluded-address 10.1.2.128 10.1.2.255

ip dhcp pool VRTEL
network 10.1.2.0 255.255.255.0
default-router 10.1.2.1
option 150 ip 10.1.2.1
exit

ip dhcp pool ALKTEL
network 10.1.4.0 255.255.255.0
default-router 10.1.4.1
option 150 ip 10.1.4.1
exit


telephony-service
max-dn 39
max-ephones 39
ip source-address  10.1.2.1 port 2000
auto assign 1 to 39
exit

ephone-dn 1
number 54001
ephone-dn 2
number 54002
ephone-dn 3
number 54003
ephone-dn 4
number 54004
ephone-dn 5
number 54005
ephone-dn 6
number 54006
ephone-dn 7
number 54007
ephone-dn 8
number 54008
ephone-dn 9
number 54009
ephone-dn 10
number 54010
ephone-dn 11
number 54011



# sw-tank-01
```
en 
conf t
hostname sw-tank-01
vtp version 2
vtp domain tanka
vtp mode server
vtp password JXTBOyTW
vlan 103
name TANK_ALK
exit
vlan 104 
name TANK_ALKTEL
exit
vlan 130
name TANK_SWMAN
exit
vlan 140
name TANK_NATALK
exit
int range fa0/1-3
switchport mode trunk
exit
spanning-tree vlan 1-4095 priority 4096

int Fa 0/4
switchport access vlan 103
spanning-tree bpduguard enable
exit
int  fa 0/6
switchport access vlan 103
switchport voice 104
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
int range fa0/5, fa0/7-15
switchport access vlan 103
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
int range fa 0/16-24
spanning-tree bpduguard enable
sh
exit
int vlan 130
ip address 10.1.30.192 255.255.255.0
no sh
exit
int range fa0/2-4
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,130
exit
```



# sw-tank-02
```
en 
conf t
hostname sw-tank-02
vtp version 2
vtp mode client
vtp domain tanka
vtp password JXTBOyTW
int range fa 0/1-3
switchport mode trunk
exit

spanning-tree vlan 1-4095 priority  8192
spanning-tree mode rapid-pvst
int fa0/4 
switchport mode access
switchport access vlan 103
switchport voice 104
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
int fa 0/5
switchport access vlan 103
int range fa0/6-24 
spanning-tree bpduguard enable
sh
exit
int range fa 0/6-24, gig0/1-2
sh
exit
int vlan 130
ip address 10.1.30.193 255.255.255.0
no sh
exit
int range fa 0/1-3
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,130
exit
```


# sw-tank-03
```
en 
conf t
hostname sw-tank-03
vtp version 2
vtp mode client
vtp domain tanka
vtp password JXTBOyTW
spanning-tree mode rapid-pvst
int range fa0/1-3, Gig0/1
switchport mode trunk

exit
int fa 0/4
switchport mode access
switchport access vlan 103
switchport voice 104
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
int fa 0/5

int range fa 0/5-14
switchport mode access
switchport access vlan 103
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
int Gig 0/2
sh
int vlan 130
ip address 10.1.30.194 255.255.255.0
no sh
exit
int range fa0/1-3, Gig0/1
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,130
```


# sw-tank-04
```
en 
conf t
hostname sw-tank-04
spanning-tree mode rapid-pvst
vtp version 2
vtp mode client
vtp domain tanka
vtp password JXTBOyTW
int range fa 0/1-3, int Gig 0/1
switchport mode trunk
exit
int fa 0/4
switchport mode access
switchport access vlan 103
exit
int fa 0/5
switchport mode access
switchport access vlan 103
switchport voice 104
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
int range fa 0/6-14
switchport mode access
switchport access vlan 103
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
int range fa 0/15-24
sh
exit
int vlan 130
ip address 10.1.30.195 255.255.255.0
no sh
exit
int range fa 0/1-3, int Gig 0/1-2
switchport trunk native vlan 140
switchport trunk allowed vlan 103,104,130
exit
```


# sw-tank-05
```
en 
conf t
hostname sw-tank-05
spanning-tree mode rapid-pvst
vtp version 2
vtp domain tankv
vtp mode server
vtp password IvYkhCjy
vlan 100
name TANK_RG
exit
vlan 101
name TANK_VEZ
exit
vlan 102
name TANK_VRTEL
exit
vlan 120
name %TANK_VRTEL%
exit
vlan 150
name TANK_NATVEZRG
exit
int range fa0/1, Gig0/1-2
switchport mode trunk
exit

spanning-tree vlan 1-4095 priority 4096 
int fa 0/2 
switchport mode access
switchport access vlan 100
switchport voice 102
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
int Gig 0/2
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 1
exit
int range fa 0/3-24
switchport mode access
spanning-tree bpduguard enable
sh
int vlan 100
ip address 10.1.0.192 255.255.255.0
no sh
exit
int range fa0/1, Gig0/1-2
switchport trunk native vlan 150
switchport trunk allowed vlan 100,101,102,120
exit
```

# sw-tank-06
```
en 
conf t
hostname sw-tank-06
spanning-tree mode rapid-pvst
vtp version 2
vtp mode client
vtp domain tankv
vtp password IvYkhCjy
int range fa0/1, Gig0/1
switchport mode trunk
exit

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
int range fa 0/2-3
switchport access vlan 101
switchport voice 102
spanning-tree bpduguard enable
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
switchport port-security maximum 2
exit
int range fa0/4-24
switchport mode access
spanning-tree bpduguard enable
sh
exit
int range Gig0/1, Gig0/2
switchport trunk native vlan 150
switchport trunk allowed vlan 100,101,102
exit
int vlan 100
ip address 10.1.0.193 255.255.255.0
no sh
exit
```


# srv-tank-01
```
ip address 10.1.20.16
mask: 255.255.255.0
default gateway: 10.1.20.1
```

| Pool Name              | Default Gateway | DNS server | Start IP Address | Subnet Mask | Maximum Users | TFTP server |
| :---------------- | :------: | :------: | :------: | :------: | :------: | :------: |
| TANK_RG | 10.1.0.1 | 10.1.20.16 | 10.1.0.64 | 255.255.255.0 | 64 | 10.1.20.16 |
| TANK_VEZ | 10.1.1.1 | 10.1.20.16| 10.1.1.64 | 255.255.255.0 | 64 | 10.1.20.16 |
| TANK_ALK | 10.1.3.1 | 10.1.20.16 | 10.1.3.64 | 255.255.255.0 | 64 | 10.1.20.16 |





