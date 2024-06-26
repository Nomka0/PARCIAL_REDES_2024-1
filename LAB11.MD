# Guía de Laboratorio No 11: Configuración de DHCP en Router y NAT/PAT
<p> Esta es una guía para el ejercicio final/parcial de fundamentos de redes. no sé si todo sea eXactamente igual en el parcial a esta guía basada en el ejercicio final, pero aquí dejo el aporte... Mucha suerte, compañero!</p>

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

![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603014445.png)

**4. 1. Configuración de opciones de seguridad puertos switch S2** 
**Y**
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
S2(config-if)#ip address 192.168.X.130 255.255.255.224
S2(config)#ip default-gateway 192.168.X.129

interface GigabitEthernet0/2
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
S2(config)#int r f0/1-24
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport port-security 
S2(config-if-range)#switchport port-security mac-address sticky
S2(config-if-range)#switchport port-security maXimum 2
S2(config-if-range)#switchport port-security violation restrict

```

**5. Configure el direccionamiento para todos los dispositivos de acuerdo con la tabla de direccionamiento.**

``` javascript
Router(config)#hostname R1
R1(config)#int g0/0
R1(config-if)#ip address 192.168.X.1 255.255.255.224
R1(config-if)#no sh

R1(config-if)#descr LAN R1
R1(config-if)#int g0/0/0
R1(config-if)#ip address 192.168.X.225 255.255.255.252
R1(config-if)#no sh

%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to down
R1(config-if)#descr WAN TO R2
R1(config-if)#INT G0/1/0
R1(config-if)#ip address 192.168.X.229 255.255.255.252
R1(config-if)#DESCR WANT TO R3
R1(config-if)#NO SH
```

//////// LO MISMO PERO EN R3 /////////////

``` javascript
Router>en
Router#conf t
Router(config)#hostname R3
R3(config)#int g0/0
R3(config-if)#ip address 192.168.X.193 255.255.255.224
R3(config-if)#no sh

R3(config-if)#int g0/0/0
R3(config-if)#ip address 192.168.X.230 255.255.255.252
R3(config-if)#no sh

R3(config-if)#int g0/1/0
R3(config-if)#ip address 192.168.X.233 255.255.255.252
R3(config-if)#no sh

R3(config-if)#descr WAN TO R2
R3(config-if)#int g0/0/0
R3(config-if)#descr WAN TO R1
R3(config-if)#int g0/0
R3(config-if)#descr LAN R3
R3(config-if)#EXIT
R3(config)#router ospf 10
R3(config-router)#router-id 3.3.3.3
R3(config-router)#network 192.168.X.192 0.0.0.31 area 0
R3(config-router)#network 192.168.X.229 0.0.0.3 area 0
R3(config-router)#network 192.168.X.232 0.0.0.3 area 0

R3(config-router)#passive-interface g0/0
R3(config-router)#eXit
R3(config)#ip dhcp eXcluded-address 192.168.X.193 192.168.X.198
R3(config)#ip dhcp pool LAN-R3
R3(dhcp-config)#network 192.168.X.192 255.255.255.224
R3(dhcp-config)#default-router 192.168.X.193
R3(dhcp-config)#dns-server 192.168.X.98
R3(dhcp-config)#eXit
R3(config)#ip dhcp eXcluded-address 192.168.X.193 192.168.X.197
R3(config)#ip dhcp eXcluded-address 192.168.X.222
```

//////LO MISMO EN R2///////
```JAVASCRIPT
interface GigabitEthernet0/0
!
interface GigabitEthernet0/0.100
 encapsulation dot1Q 100
 ip address 192.168.X.33 255.255.255.224
!
interface GigabitEthernet0/0.101
 encapsulation dot1Q 101
 ip address 192.168.X.65 255.255.255.224
!
interface GigabitEthernet0/0.102
 encapsulation dot1Q 102
 ip address 192.168.X.97 255.255.255.224
