
**2. Configuración de parámetros básico R2:**
``` javascript
Router(config)#hostname R2
R2(config)#no ip domain-lookup
R2(config)#enable secret *código de estudiante*
R2(config)#line console 0
R2(config)#password cisco
R2(config)#banner motd #ADVERTENCIA#
R2(config)#service password-encryption
```

**3. Configuración de acceso por SSH en R2:**
``` javascript
R2(config)#ip domain-name cisco.com
R2(config)#user Admin password Cisco123
R2(config)#crypto key generate rsa
R2(config)#line vty 0 4
R2(config)#transport input ssh
R2(config)#login local
``` 

	3.5. Configure una ACL para acceso SSH a los switch y routers donde solo la VLAN de gestión tenga acceso al VTY
```javascript
R2(config)#access-list 10 permit 192.168.X.130 0.0.0.31
R2(config)#access-list 10 deny any
R2(config)#line vty 0 4
R2(config)#access-class 10 in
```
* Esta es la vlan de gestión en el taller actual, comparar con la del examen, y cambiar si es necesario en el anterior comando (3.5)
![[Pasted image 20240603014445.png]]

**6. Configure enrutamiento entre vlans**

```javascript
Switch(config)#hostname S2
S2(config)#vlan 100
S2(config-vlan)#name Docentes
S2(config-vlan)#vlan 101
S2(config-vlan)#name Estudiantes
S2(config-vlan)#vlan 102
S2(config-vlan)#name Servidores
S2(config-vlan)#vlan 120
S2(config-vlan)#name Gestion
S2(config-vlan)#vlan 199
S2(config-vlan)#name Native
S2(config)#int r f0/1-6
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport access vlan 100

S2(config-if-range)#int r f0/7-12
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport access vlan 101
S2(config-if-range)#switchport mode access
S2(config-if-range)#int r f0/13-18
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport access vlan 102
S2(config-if-range)#int r f0/19-24
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport access vlan 120

S2(config-if-range)#int g0/1
S2(config-if)#switchport mode trunk
S2(config-if)#switchport trunk native vlan 199
S2(config-if)#switchport trunk allowed vlan 100,101,102,120,199
S2(config-if)#int vlan 120
S2(config-if)#ip address 192.168.x.130 255.255.255.224
S2(config)#ip default-gateway 192.168.x.129
```

``` javascript
Router(config)#hostname R1
R1(config)#int g0/0
R1(config-if)#ip address 192.168.x.1 255.255.255.224
R1(config-if)#no sh

R1(config-if)#descr LAN R1
R1(config-if)#int g0/0/0
R1(config-if)#ip address 192.168.x.225 255.255.255.252
R1(config-if)#no sh

%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to down
R1(config-if)#descr WAN TO R2
R1(config-if)#INT G0/1/0
R1(config-if)#ip address 192.168.x.229 255.255.255.252
R1(config-if)#DESCR WANT TO R3
R1(config-if)#NO SH
```

1.1 OSPF en router 1 
```
R1(config-if)#router ospf 10
R1(config-router)#router-id 1.1.1.1
R1(config-router)#net 192.168.x.0
R1(config-router)#net 192.168.x.0 0.0.0.31 area 0
R1(config-router)#net 192.168.x.224 0.0.0.3 area 0
R1(config-router)#net 192.168.x.228 0.0.0.3 area 0
R1(config-router)#passive-interface g0/0

5.1.2 dns server, y DHCP

R1(config)#ip dhcp excluded-address 192.168.x.1 192.168.x.5
R1(config)#ip dhcp pool LAN-R1
R1(dhcp-config)#network 192.168.x.0 255.255.255.224
R1(dhcp-config)#default-router 192.168.x.1
R1(dhcp-config)#dns-server 192.168.x.98
```


//////// LO MISMO PERO EN R3 /////////////

