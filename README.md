# Ecosistema de Interoperabilidad para SIMSADI® 

Este repositorio contiene los archivos necesarios para poder desplegar el servicio Mirth Connect de la empresa NextGen y un servidor HAPI FHIR para el Proyecto SIMSADI UV: Ecosistema de Interoperabilidad

Cabe destacar que este repositorio esta enfocado para el usuario final del módulo de los servicios (en otras palabras, te ayuda a implementar estos servicios, por lo tanto ciertas librerías y extensiones pueden que no estén incluídas en esta instalación). Si estás en busca del proyecto fuente de cada servicio,  puedes consultar directamente desde los repositorios oficiales de [Mirth Connect](https://github.com/nextgenhealthcare/connect) y de [HAPI-FHIR](https://github.com/hapifhir/hapi-fhir-jpaserver-starter)


## Pre-requisitos de instalación.
- Ordenador con los siguientes especificaciones:
  - Hardware x86, los modelos actuales de Intel/AMD deberían soportarlo.
  - RAM: 8 GB mínimo
  - Almacenamiento: 30 GB mínimo dedicado
  - Sistema Operativo Ordenador: Windows 10
- Maquina Virtual con SO [`CentOS Linux 7.6.1810 (x86_64)`](http://ftp.iij.ad.jp/pub/linux/centos-vault/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso) 

## Guía de implementación

1 Instalación VirtualBox y configuracion inicial

En orden para poder desplegar los servicios, se requiere de la instalación de [VirtualBox](https://www.virtualbox.org/), el cual nos permitirá crear una maquina virtual localmente en tu ordenador. Se debe acceder al sitio oficial de VirtualBox y seguir las instrucciones de descarga y del instalador.
Se debe seleccionar nueva en la pestaña dentro de virtualbox para poder configurar la maquina virtual a crear con los siguientes parametros.

**Crear maquina virtual**
|||
--- | --- 
|Nombre| CentOS 7.6 |
| Tipo      | Linux |
| Version      | RedHat(64-bit)     | 
| Tamaño de memoria | 2048 MB      | 
| Disco duro      | Crear un disco duro virtual ahora         |

**Crear de disco duro virtual**
|||
--- | --- 
|Tamaño del archivo| 20 GB |
| Tipo de archivo     | VDI |
| Almacenamiento   | Reservado dinámicamente    | 

La maquina virtual ya esta creada en VirtualBox, ahora se debe cargar la imagen ISO CentOS 7.6 en la maquina virtual, se debe hacer click derecho sobre la maquina virtual creada >Configuraciones>Almacenamiento>Controlador:IDE>Vacío> se debe seleccionar el ícono de disco a la derecha de la ventana, luego click sobre "Seleccionar archivo de disco óptico virtual" y se debe cargar la imagen `CentOS-7-x84_64-DVD-1810.iso`

Luego de haber cargado la imagen del SO CentOS 7.6 se procede a inicializar la maquina virtual, lo que derivará en el asistente de instalación de CentOS, donde se establecerán los siguientes parámetros.

- Selección de idioma: `Español, Chile`
- Seleccion de Software
  - Entorno Base: `Instalacion mínima`
- Destino de la Instalación
  - Mantener el disco duro predeterminado ya seleccionado - haz click sobre el recuadro <samp>Listo</samp> en la parte superior.
- Empezar Instalación


Durante la instalacion, se debe configurar los siguientes ajustes de usuario

- Crear clave de administrador root
- Crear usuario y clave
  - [x] Hacer que este usuario sea administrador
- Al finalizar la instalación, se debe reiniciar la maquina virtual y a continuación se deberá aceptar el acuerdo de licencia.

Luego de finalizada la configuración de instalación de CentOS, se debe acceder al terminal de la máquina virtual y se deberan realizar las configuraciones de redes. Es decir, la configuración de la conexión a internet y el "Tunneling" o redirección de puertos.

```
# CONEXION A INTERNET
# Para verificar si la maquina virtual está conectada a internet se debe utilizar el comando “nmcli” en el terminal
nmcli
# Si enp0s3: desconectado, se debe acceder al archivo ifcfg-enp0s3 y editarlo. El comando vi permite visualizar y editar archivos
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
# Para poder entrar en modo edición del archivo se debe presionar la tecla ”i” 
i
# luego se debe identificar la variable ONBOOT, la cual debe ser editada ONBOOT:no a ONBOOT:yes, seguido presionar esc para salir del modo edición, presionar Shift+ tecla ”:” y escribir :wq, este comando guarda los cambios realizados y cierra el archivo visualizado.
:wq
sudo systemctl restart network.service
nmcli
# ahora enp0s3 debe mostrar “conectado to enp0s3” 
```

Luego se deberá configurar la redireccion de puertos o "Tunneling" en orden para poder acceder a los servicios de la maquina virtual.
Esto se realiza en la sección Preferencias > Red > Redes NAT. debería aparecer un archivo "NatNetwork", si no es asi pincha sobre el icono "+" al costado del cuadro para crear una red NAT, al entrar al archivo nos dará la opcion abajo de "reenvio de puertos". Luego en la pestaña IPV4 se configuran las reglas de redireccion de puertos, se le puede dar nombre a la regla seguido del protocolo de comunicación, configurar la IP y puerto anfitrion (nuestra red) y luego la IP y puerto invitado (la red interna de la maquina virtual), como se muestra a continuacion.

![Aquí la descripción de la imagen por si no carga](https://github.com/matirafi/Mirth-SIMSADI/blob/main/github_images/Captura.JPG)

Entonces, los puertos mostrados anteriormente son los necesarios donde fueron desplegados los servicios de Mirth, HAPI-FHIR, el cliente SSH, Portainer y bases de datos. El cambio que se debe hacer es en las direcciones IP anfitrion e IP invitado, donde tu IP anfitrion debe ser tu direccion IPV4 dada por tu conexión a internet (en las propiedades de tu conexion a internet debe aparecer direccion ipv4) y la IP invitado está dada por la maquina virtual (esta se consulta mediane el comando `ip a`)


2 Instalacion del Cliente SSH: PuTTY.
En orden para instalar PuTTY se debe descargar directamente desde la pagina web [putty.org](https://www.putty.org/) y se seguir las instrucciones del instalador. Este programa nos permitirá conectarnos directamente con la maquina virtual, deben poner en Hostname la direccion IPv4 de su conexion de internet y el puerto para acceso es el 23(configurado previamente en el paso redireccion de puertos).

![Aquí la descripción de la imagen por si no carga](https://github.com/matirafi/Mirth-SIMSADI/blob/main/github_images/putty.JPG)


3 Instalación de Docker Engine.

Antes de comenzar con la instalacion de Docker Engine es recomendable revisar la documentacion oficial respecto a la instalacion de [Docker Engine en CentOS](https://docs.docker.com/engine/install/centos/). Luego debes revisar como pre-requisito del SO si el repositorio de `centos-extras` se encuentra habilitado correctamente, se recomienda ocupar el controlador de almacenamiento `overlay2` y verificar que no existan versiones anteriores de docker instaladas, si estuviesen instaladas debes desinstalaras.

```linux
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```
Para no tener problemas de actualizaciones respecto a los paquetes de CentOS, es recomendable actualizarlos antes de comenzar la instalacion de docker
```linux
sudo yum update
```
El metodo de instalacion a ocupar es mediante el repositorio de docker, el cual debe ser configurado como primera instancia. Se debe instalar el paquete `yum-utils` y luego la configuración.
```linux
sudo yum install -y yum-utils
sudo yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
```
Luego se instala la *ultima* version de Docker Engine, containerd y Docker Compose.
```centos
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
Ahora docker se encuentra instalado correctamente, pero no está iniciado.
```centos
# para verificar el estado de docker
sudo systemctl status docker
# para iniciar docker
sudo systemctl start docker
# para detener docker
sudo systemctl stop docker
```
Como ultimo paso adicional, para poder gestionar los contenedores de forma comoda utilizamos un software llamado Portainer, el cual se instala como un contenedor Docker de la siguiente forma.

```centos
docker run -d -p 8080:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```


4 Instalación Mirth Connect.

`docker-compose.yml`

```Java
version: "3.1"
services:
  mc:
    image: nextgenhealthcare/connect
    environment:
      - DATABASE=postgres
      - DATABASE_URL=jdbc:postgresql://172.16.255.61:5433/mirthdb
      - DATABASE_MAX_CONNECTIONS=20
      - DATABASE_USERNAME=mirthdb
      - DATABASE_PASSWORD=mirthdb
      - DATABASE_MAX_RETRY=2
      - DATABASE_RETRY_WAIT=10000
      - KEYSTORE_STOREPASS=Biomedica.123
      - KEYSTORE_KEYPASS=Biomedica.123
      - VMOPTIONS=-Xmx512m
    ports:
      - 8180:8080/tcp
      - 8443:8443/tcp
    depends_on:
      - db
  db:
    image: postgres
    environment:
      - POSTGRES_USER=mirthdb
      - POSTGRES_PASSWORD=mirthdb
      - POSTGRES_DB=mirthdb
    ports:
      - 5433:5432
```
