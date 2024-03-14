# born2beroot
*/

Este proyecto no tiene código. Estos son apuntes para preparar la evaluación correspondiente.
¿Qué es una máquina virtual?
Una máquina virtual es un entorno de software que simula una computadora física y permite ejecutar sistemas operativos y aplicaciones como si estuvieran en hardware real. Esto proporciona flexibilidad y aislamiento, lo que permite ejecutar múltiples sistemas operativos en una sola máquina física.

¿Por qué has escogido Debian?
Debian es una opción popular debido a su estabilidad, amplia selección de paquetes de software y su compromiso con el software libre. Además, Debian tiene una gran comunidad de usuarios y una amplia documentación disponible, lo que facilita el aprendizaje y la resolución de problemas.

Diferencias básicas entre Rocky y Debian
Rocky Linux y Debian son distribuciones de Linux que difieren en su base y enfoque. Rocky Linux se basa en el código fuente de Red Hat Enterprise Linux (RHEL), mientras que Debian tiene su propia base de desarrollo. En términos de gestión de paquetes, Rocky Linux utiliza DNF (Dandified YUM) como administrador de paquetes, mientras que Debian utiliza apt (Advanced Package Tool). Además, Rocky Linux está diseñado para ser una alternativa gratuita y de código abierto a RHEL, mientras que Debian es conocido por su compromiso con el software libre y su amplia comunidad de desarrollo.

¿Cuál es el propósito de las máquinas virtuales?
El propósito principal de las máquinas virtuales es permitir la ejecución de múltiples sistemas operativos en un solo hardware físico. Esto proporciona flexibilidad y aislamiento para probar software, desarrollar y ejecutar aplicaciones, y crear entornos de pruebas seguros. Las máquinas virtuales también son útiles para la consolidación de servidores, la recuperación de desastres y la optimización de recursos de hardware.

Diferencias entre apt y aptitude
Apt y aptitude son dos herramientas de gestión de paquetes para sistemas Debian y derivados. Apt es una interfaz de línea de comandos para administrar paquetes de software, mientras que aptitude es una interfaz de usuario basada en texto con características avanzadas como la resolución de dependencias y la búsqueda de paquetes. Aptitude utiliza apt como backend para realizar acciones de instalación, actualización y eliminación de paquetes.

¿Qué es AppArmor?
AppArmor es un sistema de control de acceso obligatorio (MAC) para el kernel Linux que permite restringir los permisos de acceso de los programas a recursos del sistema. AppArmor utiliza perfiles de seguridad para definir qué acciones puede realizar un programa y qué recursos puede acceder. Esto ayuda a proteger el sistema contra ataques maliciosos y a mitigar los riesgos de seguridad.

¿Qué es LVM?
LVM, o Logical Volume Manager, es una herramienta de gestión de almacenamiento que permite administrar volúmenes lógicos en sistemas Linux. Con LVM, los administradores pueden crear, redimensionar y administrar volúmenes lógicos, lo que proporciona flexibilidad y escalabilidad para la gestión del almacenamiento. LVM permite agregar y quitar discos de forma dinámica sin interrumpir el sistema y facilita la realización de copias de seguridad y la migración de datos.


El script proporcionado es un script de Bash que recopila información sobre la arquitectura del sistema operativo, el uso de la CPU, la memoria RAM, el disco, el estado del LVM, las conexiones TCP, los usuarios conectados, la red y el número de comandos ejecutados con sudo. A continuación, explicaré brevemente cada parte del script:

bash
Copy code
#!/bin/bash

# ARCH
arch=$(uname -a)
uname -a muestra información sobre el sistema, incluida la arquitectura del hardware y la versión del kernel. La salida se asigna a la variable arch.
bash
Copy code
# CPU PHYSICAL
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)
grep "physical id" /proc/cpuinfo busca líneas que contengan "physical id" en el archivo /proc/cpuinfo, que proporciona información sobre el procesador. wc -l cuenta el número de líneas coincidentes, lo que da el número de núcleos físicos.
bash
Copy code
# CPU VIRTUAL
cpuv=$(grep "processor" /proc/cpuinfo | wc -l)
Similar al anterior, pero busca líneas que contengan "processor" para contar el número de núcleos virtuales.
bash
Copy code
# RAM
ram_total=$(free --mega | awk '$1 == "Mem:" {print $2}')
ram_use=$(free --mega | awk '$1 == "Mem:" {print $3}')
ram_percent=$(free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
free --mega muestra información sobre el uso de la memoria en megabytes. awk filtra la línea que comienza con "Mem:" y extrae el total de RAM y el uso actual, y luego calcula el porcentaje de uso.
bash
Copy code
# DISK
disk_total=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}')
disk_use=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}')
disk_percent=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}')
df -m muestra información sobre el espacio en disco en megabytes. Los comandos grep y awk filtran y suman el espacio total utilizado y el total, luego calculan el porcentaje de uso.
bash
Copy code
# CPU LOAD
cpul=$(vmstat 1 2 | tail -1 | awk '{printf $15}')
cpu_op=$(expr 100 - $cpul)
cpu_fin=$(printf "%.1f" $cpu_op)
vmstat 1 2 muestra estadísticas de la CPU. tail -1 toma solo la última línea, y awk extrae la carga de la CPU. Luego se calcula el porcentaje de CPU no utilizado.
bash
Copy code
# LAST BOOT
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
who -b muestra la fecha y hora del último arranque del sistema. awk filtra y muestra la fecha y hora.
bash
Copy code
# LVM USE
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)
Verifica si LVM está en uso buscando la cadena "lvm" en la salida de lsblk.
bash
Copy code
# TCP CONNEXIONS
tcpc=$(ss -ta | grep ESTAB | wc -l)
ss -ta muestra las conexiones TCP. grep ESTAB filtra las conexiones establecidas. wc -l cuenta el número de líneas.
bash
Copy code
# USER LOG
ulog=$(users | wc -w)
users muestra los nombres de los usuarios conectados. wc -w cuenta el número de palabras, que corresponde al número de usuarios.
bash
Copy code
# NETWORK
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')
hostname -I muestra la dirección IP del host. ip link muestra información sobre las interfaces de red, y awk extrae la dirección MAC.
bash
Copy code
# SUDO
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
journalctl _COMM=sudo muestra registros relacionados con comandos ejecutados con sudo. grep COMMAND filtra las líneas relevantes, y wc -l cuenta el número de líneas.
bash
Copy code
# Wall message
wall "	Architecture: $arch
	CPU physical: $cpuf
	vCPU: $cpuv
	Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	Disk Usage: $disk_use/${disk_total} ($disk_percent%)
	CPU load: $cpu_fin%
	Last boot: $lb
	LVM use: $lvmu
	Connections TCP: $tcpc ESTABLISHED
	User log: $ulog
	Network: IP $ip ($mac)
	Sudo: $cmnd cmd"
El mensaje final se envía a todos los usuarios a través del comando wall, mostrando la información recopilada.
*/