``` javascript
Router>en
Router#conf t
Router(config)#hostname R3
R3(config)#int g0/0
R3(config-if)#ip address 192.168.x.193 255.255.255.224
R3(config-if)#no sh

R3(config-if)#int g0/0/0
R3(config-if)#ip address 192.168.x.230 255.255.255.252
R3(config-if)#no sh

R3(config-if)#int g0/1/0
R3(config-if)#ip address 192.168.x.233 255.255.255.252
R3(config-if)#no sh

R3(config-if)#descr WAN TO R2
R3(config-if)#int g0/0/0
R3(config-if)#descr WAN TO R1
R3(config-if)#int g0/0
R3(config-if)#descr LAN R3
R3(config-if)#EXIT
R3(config)#router ospf 10
R3(config-router)#router-id 3.3.3.3
R3(config-router)#network 192.168.x.192 0.0.0.31 area 0
R3(config-router)#network 192.168.x.229 0.0.0.3 area 0
R3(config-router)#network 192.168.x.232 0.0.0.3 area 0

R3(config-router)#passive-interface g0/0
R3(config-router)#exit
R3(config)#ip dhcp excluded-address 192.168.x.193 192.168.x.198
R3(config)#ip dhcp pool LAN-R3
R3(dhcp-config)#network 192.168.x.192 255.255.255.224
R3(dhcp-config)#default-router 192.168.x.193
R3(dhcp-config)#dns-server 192.168.x.98
R3(dhcp-config)#exit
R3(config)#ip dhcp excluded-address 192.168.x.193 192.168.x.197
R3(config)#ip dhcp excluded-address 192.168.x.222
```

//////LO MISMO EN R2///////
```JAVASCRIPT
interface GigabitEthernet0/0
!
interface GigabitEthernet0/0.100
 encapsulation dot1Q 100
 ip address 192.168.x.33 255.255.255.224
!
interface GigabitEthernet0/0.101
 encapsulation dot1Q 101
 ip address 192.168.x.65 255.255.255.224
!
interface GigabitEthernet0/0.102
 encapsulation dot1Q 102
 ip address 192.168.x.97 255.255.255.224
!
interface GigabitEthernet0/0.120
 encapsulation dot1Q 120
 ip address 192.168.x.129 255.255.255.224
!
interface GigabitEthernet0/0.199
 encapsulation dot1Q 199 native
 ip address 192.168.x.161 255.255.255.224
!
interface GigabitEthernet0/0/0
 ip address 192.168.x.226 255.255.255.252
!
interface GigabitEthernet0/1/0
 ip address 192.168.x.234 255.255.255.252
!
interface GigabitEthernet0/3/0
 ip address 200.31.12.1 255.255.255.252
!
router ospf 10
 router-id 2.2.2.2
 log-adjacency-changes
 passive-interface GigabitEthernet0/0.100
 passive-interface GigabitEthernet0/0.101
 passive-interface GigabitEthernet0/0.199
 passive-interface GigabitEthernet0/0.120
 network 192.168.x.128 0.0.0.31 area 0
 network 192.168.x.160 0.0.0.31 area 0
 network 192.168.x.32 0.0.0.31 area 0
 network 192.168.x.64 0.0.0.31 area 0
 network 192.168.x.96 0.0.0.31 area 0
 network 192.168.x.224 0.0.0.3 area 0
 network 192.168.x.232 0.0.0.3 area 0
 network 200.31.12.0 0.0.0.3 area 0


R2(config)#ip dhcp excluded-address 192.168.x.97 192.168.x.101
R2(config)#ip dhcp excluded-address 192.168.x.129 192.168.x.133
R2(config)#ip dhcp excluded-address 192.168.x.161 192.168.x.165
R2(config)#ip dhcp pool VLAN100
R2(dhcp-config)#network 192.168.x.32 255.255.255.224
R2(dhcp-config)#default-router 192.168.x.33
R2(dhcp-config)#ip dhcp pool VLAN101
R2(dhcp-config)#network 192.168.x.64 255.255.255.224
R2(dhcp-config)#default-router 192.168.x.65
R2(dhcp-config)#ip dhcp pool VLAN102
R2(dhcp-config)#network 192.168.x.96 255.255.255.224
R2(dhcp-config)#default-router 192.168.x.97
R2(dhcp-config)#ip dhcp pool VLAN120
R2(dhcp-config)#network 192.168.x.128 255.255.255.224
R2(dhcp-config)#default-router 192.168.x.129
R2(dhcp-config)#ip dhcp pool VLAN199
R2(dhcp-config)#network 192.168.x.160 255.255.255.224
R2(dhcp-config)#default-router 192.168.x.161
```


# TABLA TALLER ORIGINAL
*  Esta tabla es para comparar si son las mismas ips, si es así, se pueden usar los comandos de arriba sin muchos cambios
![[Pasted image 20240603013819.png]]