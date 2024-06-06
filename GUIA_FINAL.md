# EXAMEN FINAL  FUNDAMENTOS DE REDES  2024-1

<img src="https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240604174024.png" width="540" height="550"/> <img src="https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240604174131.png" width="400"  height="550"/>


<p> Esta es una guía para el parcial de fundamentos de redes.  Esta hecho sobre el parcial montado actualmente (Junio-03-2024) en el campus, puede ser que no todo sea exactamente igual en la versión final del parcial, pero en dicho caso, puede servir de guía. Mucha suerte, compañero!</p>

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

```javascript
R2(config)#ip domain-name cisco.com
R2(config)#username Admin password P4ssw0rd123 //usuario y password para ssh
R2(config)#crypto key generate rsa

//IP correspondiente a la interfaz vlan de gestión en S2 con su respectiva wildcard. Esto es para que ÚNICAMENTE 
//se pueda hacer conexión SSH desde S2

R2(config)#access-list 10 permit 192.168.X.130 0.0.0.31 
R2(config)#access-list 10 deny any
R2(config)#line vty 0 4
R2(config-line)#transport input ssh
R2(config-line)#login local
R2(config-line)#access-class 10 in
R2(config-line)#exit
```

**4. Configuración de opciones de seguridad puertos switch S2**

```javascript
S2(config)#ip default-gateway 192.168.X.129 // IP correspondiente a la vlan de gestión en R2
//Creación de vlans según la tabla de vlans
S2(config)#vlan 10
S2(config-vlan)#name Ventas
S2(config-vlan)#vlan 11
S2(config-vlan)#name Administrativos
S2(config-vlan)#vlan 12
S2(config-vlan)#name Servidores
S2(config-vlan)#vlan 13
S2(config-vlan)#name Gestion
S2(config-vlan)#vlan 14
S2(config-vlan)#name Native
S2(config-if-range)#int r f0/1-24
S2(config-if-range)#switchport mode access //4.1 Los puertos de acceso de S2 deben estar en modo acceso
S2(config-if-range)#int r f0/1-6
S2(config-if-range)#switchport access vlan 10
S2(config-if-range)#int r f0/7-12
S2(config-if-range)#switchport access vlan 11
S2(config-if-range)#int r f0/13-18
S2(config-if-range)#switchport access vlan 12
S2(config-if-range)#int r f0/19-24
S2(config-if-range)#switchport access vlan 13
//Notese que aquí como es la vlan Nativa, hay un par de cambios en los comandos
S2(config-if-range)#int g0/1
S2(config-if)#switchport mode trunk //ya no es access, ahora es trunk
S2(config-if)#switchport trunk native vlan 14 // se le añade "native"
S2(config-if)#switchport trunk allowed vlan 10,11,12,13,14 // todas las vlans permitidas
S2(config-if)#int vlan 13 //interfaz vlan gestión
S2(config-if)#ip address 192.168.X.130 255.255.255.224 //según la tabla de direccionamiento del word del parcial
S2(config-if)#exit

//4.2Los puertos sin ultilizar debe estar deshabilitados
S2(config)#int g0/2
S2(config-if)#sh 

S2(config-if)#int vlan 1
S2(config-if)#sh
S2(config-if)#int r f0/1-24
S2(config-if-range)#switchport mode access 

//4.3 Habilite la seguridad de puertos en los puertos de los switch
S2(config-if-range)#switchport port-security 

//4.4 Agregar todas las direcciones MAC seguras que se detectan dinámicamente en un puerto (hasta el máximo establecido) a la configuración en ejecución del switch
S2(config-if-range)#switchport port-security mac-address sticky 

//4.5 Configure máximo 2 MAC permitidas por puerto
S2(config-if-range)#switchport port-security maximum 2 

//4.6
S2(config-if-range)#switchport port-security violation restrict 
```

**5. Configure el direccionamiento para todos los dispositivos de acuerdo con la tabla de direccionamiento.**