!
interface GigabitEthernet0/0.120
 encapsulation dot1Q 120
 ip address 192.168.X.129 255.255.255.224
!
interface GigabitEthernet0/0.199
 encapsulation dot1Q 199 native
 ip address 192.168.X.161 255.255.255.224
!
interface GigabitEthernet0/0/0
 ip address 192.168.X.226 255.255.255.252
!
interface GigabitEthernet0/1/0
 ip address 192.168.X.234 255.255.255.252
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
 network 192.168.X.128 0.0.0.31 area 0
 network 192.168.X.160 0.0.0.31 area 0
 network 192.168.X.32 0.0.0.31 area 0
 network 192.168.X.64 0.0.0.31 area 0
 network 192.168.X.96 0.0.0.31 area 0
 network 192.168.X.224 0.0.0.3 area 0
 network 192.168.X.232 0.0.0.3 area 0
 network 200.31.12.0 0.0.0.3 area 0


R2(config)#ip dhcp eXcluded-address 192.168.X.97 192.168.X.101
R2(config)#ip dhcp eXcluded-address 192.168.X.129 192.168.X.133
R2(config)#ip dhcp eXcluded-address 192.168.X.161 192.168.X.165
R2(config)#ip dhcp pool VLAN100
R2(dhcp-config)#network 192.168.X.32 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.33
R2(dhcp-config)#ip dhcp pool VLAN101
R2(dhcp-config)#network 192.168.X.64 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.65
R2(dhcp-config)#ip dhcp pool VLAN102
R2(dhcp-config)#network 192.168.X.96 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.97

R2(config)#ip dhcp pool VLAN100
R2(dhcp-config)#dns-server 192.168.X.98
R2(dhcp-config)#ip dhcp pool VLAN101
R2(dhcp-config)#dns-server 192.168.X.98
```



**5.2 Configurar el resto**
* S1
```javascript
S1(config)#int vlan1

S1(config-if)#ip add

S1(config-if)#ip address 192.168.X.30 255.255.255.224
S1(config-if)#no sh
```
* S3
```javascript
S3(config)#int vlan1

S3(config-if)#no sh

S3(config-if)#ip address 192.168.X.222 255.255.255.224
```

**NOTA:** Se calculó la última ip de lan para S1 y S3 usando esta calculadora: https://www.subnet-calculator.com/ , simplemente coloca la ip de la lan de la red con su máscara correspondiente (r1 para s1 y r3 para s3) y te dará el rango de ips disponibles

*  Server local **(Reemplazar X por X)**

	![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603102603.png)


**7. HTTP y DNS en server local.**

![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603041108.png)

![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603041131.png)

**8.Configurar enrutamiento dinámico con OSPF**
```javascript
R1(config-if)#router ospf 10
R1(config-router)#router-id 1.1.1.1
R1(config-router)#net 192.168.X.0
R1(config-router)#net 192.168.X.0 0.0.0.31 area 0
R1(config-router)#net 192.168.X.224 0.0.0.3 area 0
R1(config-router)#net 192.168.X.228 0.0.0.3 area 0
R1(config-router)#passive-interface g0/0

5.1.2 dns server, y DHCP

R1(config)#ip dhcp eXcluded-address 192.168.X.1 192.168.X.5
R1(config)#ip dhcp pool LAN-R1
R1(dhcp-config)#network 192.168.X.0 255.255.255.224
R1(dhcp-config)#default-router 192.168.X.1
R1(dhcp-config)#dns-server 192.168.X.98
```


* Una vez configurado, comprobar en cada pc si se puede asignar DHCP

	![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603102451.png)

**9. Configurar una ruta predeterminada**
```javascript
R2(config)#ip route 0.0.0.0 0.0.0.0 g0/3/0

