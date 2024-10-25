![](https://user-images.githubusercontent.com/73097560/115834477-dbab4500-a447-11eb-908a-139a6edaec5c.gif)

<div style="text-align: center;">
    &nbsp; <span style="font-size: 18px;">Universidad de San Carlos de Guatemala</span><br>
    <span style="font-size: 18px;">Facultad de Ingeniería</span><br>
    <span style="font-size: 18px;">Escuela de Ciencias y Sistemas</span><br>
    <span style="font-size: 18px;">Laboratorio Redes de Computadoras 1 Sección N</span><br>
    <span style="font-size: 18px;">Julio Alfredo Fernández Rodríguez 201902416</span><br>
    <span style="font-size: 18px;">	Sharon Estefany Tagual Godoy201906173</span><br>
    <span style="font-size: 18px;">Albertt Wosvelí Itzep Raymundo 201908658</span> &nbsp;
</div>

<br>

![](https://user-images.githubusercontent.com/73097560/115834477-dbab4500-a447-11eb-908a-139a6edaec5c.gif)


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

