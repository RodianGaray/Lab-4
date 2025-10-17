# Laboratorio 4 - Configuración de Red Cisco 2960 y Virtualización

## Descripción del Proyecto
Implementación completa de laboratorio de redes que incluye configuración de switch Cisco 2960, interconexión de múltiples dispositivos, análisis de red con Nmap, y virtualización de sistemas operativos usando QEMU.

## 1. Configuración Cisco 2960 y Conexión de Red

### Manual de Referencia Cisco 2960

#### Comandos Básicos de Configuración

```
# Modos de Operación del Switch
Switch> enable                    # Entrar al modo privilegiado
Switch# configure terminal        # Entrar al modo configuración global
Switch(config)#                   # Indicador de modo configuración global

# Configuración Básica del Switch
Switch(config)# hostname SW-LAB4
SW-LAB4(config)# enable secret laboratorio4
SW-LAB4(config)# line console 0
SW-LAB4(config-line)# password cisco
SW-LAB4(config-line)# login
SW-LAB4(config-line)# exit

# Configuración de Interfaces
SW-LAB4(config)# interface fastethernet 0/1
SW-LAB4(config-if)# description PC-Monitor
SW-LAB4(config-if)# switchport mode access
SW-LAB4(config-if)# switchport access vlan 10
SW-LAB4(config-if)# no shutdown
SW-LAB4(config-if)# exit

# Configuración de VLANs
SW-LAB4(config)# vlan 10
SW-LAB4(config-vlan)# name PC-Monitor
SW-LAB4(config-vlan)# exit
SW-LAB4(config)# vlan 20
SW-LAB4(config-vlan)# name Dispositivos-Lab
SW-LAB4(config-vlan)# exit

# Verificación de Configuración
SW-LAB4# show vlan brief
SW-LAB4# show running-config
SW-LAB4# show interfaces status
```
#### Comandos de Monitoreo y Diagnóstico
```
# Verificación de Estado
show interfaces fastethernet 0/1    # Estado de interfaz específica
show mac address-table              # Tabla de direcciones MAC
show vlan                           # Información de VLANs
show spanning-tree                  # Estado spanning-tree
show ip interface brief             # Resumen interfaces IP

# Comandos de Diagnóstico
ping 192.168.1.10                   # Prueba de conectividad
traceroute 192.168.1.20             # Rastreo de ruta
show logging                        # Registros del sistema

# Comandos de Respaldo
copy running-config startup-config  # Guardar configuración
show startup-config                 # Ver configuración guardada
```
#### Conexión Paso a Paso entre PC y Switch
1. Conexión Física
```
# Requerimientos de Hardware:
# - Switch Cisco 2960
# - Cable Ethernet straight-through
# - PC con puerto Ethernet
# - Cable de consola (para configuración inicial)

# Pasos de Conexión:
1. Conectar cable de consola del PC al puerto console del switch
2. Conectar cable Ethernet del PC al puerto FastEthernet 0/1
3. Verificar LEDs de estado en el switch (deben estar verdes)
```
### 2. Configuración de Conexión Serial
```
# En sistemas Linux/Ubuntu:
sudo apt install screen
sudo screen /dev/ttyUSB0 9600

# En Windows:
# Usar Putty o HyperTerminal con configuración:
# - Puerto: COM1 (verificar en administrador de dispositivos)
# - Velocidad: 9600 bauds
# - Bits de datos: 8
# - Paridad: None
# - Bits de parada: 1
# - Control de flujo: None
```
### 3. Configuración de Red en PC Monitor
```
# Verificar interfaces de red
ip addr show
# o
ifconfig

# Configurar IP estática para eth0
sudo ip addr add 192.168.1.10/24 dev eth0
sudo ip link set eth0 up

# Verificar configuración
ip addr show eth0

# Configurar gateway si es necesario
sudo ip route add default via 192.168.1.1
```
#### Topología de Red Implementada
text
                    +-------------------+
                    |  Cisco Switch 2960 |
                    |     VLAN 10       |
                    +---------+---------+
                              |
          +-------------------+-------------------+
          |                   |                   |
