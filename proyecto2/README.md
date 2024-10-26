![](https://user-images.githubusercontent.com/73097560/115834477-dbab4500-a447-11eb-908a-139a6edaec5c.gif)

<div style="text-align: center;">
    <span style="font-size: 18px;">Universidad de San Carlos de Guatemala</span><br>
    <span style="font-size: 18px;">Facultad de Ingeniería</span><br>
    <span style="font-size: 18px;">Escuela de Ciencias y Sistemas</span><br>
    <span style="font-size: 18px;">Laboratorio Redes de Computadoras 1 Sección N</span><br>
    <span style="font-size: 18px;">Julio Alfredo Fernández Rodríguez 201902416</span><br>
    <span style="font-size: 18px;">	Sharon Estefany Tagual Godoy201906173</span><br>
    <span style="font-size: 18px;">Albertt Wosvelí Itzep Raymundo 201908658</span> 
</div>

<br>

![](https://user-images.githubusercontent.com/73097560/115834477-dbab4500-a447-11eb-908a-139a6edaec5c.gif)

# <img src="https://user-images.githubusercontent.com/74038190/216649443-702212b5-2704-4b2c-8ab0-38bf536a0d41.gif" width="70px" /> &nbsp; Configuracion parte simulada &nbsp; 



### Configuración de Dispositivos de Red

A continuación se describe la configuración detallada de los dispositivos de red utilizados en esta topología, organizada por cada equipo y con explicaciones específicas para cada sección de configuración.

---

### Configuración de **Switch0**

Este dispositivo es el primero en la topología y tiene la tarea de configurar diferentes VLANs para separar el tráfico de distintas áreas de la organización.

1. **Creación de VLANs**: Las VLANs 18, 28, 38 y 48 representan departamentos específicos como RRHH, Contabilidad, Ventas e Informática.
2. **Asignación de Puertos a VLANs**: Algunos puertos se asignan a VLANs específicas para asegurar que solo el tráfico de esas áreas circule por los puertos asignados.
3. **Configuración de EtherChannel**: Los puertos `fa0/4` y `fa0/5` están configurados para formar un grupo de enlace (EtherChannel) en modo activo.
4. **Configuración VTP**: Este switch está en modo cliente, dentro del dominio `jutiapa`, para recibir las configuraciones de VLAN de un servidor VTP.

```plaintext
enable
conf t
hostname Switch0

vlan 18
name RRHH

vlan 28
name Contabilidad

vlan 38
name Ventas

vlan 48
name Informatica

interface fa0/3
switchport mode access
switchport access vlan 18

interface fa0/1
switchport mode access
switchport access vlan 28

interface fa0/2
switchport mode access
switchport access vlan 38

interface fa0/4
channel-group 1 mode active

interface fa0/5
channel-group 1 mode active

interface fa0/6
switchport mode trunk

vtp mode client
vtp domain jutiapa
```

---

### Configuración de **Switch1**

Este segundo switch sigue una configuración similar a Switch0, con algunas diferencias en la asignación de VLANs a puertos y en la configuración de EtherChannel en modo pasivo.

1. **Asignación de VLANs a Puertos**: Aquí los puertos `fa0/3`, `fa0/1` y `fa0/2` se asignan a VLANs diferentes, asegurando que los distintos departamentos tengan el tráfico separado.
2. **EtherChannel Pasivo**: Los puertos `fa0/4` y `fa0/5` se configuran para agruparse en modo pasivo en lugar de activo.
3. **Configuración VTP**: Configurado como cliente en el dominio `jutiapa`.

```plaintext
enable
conf t
hostname Switch1

vlan 18
name RRHH

vlan 28
name Contabilidad

vlan 38
name Ventas

vlan 48
name Informatica

interface fa0/3
switchport mode access
switchport access vlan 48

interface fa0/1
switchport mode access
switchport access vlan 18

interface fa0/2
switchport mode access
switchport access vlan 38

interface fa0/4
channel-group 1 mode passive

interface fa0/5
channel-group 1 mode passive

interface fa0/6
switchport mode trunk

vtp mode client
vtp domain jutiapa
```

---

### Configuración de **ESW1** (Switch)

ESW1 se configura como el servidor VTP de la red, definiendo VLANs y actuando como punto de acceso a la red IP para cada VLAN mediante interfaces de SVI (Switch Virtual Interface).

1. **Creación de VLANs**: Define las VLANs necesarias para los diferentes departamentos.
2. **Asignación de IP a SVI**: Se asignan direcciones IP a cada VLAN para permitir el enrutamiento inter-VLAN.
3. **Configuración VTP**: Se establece como servidor VTP y se asigna el dominio `jutiapa`.
4. **Spanning Tree RPVST**: Configuración de Rapid PVST para mejorar la eficiencia en la convergencia de la red.

```plaintext
enable
conf t
hostname ESW1

vlan 18
name RRHH

vlan 28
name Contabilidad

vlan 38
name Ventas

vlan 48
name Informatica

interface vlan 18
ip address 192.168.12.49 255.255.255.240

interface vlan 38
ip address 192.168.12.1 255.255.255.224

interface vlan 48
ip address 192.168.12.33 255.255.255.240

interface vlan 28
ip address 192.168.12.65 255.255.255.248

interface fa0/1
switchport mode trunk

interface fa0/2
switchport mode trunk

interface fa0/3
switchport mode trunk

interface fa0/4
switchport mode trunk

vtp mode server
vtp domain jutiapa

spanning-tree mode rapid-pvst
```

---

### Configuración del **Router J2**

El Router J2 gestiona el tráfico de las diferentes VLANs y proporciona redundancia mediante HSRP (Hot Standby Router Protocol). También tiene una conexión con un router externo a través de la interfaz `fa1/0`.

1. **Subinterfaces de VLAN**: Cada VLAN tiene una subinterfaz con su respectiva IP y configurada con HSRP para failover.
2. **Conexión WAN**: La interfaz `fa1/0` conecta a una red externa.

```plaintext
enable
conf t
hostname J2

interface fa0/0.18
encapsulation dot1Q 18
ip address 192.168.12.50 255.255.255.240
standby ip 192.168.12.62

interface fa0/0.28
encapsulation dot1Q 28
ip address 192.168.12.66 255.255.255.248
standby ip 192.168.12.70

interface fa0/0.38
encapsulation dot1Q 38
ip address 192.168.12.2 255.255.255.224
standby ip 192.168.12.30

interface fa0/0.48
encapsulation dot1Q 48
ip address 192.168.12.34 255.255.255.240
standby ip 192.168.12.46

interface fa1/0
ip address 11.0.0.2 255.255.255.252
```

---

### Configuración del **Router J1**

J1 es el router de respaldo y comparte la configuración HSRP con J2. Al igual que J2, J1 tiene subinterfaces para cada VLAN.

```plaintext
enable
conf t
hostname J1

interface fa0/0.18
encapsulation dot1Q 18
ip address 192.168.12.51 255.255.255.240
standby ip 192.168.12.62

interface fa0/0.28
encapsulation dot1Q 28
ip address 192.168.12.67 255.255.255.248
standby ip 192.168.12.70

interface fa0/0.38
encapsulation dot1Q 38
ip address 192.168.12.3 255.255.255.224
standby ip 192.168.12.30

interface fa0/0.48
encapsulation dot1Q 48
ip address 192.168.12.35 255.255.255.240
standby ip 192.168.12.46

interface fa1/0
ip address 11.0.0.6 255.255.255.252
```

---

### Configuración del **Router JUTIAPA**

Este router centraliza la conectividad para múltiples redes a través de interfaces independientes y utiliza RIP para enrutar tráfico entre las redes `10.0.0.0` y `11.0.0.0`.

```plaintext
enable
conf t
hostname JUTIAPA

interface fa1/0
ip address 10.0.1.1 255.255.255.252

interface fa0/0
ip address 10.0.2.1 255.255.255.252

interface fa3/0
ip address 10.0.3.1 255.255.255.252

interface fa4/0
ip address 10.0.4.1 255.255.255.252

interface fa5/0
ip address 10.0.5.1 255.255.255.252

interface fa6/0
ip address 10.0.6.1 255.255.255.252

router rip
version 2
network 10.0.0.0
network 11.0.0.0
redistribute ospf 1 metric 1
redistribute eigrp 100 metric 1
redistribute static
```

---



# <img src="https://user-images.githubusercontent.com/74038190/216655818-2e7b9a31-49bf-4744-85a8-db8a2577c45c.gif" width="70px" /> &nbsp; Configuracion parte fisica &nbsp; 


### Descripción General
Esta configuración permite la comunicación entre dos redes locales, cada una con un switch, conectadas mediante dos routers. Cada router conecta un switch y se configura una ruta estática para permitir la comunicación entre ambas redes. Las IPs están asignadas de manera que identifican de forma única cada segmento de la red.

---

### Configuración General de la Red

1. **Red 1 (izquierda)**
   - Red: `192.178.0.0/24`
   - El switch izquierdo (`SW1`) tiene la IP `192.178.0.2`.
   - La VPC izquierda tiene la IP `192.178.0.1`.

2. **Red 2 (derecha)**
   - Red: `192.168.0.0/24`
   - El switch derecho (`SW2`) tiene la IP `192.168.0.2`.
   - La VPC derecha tiene la IP `192.168.0.1`.

3. **Red de conexión entre Routers**
   - Red de enlace: `10.0.0.0/24`
   - El router 1 (`R1`) tiene la IP `10.0.0.1`.
   - El router 2 (`R2`) tiene la IP `10.0.0.2`.

---

### Configuración de Routers

#### Configuración de `R1`

```plaintext
enable               // Activa el modo privilegiado.
conf t               // Entra en modo de configuración global.
hostname R1          // Establece el nombre del router como "R1".

// Interfaz de conexión al switch izquierdo (gig0/0/1)
int gig0/0/1         
ip add 192.178.0.2 255.255.255.0   // Asigna la IP de la interfaz.
no shut              // Activa la interfaz.

// Interfaz de conexión a `R2` (gig0/0/0)
int gig0/0/0
ip add 10.0.0.1 255.255.255.0      // Asigna la IP para la conexión entre routers.
no shut              // Activa la interfaz.

// Configura una ruta estática hacia la red derecha (192.168.0.0/24) usando `R2` como gateway.
ip route 192.168.0.0 255.255.255.0 10.0.0.2
do wr                // Guarda la configuración.
```

#### Configuración de `R2`

```plaintext
enable               // Activa el modo privilegiado.
conf t               // Entra en modo de configuración global.
hostname R2          // Establece el nombre del router como "R2".

// Interfaz de conexión al switch derecho (gig0/0/1)
int gig0/0/1         
ip add 192.168.0.2 255.255.255.0   // Asigna la IP de la interfaz.
no shut              // Activa la interfaz.

// Interfaz de conexión a `R1` (gig0/0/0)
int gig0/0/0
ip add 10.0.0.2 255.255.255.0      // Asigna la IP para la conexión entre routers.
no shut              // Activa la interfaz.

// Configura una ruta estática hacia la red izquierda (192.178.0.0/24) usando `R1` como gateway.
ip route 192.178.0.0 255.255.255.0 10.0.0.1
do wr                // Guarda la configuración.
```

---

### Configuración de Switches

#### Configuración de `SW1` (izquierdo)

```plaintext
enable               // Activa el modo privilegiado.
conf t               // Entra en modo de configuración global.
hostname SW1         // Establece el nombre del switch como "SW1".

vlan 10              // Crea VLAN 10.
name Left_Network    // Asigna el nombre "Left_Network" a la VLAN.
exit                 // Sale de la configuración de la VLAN.

// Configura el puerto de conexión a la VPC en modo acceso y lo asigna a la VLAN 10.
int fa0/1
switchport mode access
switchport access vlan 10
exit

do wr                // Guarda la configuración.
```

#### Configuración de `SW2` (derecho)

```plaintext
enable               // Activa el modo privilegiado.
conf t               // Entra en modo de configuración global.
hostname SW2         // Establece el nombre del switch como "SW2".

vlan 20              // Crea VLAN 20.
name Right_Network   // Asigna el nombre "Right_Network" a la VLAN.
exit                 // Sale de la configuración de la VLAN.

// Configura el puerto de conexión a la VPC en modo acceso y lo asigna a la VLAN 20.
int fa0/1
switchport mode access
switchport access vlan 20
exit

do wr                // Guarda la configuración.
```

---

### Resumen de la Configuración

1. **R1 y R2** están conectados entre sí mediante la red `10.0.0.0/24`, permitiendo la comunicación entre las redes `192.178.0.0/24` y `192.168.0.0/24`.
2. **SW1** y **SW2** tienen VLANs configuradas para las VPC de sus respectivas redes.
3. Se establecen rutas estáticas en los routers para direccionar el tráfico entre ambas redes.