R2(config)#router ospf 10
R2(config-router)#default-information originate 
```

**10. Configure la NAT/PAT** 

* completar tabla de server local así:

	![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603050058.png)

 **NAT**
 * configuración nat 
```javascript
ip nat inside source static <ip privada> (192.168.X.98) <ip pública> (200.123.226.1)
```

* colocamos en cada g0/0.X el comando 'ip nat inside'
```javascript
interface GigabitEthernet0/0.100
ip nat inside

!

interface GigabitEthernet0/0.101
ip nat inside
!

interface GigabitEthernet0/0.102
ip nat inside
!

interface GigabitEthernet0/0.120
ip nat inside

!

interface GigabitEthernet0/0.199
ip nat inside
```

* También en las interfaces g0/0/0 y g0/1/0...
```javascript 
R2(config)#int g0/0/0
R2(config-if)#ip nat inside 
R2(config-if)#int g0/1/0
R2(config-if)#ip nat inside 
```

* y en g0/3/0, 'ip nat outside'

```javascript
R2(config-if)#int g0/3/0

R2(config-if)#ip nat outside
```

**PAT**
* Aquí permitiremos todas las redes lan (r1,r2 y r3. en r2 también las g0/0.X).
```javascript

R2(config)#access-list 1 permit 192.168.X.0 0.0.0.31

R2(config)#access-list 1 permit 192.168.X.32 0.0.0.31

R2(config)#access-list 1 permit 192.168.X.192 0.0.0.31

R2(config)#access-list 1 permit 192.168.X.229 0.0.0.31

R2(config)#access-list 1 permit 192.168.X.232 0.0.0.31

```

* Y después esto:
```javascript
R2(config)#ip nat inside source list 1 interface g0/3/0 overload
```

**12. Modifique la configuración del direccionamiento dinámico para que ahora el DHCP solamente lo haga el Router R2 para todas las redes, recuerde el comando ip helper-address**

* Vamos a deshacer las configuraciones de dhcp en R1 y R3 y a colocar el **ip helper-address**
```javascript
R1(config)#ip dhcp pool LAN-R1

R1(dhcp-config)#no network 192.168.X.0 255.255.255.224

R1(dhcp-config)#no default-router 192.168.X.1

no dns-server 192.168.X.98

R1(dhcp-config)#exit

R1(config)#no ip dhcp pool LAN-R1

R1(config)#int g0/0
R1(config-if)#ip helper-address 192.168.X.226 //Esta ip corresponde a la ip de g0/0/0 en R2 (Donde esta conectado R1 a R2)
```

```javascript
R3(config)#ip dhcp pool LAN-R1

R3(dhcp-config)#no network 192.168.X.192 255.255.255.224

R3(dhcp-config)#no default-router 192.168.X.193

no dns-server 192.168.X.98

R3(dhcp-config)#exit

R3(config)#no ip dhcp pool LAN-R1
R3(config)#int g0/0
R3(config-if)#ip helper-address 192.168.X.234 //Esta ip corresponde a la ip de g0/1/0 en R2 (Donde esta conectado R3 a R2)
```

* DHCP en R2
```javascript
R2(config)#ip dhcp excluded-address 192.168.X.33 192.168.X.37
R2(config)#ip dhcp excluded-address 192.168.X.65 192.168.X.69
R2(config)#ip dhcp excluded-address 192.168.X.1 192.168.X.5
R2(config)#ip dhcp excluded-address 192.168.X.193 192.168.X.197
R2(config)#ip dhcp excluded-address 192.168.X.222

R2(config)#ip dhcp pool LAN-R1
R2(dhcp-config)#network 192.168.X.0 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.1
R2(dhcp-config)#dns-server 192.168.X.98

R2(config)#ip dhcp pool LAN-R3
R2(dhcp-config)#network 192.168.X.192 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.193
R2(dhcp-config)# dns-server 192.168.X.98

```
# TABLA TALLER ORIGINAL
*  Esta tabla es para comparar si son las mismas ips, si es así, se pueden usar los comandos de arriba sin muchos cambios


![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603013819.png)
