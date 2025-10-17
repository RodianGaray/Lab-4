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
## Guía de Procesos de Instalación
Proceso Ubuntu
Preparación: Descargar ISO, crear disco virtual

Instalación: Seguir asistente gráfico

Configuración: Usuario, red, particiones

Post-instalación: Actualizaciones, herramientas

Proceso CentOS
Medios: ISO de CentOS Stream

Instalación: Anaconda installer

Configuración: SELinux, firewall, repositorios

Verificación: cat /etc/redhat-release

Proceso Alpine
Minimalismo: ISO pequeña (~100MB)

Instalación: Modo texto

Configuración: setup-alpine script

Características: musl libc, BusyBox

Proceso Scientific Linux
Origen: Derivado de RHEL

Instalación: Similar a CentOS

Enfoque: Comunidad científica

Características: Compatibilidad con software científico

4. Verificación y Documentación