+---------+---------+ +-------+-------+ +---------+---------+
|    PC Monitor     | |   PC ETM      | |   Raspberry Pi    |
|   192.168.1.10/24| |192.168.1.20/24| | 192.168.1.30/24  |
+-------------------+ +---------------+ +-------------------+
## 2. Pruebas de Conectividad y Análisis de Red
### Script de Pruebas de Red Automatizado
```
#!/bin/bash
# scripts/pruebas_red.sh

echo "=== PRUEBAS DE CONECTIVIDAD - LABORATORIO 4 ==="
echo "Fecha: $(date)"
echo "=============================================="

# Dispositivos en la red
DEVICES=("192.168.1.10" "192.168.1.20" "192.168.1.30" "192.168.1.1")

# 1. Pruebas de Ping entre todos los dispositivos
echo "1. PRUEBAS DE PING ENTRE DISPOSITIVOS"
for device in "${DEVICES[@]}"; do
    if ping -c 3 -W 2 $device &> /dev/null; then
        echo "✅ $device - CONECTADO"
    else
        echo "❌ $device - SIN CONEXIÓN"
    fi
done
echo

# 2. Análisis de Tabla ARP
echo "2. TABLA ARP - DISPOSITIVOS LOCALES"
arp -a
echo

# 3. Verificación de Rutas
echo "3. TABLA DE RUTEO"
ip route show
echo
```
### Análisis de Red con Nmap
```
#!/bin/bash
# scripts/analisis_nmap.sh

echo "=== ANÁLISIS DE RED CON NMAP ==="
NETWORK="192.168.1.0/24"

# 1. Descubrimiento de hosts activos
echo "1. HOSTS ACTIVOS EN LA RED:"
sudo nmap -sn $NETWORK

# 2. Escaneo de puertos en dispositivos específicos
echo "2. ESCANEO DE PUERTOS - PC MONITOR:"
sudo nmap -sS -p 1-1000 192.168.1.10

echo "3. ESCANEO DE PUERTOS - RASPBERRY PI:"
sudo nmap -sS -p 1-1000 192.168.1.30

# 3. Detección de servicios y versiones
echo "4. DETECCIÓN DE SERVICIOS:"
sudo nmap -sV 192.168.1.10,20,30

# 4. Escaneo de vulnerabilidades básico
echo "5. ESCANEO DE SEGURIDAD:"
sudo nmap --script safe 192.168.1.1
```
### Explicación de Conceptos de Red
#### Comandos de Verificación
```
# ifconfig vs ip addr
echo "=== INFORMACIÓN DE INTERFACES ==="
ifconfig
# o el comando moderno:
ip addr show

# Explicación de la salida:
# - eth0: Interfaz Ethernet
# - inet: Dirección IP (192.168.1.10)
# - netmask: Máscara de subred (255.255.255.0)
# - broadcast: Dirección de broadcast
```
### Conceptos Explicados
#### Puerta de Enlace (Gateway)
```
ip route show default
# Salida: default via 192.168.1.1 dev eth0
```
Definición: Dispositivo que conecta la red local con otras redes
Función: Enruta tráfico hacia fuera de la red local
Ejemplo: Router que conecta a Internet
#### Máscara de Subred
```
ip addr show eth0
# inet 192.168.1.10/24
```
Definición: Define qué parte de la IP identifica la red y qué parte identifica el host
/24: Equivale a 255.255.255.0 (primeros 24 bits para red)
Función: Determina si un host está en la misma red local

#### VLAN (Virtual LAN)
```
# En el switch Cisco:
show vlan brief
```
Definición: Red lógica dentro de una red física
Ventajas: Segmentación, seguridad, gestión de tráfico
Ejemplo: VLAN 10 para PCs de administración
#### Bloque CIDR  
```
# Notación: 192.168.1.0/24
```
Definición: Notación para especificar rangos de IP
/24: 256 direcciones (192.168.1.0 - 192.168.1.255)
/26: 64 direcciones (192.168.1.0 - 192.168.1.63)