* En R1
```javascript
//Tener en cuenta ip y mascara de subred respectivamente según la tabla de direcciones en el word del laboratorio
//No olvides poner NO SH para cada interfaz. Recuerda que así tengas la configuración correcta si no 
// enciendes cada interfaz con este comando, entonces no va a funcionar
//Las Descr son opcionales. Pero supongo q es mejor ponerlas
Router(config)#hostname R1
R1(config)#int g0/0
R1(config-if)#no sh
R1(config-if)#ip address 192.168.X.1 255.255.255.224
R1(config-if)#DESCR LAN R1
R1(config-if)#int g0/0/0
R1(config-if)#DESCR WAN R2
R1(config-if)#NO SH
R1(config-if)#IP ADDress 192.168.X.225 255.255.255.252
R1(config-if)#int g0/1/0
R1(config-if)#DESCR WAN R3
R1(config-if)#no sh
R1(config-if)#ip address 192.168.X.229 255.255.255.252
```

* En R2
```javascript
//Primero entramos a g0/0 y la encendemos con SH
//Esto es para habilitar la interfaz, y para que cada vez que entremos a una subinterfaz (EJ: g0/0.10)
//esta se encienda automaticamente (es más, creo que si no se enciende g0/0, entonces las subinterfaces 
//no se encenderán, incluso aunque les pongas no sh individualmente!)
R2(config)#int g0/0
R2(config-if)#NO SH

R2(config-if)#int g0/0.10
//IMPORTANTE!!!!! Antes de poner la ip par cada subinterfaz, tienes que poner esta configuración de encapsulación
//donde lo único que cambia es el último valor, el cuál es el número de la subinterfaz 
//(EJ: para g0/0.10 el valor es 10, y para g0/0.11 es 11)
//Se colocan las ips y máscaras de acuerdo a la tabla de direcciones del word del parcial
R2(config-subif)#encapsulation dot1Q 10 
R2(config-subif)#ip address 192.168.X.33 255.255.255.240
R2(config-subif)#int g0/0.11
R2(config-subif)#encapsulation dot1Q 11
R2(config-subif)#ip address 192.168.X.65 255.255.255.240
R2(config-subif)#int g0/0.12
R2(config-subif)#encapsulation dot1Q 12
R2(config-subif)#ip address 192.168.X.97 255.255.255.240
R2(config-subif)#int g0/0.13
R2(config-subif)#encapsulation dot1Q 13
R2(config-subif)#ip address 192.168.X.129 255.255.255.240
R2(config-subif)#int g0/0.14
//Fijate que estamos en la subinterfaz correspondiente a la vlan 14, la cual es la vlan Nativa. Por lo tanto
//tenemos que poner al final de la configuración de encapsulación la palabra "native"
R2(config-subif)#encapsulation dot1Q 14 native //IMPORTANTE: No olvidar colocar native aquí
R2(config-subif)#ip address 192.168.X.161 255.255.255.240
R2(config-subif)#int g0/0/0
R2(config-if)#ip address 192.168.X.226 255.255.255.252
R2(config-if)#no sh
R2(config-if)#descr wan to R1
R2(config-if)#int g0/1/0
R2(config-if)#ip address 192.168.X.234 255.255.255.252
R2(config-if)#descr wan to R3
R2(config-if)#no sh
R2(config-if)#int g0/3/0
R2(config-if)#descr WAN TO ISP
R2(config-if)#ip address 200.31.12.1 255.255.255.252
R2(config-if)#no sh 
```

* En R3
```javascript
// Lo mismo... NO SH para encender, descripción opcional y la ip y máscara de red de acuerdo a la tabla 
//de direccionamiento del word del parcial
Router(config)#hostname R3
R3(config)#int g0/0
R3(config-if)#Descr LAN R3
R3(config-if)#ip address 192.168.X.193 255.255.255.224
R3(config-if)#no sh


R3(config-if)#int g0/0/0
R3(config-if)#descr WAN TO R1
R3(config-if)#ip ad
R3(config-if)#ip address 192.168.X.230 255.255.255.252
R3(config-if)#no sh


R3(config-if)#int g0/1/0
R3(config-if)#descr WAN TO R2
R3(config-if)#ip address 192.168.X.233 255.255.255.252
R3(config-if)#no sh 
```