#### Transferencia de Archivos con SCP
```
#!/bin/bash
# scripts/transferencia_archivos.sh

echo "=== TRANSFERENCIA DE ARCHIVOS CON SCP ==="

# 1. Verificar que el servicio SSH esté activo en el destino
echo "1. VERIFICANDO SERVICIO SSH EN DESTINO:"
sudo nmap -p 22 192.168.1.20

# 2. Transferir archivo del PC Monitor al PC ETM
echo "2. TRANSFIRIENDO ARCHIVO:"
ARCHIVO_PRUEBA="prueba_laboratorio4.txt"

# Crear archivo de prueba
echo "Este es un archivo de prueba para Laboratorio 4" > $ARCHIVO_PRUEBA

# Transferir archivo (asumiendo usuario 'usuario' en PC destino)
scp $ARCHIVO_PRUEBA usuario@192.168.1.20:/home/usuario/

# Verificar transferencia
if [ $? -eq 0 ]; then
    echo "✅ Archivo transferido exitosamente"
else
    echo "❌ Error en la transferencia"
    echo "Solución: Verificar que SSH esté activo en el destino"
    echo "Comando para activar SSH en Ubuntu: sudo systemctl enable ssh && sudo systemctl start ssh"
fi

# 3. Transferencia desde Raspberry Pi
echo "3. TRANSFERENCIA DESDE RASPBERRY PI:"
scp pi@192.168.1.30:/home/pi/documento.txt ./documentos_descargados/

# 4. Transferencia recursiva de directorios
echo "4. TRANSFERENCIA DE DIRECTORIO COMPLETO:"
scp -r ./evidencias/ usuario@192.168.1.20:/home/usuario/backup_lab4/
```
## 3. Virtualización con QEMU
### Script de Instalación Automatizada
```
#!/bin/bash
# config/qemu_scripts/instalacion_vms.sh

echo "=== INSTALACIÓN DE MÁQUINAS VIRTUALES CON QEMU ==="

# Crear directorio para VMs
mkdir -p ~/qemu_vms
cd ~/qemu_vms

# Función para crear disco virtual
crear_disco() {
    local nombre=$1
    local tamaño=$2
    qemu-img create -f qcow2 ${nombre}.qcow2 ${tamaño}
    echo "Disco virtual ${nombre}.qcow2 creado (${tamaño})"
}
```
### Instalación de Ubuntu
```
#!/bin/bash
# config/qemu_scripts/instalar_ubuntu.sh

echo "=== INSTALACIÓN UBUNTU SERVER ==="

# Crear disco de 20GB para Ubuntu
crear_disco "ubuntu_server" "20G"

# Descargar ISO de Ubuntu Server (si no existe)
if [ ! -f "ubuntu-22.04.3-live-server-amd64.iso" ]; then
    wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso
fi
# Instalar Ubuntu
qemu-system-x86_64 \
    -hda ubuntu_server.qcow2 \
    -cdrom ubuntu-22.04.3-live-server-amd64.iso \
    -boot d \
    -m 2G \
    -smp 2 \
    -vga virtio \
    -display gtk \
    -netdev user,id=net0 \
    -device virtio-net-pci,netdev=net0

echo "Ubuntu Server instalado exitosamente"
echo "Comando para iniciar: qemu-system-x86_64 -hda ubuntu_server.qcow2 -m 2G -smp 2 -vga virtio"
```
### Instalación de CentOS
```
#!/bin/bash
# config/qemu_scripts/instalar_centos.sh

echo "=== INSTALACIÓN CENTOS STREAM ==="

# Crear disco de 15GB para CentOS
crear_disco "centos_stream" "15G"
# Instalar CentOS Stream
qemu-system-x86_64 \
    -hda centos_stream.qcow2 \
    -cdrom /ruta/a/CentOS-Stream-9-latest-x86_64-dvd1.iso \
    -boot d \
    -m 2G \
    -smp 2 \
    -vga std \
    -display gtk \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0

echo "CentOS Stream instalado exitosamente"
echo "Acceso SSH: ssh usuario@localhost -p 2222"
```
### Instalación de Alpine Linux
```
#!/bin/bash
# config/qemu_scripts/instalar_alpine.sh

echo "=== INSTALACIÓN ALPINE LINUX ==="

# Crear disco de 5GB para Alpine (sistema minimalista)
crear_disco "alpine_linux" "5G"

# Descargar Alpine Standard
if [ ! -f "alpine-standard-3.18.4-x86_64.iso" ]; then
    wget https://dl-cdn.alpinelinux.org/alpine/v3.18/releases/x86_64/alpine-standard-3.18.4-x86_64.iso
fi

# Instalar Alpine
qemu-system-x86_64 \
    -hda alpine_linux.qcow2 \
    -cdrom alpine-standard-3.18.4-x86_64.iso \
    -boot d \
    -m 512M \
    -smp 1 \
    -vga std \
    -display gtk \
    -netdev user,id=net0 \
    -device virtio-net-pci,netdev=net0

echo "Alpine Linux instalado exitosamente"
echo "Sistema minimalista listo para configuración"
```
### Instalación de Scientific Linux
```
#!/bin/bash
# config/qemu_scripts/instalar_scientific.sh

echo "=== INSTALACIÓN SCIENTIFIC LINUX ==="

# Crear disco de 15GB para Scientific Linux
crear_disco "scientific_linux" "15G"

# Instalar Scientific Linux
qemu-system-x86_64 \
    -hda scientific_linux.qcow2 \
    -cdrom /ruta/a/SL-7-x86_64-DVD.iso \
    -boot d \
    -m 2G \
    -smp 2 \
    -vga std \
    -display gtk \
    -netdev user,id=net0 \
    -device e1000,netdev=net0

echo "Scientific Linux instalado exitosamente"
echo "Distribución científica basada en RHEL lista para usar"
```
## Imagenes de prueba de proceso 
![Figura 1. Comunicación establecida mediante la terminal de Ubuntu utilizando la herramienta screen.](ruta/de/la/imagen.png)
**Figura 1.** Comunicación establecida mediante la terminal de Ubuntu utilizando la herramienta *screen*.

![Figura 2. Configuración inicial básica del switch Cisco 2960.](ruta/de/la/imagen.png)
**Figura 2.** Configuración inicial básica del switch Cisco 2960 (visualización de comandos en el entorno Cisco).

![Figura 3. Creación y ajuste de la VLAN 10.](ruta/de/la/imagen.png)
**Figura 3.** Creación y ajuste de la VLAN 10 en el switch Cisco 2960.

![Figura 4. Configuración de la VLAN PcMonitor.](ruta/de/la/imagen.png)
**Figura 4.** Configuración de la VLAN identificada con el nombre “PcMonitor”.

![Figura 5. Configuración de red en la Raspberry.](ruta/de/la/imagen.png)
**Figura 5.** Ajuste de parámetros de red en la Raspberry para la comunicación con otros dispositivos a través del switch.

![Figura 6. Configuración IP del PC Monitor.](ruta/de/la/imagen.png)
**Figura 6.** Asignación y verificación de la dirección IP en el PC Monitor.

![Figura 7. Prueba de ping desde PC Monitor (1).](ruta/de/la/imagen.png)
**Figura 7.** Prueba de conectividad desde el PC Monitor mediante el comando *ping* (primera verificación).

![Figura 8. Prueba de ping desde PC Monitor (2).](ruta/de/la/imagen.png)
**Figura 8.** Confirmación adicional de conectividad desde el PC Monitor usando el comando *ping* (segunda verificación).

![Figura 9. Conexión con el PC ETM.](ruta/de/la/imagen.png)
**Figura 9.** Verificación de conexión entre la Raspberry y el equipo ETM a través del comando *ping*.

![Figura 10. Escaneo de red con NMAP.](ruta/de/la/imagen.png)
**Figura 10.** Ejecución del comando *NMAP* para el análisis de la red.

![Figura 11. Puerto 22 cerrado.](ruta/de/la/imagen.png)
**Figura 11.** Comprobación del estado del puerto 22, el cual se encuentra cerrado durante el intento de transferencia de archivos.

![Figura 12. Instalación de Ubuntu (1).](ruta/de/la/imagen.png)
**Figura 12.** Proceso de instalación de Ubuntu – Etapa 1.