* Switch 1
```javascript
// Lo mismo... NO SH para encender, descripción opcional y la ip y máscara de red de acuerdo a la tabla de
//direccionamiento del word del parcial
S1>en
S1#conf t
S1(config)#int vlan 1
S1(config-if)#ip address 192.168.X.30 255.255.255.224
S1(config-if)#no sh
S1(config-if)#descr VLAN 1 SWITCH1 INTERFACE
S1(config-if)#EXIT


S1(config)#ip default-gateway 192.168.X.29 //IP de la interfaz de vlan 1 -1.
S1(config)#ip name-server 192.168.X.98 // IP privada de server local. la cual es el dns para todos.
```

* En Switch 2
```javascript
S2>en
S2#conf t
S2(config)#int vlan 13
S2(config-if)#descr VLAN 13 INTERFACE S2
S2(config-if)#ip address 192.168.X.130 255.255.255.224
S2(config-if)#no sh
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.X.129 //IP de la interfaz de vlan 13 -1.
S2(config)#ip name-server 192.168.X.98 // IP privada de server local. la cual es el dns para todos
```

* En Switch 3
```javascript
Switch>en
Switch#conf t
Switch(config)#hostname S3
S3(config)#int vlan 1
S3(config-if)#descr VLAN 1 INTERFACE S3
S3(config-if)#ip address 192.168.X.222 255.255.255.224
S3(config-if)#no sh

S3(config-if)#exit
S3(config)#ip default-gateway 192.168.X.221 //IP de la interfaz de vlan 1 -1.
S3(config)#ip name-server 192.168.X.98 // IP privada de server local. la cual es el dns para todos
```

## Comprobación de conexión de SSH en S2
```javascript
ssh -l Admin 192.168.X.129 <ip de vlan de gestiónen
la subinterfaz de r2 >(en ese caso, si te fijas  en
las tablas, es la vlan 13, entonces es la ip 
correspondiente a la interfaz g0/0.13) 
```

## 5.1

* En Router 2
### DHCP para vlans 
```javascript
R2(config)#ip dhcp pool VLAN10
R2(dhcp-config)#NEtwork 192.168.X.32 255.255.255.240 //la ip de la interfaz de la vlan -1 el último número
R2(dhcp-config)#DEFault-router 192.168.X.33
R2(dhcp-config)#DNs-server 192.168.X.98
R2(dhcp-config)#ip dhcp pool VLAN11
R2(dhcp-config)#NEtwork 192.168.X.64 255.255.255.240//la ip de la interfaz de la vlan -1 el último número
R2(dhcp-config)#DEFault-router 192.168.X.65 
R2(dhcp-config)#DNs-server 192.168.X.98
R2(dhcp-config)#exit
```
* Mismos conceptos para LAN-R1 y Lan-R3...
### DHCP para LAN-R1
```javascript
R2(config)#ip dhcp pool LAN-R1
R2(dhcp-config)#network 192.168.X.0 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.1
R2(dhcp-config)#dns-server 192.168.X.98
```

### DHCP para LAN-R3
```javascript
R2(config)#ip dhcp pool LAN-R3
R2(dhcp-config)#network 192.168.X.192 255.255.255.224
R2(dhcp-config)#default-router 192.168.X.193
R2(dhcp-config)# dns-server 192.168.X.98
```

### Excluyendo IPs
* Se excluyen las 5 primeras IPs de cada LAN en los diferentes routers, también los de las vlans 10 y 11 (g0/0.10 y g0/0.11) y la 192.168.X.222 igualmente, ya que al meter la ip LAN más alta (la del Router 3) y su respectiva máscara en la calculadora https://www.subnet-calculator.com/, obtenemos que la última IP es esa.
```javascript
//las primeras 5 ips en R1 (desde la ip de g0/0)
R2(config)#ip dhcp excluded-address 192.168.X.1 192.168.X.5 

//las primeras 5 ips en vlan 10 de R2 (desde la ip de g0/0.10)
R2(config)#ip dhcp excluded-address 192.168.X.33 192.168.X.37

//las primeras 5 ips en vlan 11 de R2 (desde la ip de g0/0.11)
R2(config)#ip dhcp excluded-address 192.168.X.65 192.168.X.69

//las primeras 5 ips en R3(desde la ip de g0/0)
R2(config)#ip dhcp excluded-address 192.168.X.193 192.168.X.197

//última ip de la red según la calculadora
R2(config)#ip dhcp excluded-address 192.168.X.222 
```

## En Router 1 
```javascript
R1(config)#int g0/0

//Esta ip corresponde a la ip de g0/0/0 en R2 (Donde esta conectado R1 a R2)
R1(config-if)#ip helper-address 192.168.X.226 
```

## En Router 3 
```javascript
R3(config)#int g0/0

//Esta ip corresponde a la ip de g0/1/0 en R2 (Donde esta conectado R3 a R2)
R3(config-if)#ip helper-address 192.168.X.234 
```

# PUNTO 8 OSPF
## ROUTER 2

### ¿CÓMO SACAR UNA WILDCARD?

Sacar la wildcard de la submáscara en una red/ip es muy sencillo, lo único que hay que hacer es tener en cuenta que en cada octeto el número máximo que puede tener es 255, osea que la "máxima" máscara que podríamos tener sería  ```255.255.255.255``` .

Teniendo esto en cuenta, la wildcard se obtiene restando cada octeto de la máscara a la que le queremos sacar la wildcard por ese máximo ```255.255.255.255``` . Por ejemplo, para sacar la wildcard de una de las direcciones que necesitamos ```255.255.255.240``` vamos a restar cada octeto del máximo por este, es decir ```255.255.255.255 - 255.255.255.240```; si restamos 255 - 255 obtenemos 0 así que los primeros 3 octetos serán 0, ahora, el último es el importante, el último es ```240``` así que restaremos ```255``` por ```240```, ```255-240```, lo que nos da ```15```. Así que nuestra wildcard para ```255.255.255.224``` es ```0.0.0.15```

```javascript
R2(config)#router ospf 10
R2(config-router)#router-id 2.2.2.2
// Las IPs correspondientes a las subinterfaces de g0/0, pero -1, con sus respectivas wildcards

//Si te fijas, esta fue la wildcard que sacamos antes "0.0.0.15"
R2(config-router)#network 192.168.X.32 0.0.0.15 area 0 
R2(config-router)#network 192.168.X.64 0.0.0.15 area 0
R2(config-router)#network 192.168.X.96 0.0.0.15 area 0
R2(config-router)#network 192.168.X.128 0.0.0.15 area 0
R2(config-router)#network 192.168.X.160 0.0.0.15 area 0
R2(config-router)#network 192.168.X.225 0.0.0.3 area 0
R2(config-router)#network 192.168.X.233 0.0.0.3 area 0
R2(config-router)#network 200.31.12.0 0.0.0.3 area 0
R2(config-router)#passive-interface g0/0.10
R2(config-router)#passive-interface g0/0.11
R2(config-router)#passive-interface g0/0.12
R2(config-router)#passive-interface g0/0.13
R2(config-router)#passive-interface g0/0.14
```

* Aplicamos el mismo procedimiento tanto para R1 como para R3
## ROUTER 1
```javascript
R1>en
R1#conf t
R1(config)#router ospf 10
R1(config-router)#router-id 1.1.1.1
R1(config-router)#network 192.168.X.0 0.0.0.31 area 0
R1(config-router)#network 192.168.X.224 0.0.0.3 area 0
R1(config-router)#network 192.168.X.228 0.0.0.3 area 0
R1(config-router)#passive-interface g0/0 
```

## ROUTER 3 
```javascript
R3>en
R3#conf t
R3(config)#router ospf 10
R3(config-router)#router-id 3.3.3.3
R3(config-router)#network 192.168.X.192 0.0.0.31 area 0
R3(config-router)#network 192.168.X.229 0.0.0.3 area 0
R3(config-router)#network 192.168.X.232 0.0.0.3 area 0
R3(config-router)#passive-interface g0/0
```

* Una vez configurado, comprobar en cada pc si se puede asignar DHCP

	![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603102451.png)

**Configuración de enrutamiento de Server local**

*  **MUY IMPORTANTE!!!!**: Ten en cuenta el valor de X (en mi caso aquí fue 15). La ip es la de la tabla de direccionamiento en el parcial, la máscara también, la default gateway es la misma IP, pero restandole 1. Y el DNS Server es la misma IP del server (generalmente eso último del DNS es igual para los demas routers)
![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603132938.png)