![Figura 13. Instalación de Ubuntu (2).](ruta/de/la/imagen.png)
**Figura 13.** Proceso de instalación de Ubuntu – Etapa 2.

![Figura 14. Instalación de Ubuntu (3).](ruta/de/la/imagen.png)
**Figura 14.** Proceso de instalación de Ubuntu – Etapa 3.

![Figura 15. Instalación de Ubuntu (4).](ruta/de/la/imagen.png)
**Figura 15.** Proceso de instalación de Ubuntu – Etapa 4.

![Figura 16. Instalación de Ubuntu (5).](ruta/de/la/imagen.png)
**Figura 16.** Proceso de instalación de Ubuntu – Etapa 5.

![Figura 17. Instalación de Ubuntu (6).](ruta/de/la/imagen.png)
**Figura 17.** Proceso de instalación de Ubuntu – Etapa 6.

![Figura 18. Instalación de Ubuntu (7).](ruta/de/la/imagen.png)
**Figura 18.** Proceso de instalación de Ubuntu – Etapa 7.

![Figura 19. Instalación de Ubuntu (8).](ruta/de/la/imagen.png)
**Figura 19.** Proceso de instalación de Ubuntu – Etapa 8.

![Figura 20. Instalación de Ubuntu (9).](ruta/de/la/imagen.png)
**Figura 20.** Proceso de instalación de Ubuntu – Etapa 9.

![Figura 21. Instalación de CentOS (1).](ruta/de/la/imagen.png)
**Figura 21.** Instalación del sistema operativo CentOS – Etapa 1.

![Figura 22. Instalación de CentOS (2).](ruta/de/la/imagen.png)
**Figura 22.** Instalación del sistema operativo CentOS – Etapa 2.

![Figura 23. Instalación de CentOS (3).](ruta/de/la/imagen.png)
**Figura 23.** Instalación del sistema operativo CentOS – Etapa 3.

![Figura 24. Creación de la máquina virtual para Alpine.](ruta/de/la/imagen.png)
**Figura 24.** Creación de la máquina virtual para la instalación de Alpine.

![Figura 25. Creación del disco virtual para Alpine.](ruta/de/la/imagen.png)
**Figura 25.** Generación del disco virtual destinado a la máquina Alpine.

![Figura 26. Instalación de Alpine (1).](ruta/de/la/imagen.png)
**Figura 26.** Inicio del proceso de instalación de Alpine – Etapa 1.

![Figura 27. Instalación de Alpine (2).](ruta/de/la/imagen.png)
**Figura 27.** Inicio del proceso de instalación de Alpine – Etapa 2.

![Figura 28. Configuración de usuario en Alpine.](ruta/de/la/imagen.png)
**Figura 28.** Configuración inicial del usuario dentro del sistema Alpine.

![Figura 29. Configuración interfaz y disco (1).](ruta/de/la/imagen.png)
**Figura 29.** Ajuste de la interfaz de usuario y del disco – Parte 1.

![Figura 30. Configuración interfaz y disco (2).](ruta/de/la/imagen.png)
**Figura 30.** Ajuste de la interfaz de usuario y del disco – Parte 2.

![Figura 31. Configuración interfaz y disco (3).](ruta/de/la/imagen.png)
**Figura 31.** Ajuste de la interfaz de usuario y del disco – Parte 3.

![Figura 32. Instalación de Alpine finalizada.](ruta/de/la/imagen.png)
**Figura 32.** Instalación de Alpine completada exitosamente.

![Figura 33. Instalación manual de Scientific Linux.](ruta/de/la/imagen.png)
**Figura 33.** Creación manual de la instalación de Scientific Linux y configuración del disco virtual.

![Figura 34. Pantalla de inicio de Scientific Linux.](ruta/de/la/imagen.png)
**Figura 34.** Pantalla de inicio del sistema Scientific Linux al comenzar la instalación.

![Figura 35. Configuración de Scientific Linux.](ruta/de/la/imagen.png)
**Figura 35.** Configuración de red y otros parámetros durante la instalación de Scientific Linux.

![Figura 36. Instalación de Scientific Linux finalizada.](ruta/de/la/imagen.png)
**Figura 36.** Instalación de Scientific Linux finalizada.