# Punto 6
<p> Ya se hizo antes cuando se configuró el router 2 (router on a stick, e inter vlan), y lo demás (switches, etc) </p>


# Punto 7
## 7.1 

* Se cambia el index para que muestre ABCVentas

	![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603135722.png)

## 7.2 

* Se agrega www.google.com con la IP de google que se muestra abajito del server encima del dibujo de la nube en la topología.
* Se agrega www.ABCVentas.com con la IP privada del server local **(SE ENCUENTRA EN LA TABLA DE DIRECCIONAMIENTO DEL WORD DEL PARCIAL)**.

	![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603142637.png)

**Importante**: No olvidar darle **"ON"** al servicio de DNS

**9. Configurar una ruta predeterminada**

```javascript

//Esos 0s son básicamente porque queremos decir que lo mande para cualquier dirección IP con cualquier máscara por la interfaz a la que va conectado al ISP.
R2(config)#ip route 0.0.0.0 0.0.0.0 g0/3/0 

R2(config)#router ospf 10

// MUY IMPORTANTE COLOCAR, sino los routers no tendrán acceso a internet exterior.
R2(config-router)#default-information originate 
```

**10. Configure la NAT/PAT** 

* Completar tabla de server local así: (teniendo en cuenta el valor de X)

![ScreenShot](https://raw.githubusercontent.com/Nomka0/PARCIAL_REDES_2024-1/final/Pasted%20image%2020240603140913.png)

 **NAT** 
```javascript

// Las IPs privadas y públicas son las que corresponden al server
// local en la tabla de direccionamiento del word del parcial
R2(config)#ip nat inside source static <ip privada> (192.168.X.98) <ip pública> (200.123.226.1)
```

* Colocamos en cada g0/0.x el comando 'ip nat inside'
```javascript
R2(config)#int g0/0.10

R2(config-subif)#ip nat inside

R2(config-subif)#int g0/0.11

R2(config-subif)#ip nat inside

R2(config-subif)#int g0/0.12

R2(config-subif)#ip nat inside

R2(config-subif)#int g0/0.13

R2(config-subif)#ip nat inside

R2(config-subif)#int g0/0.14

R2(config-subif)#ip nat inside
```

* También en las interfaces g0/0/0 y g0/1/0...
```javascript 
R2(config)#int g0/0/0
R2(config-if)#ip nat inside 
R2(config-if)#int g0/1/0
R2(config-if)#ip nat inside 
```

* Y en g0/3/0, **'ip nat outside'**...
```javascript
R2(config-if)#int g0/3/0

R2(config-if)#ip nat outside
```

**PAT**
* Aquí permitiremos todas las redes lan (r1,r2 y r3. en r2 también las g0/0.x). y el server privado también
```javascript
R2(config)#access-list 1 permit 192.168.X.0 0.0.0.31

R2(config)#access-list 1 permit 192.168.X.32 0.0.0.15

R2(config)#access-list 1 permit 192.168.X.64 0.0.0.15

R2(config)#access-list 1 permit 192.168.X.96 0.0.0.15

R2(config)#access-list 1 permit 192.168.X.128 0.0.0.15

R2(config)#access-list 1 permit 192.168.X.160 0.0.0.15

R2(config)#access-list 1 permit 192.168.X.192 0.0.0.31

R2(config)#access-list 1 permit 192.168.X.98 0.0.0.15
```

* Y después esto:
```javascript
R2(config)#ip nat inside source list 1 interface g0/3/0 overload
```


# Observaciones:
* En los campos a llenar de DNS se coloca la misma ip privada del **Server local**.
* Llenar los campos a completar de las pcs DHCP con los que se generan en ellos después de hacer la configuración OSPF y DHCP.
* La gateway es la ip del Local Server es la ip privada -1 al final.

# Comprobación: 
* Hacer ping entre pcs en la misma lan e interlan.
* Comprobar conexión a ssh en S2.
+ Hacer ping desde el servidor local al servidor externo (Google).
* Hacer ping **desde el command prompt** del server de Google a la ip pública del server local.
* Probar que las páginas web funcionen en los navegodres de los pcs.
