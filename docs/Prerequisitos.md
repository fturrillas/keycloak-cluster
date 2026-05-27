# Prerequisitos

Es posible que no todos estos requisitos son obligatorios, pero está es la configuración que funciona sin hacer cambios en los archivos de configuracion.


## Tareas previas a la instalación

Algunos de los prerequisitos para la instalación son: 

- Instalar RHEL 10 en los servidores
- Tener los servidores registrados con una suscripción activa
- Actualizar el sistema operativo
- Configuración de red completa en los servidores
- Crear un usuario con permisos de sudo en los servidores y copiar una llave ssh en el usuario para conectarse sin contraseña


## Networking

Los scripts de instalación asume que en los servidores hay 3 interfaces de red

- gestion
- red de front end 
- red de backend

y de acuerdo a esto configuran los permisos en firewalld.


# Configuración Objetivo

## Usuario y directorios de instalación

- Se crea el usuario keycloak y dentro de ese usuario corren los procesos. 
- La instalación de cada aplicación se ejecuta sobre el directorio /opt/keycloak. Por ejemplo postgres en /opt/keycloak/postgres. 
- En el host se crean directorios locales y se comparten como volumenes hacia los contenedores.
- Al usuario keycloak se le dan permisos de linger para poder correr los contenedores sin que sea necesario tener una sesión activa en el servidor
- Se crean servicios systemd de usuario para administrar los contenedores con systemctl

## Configuracion de red

### Zona Backend

Se crea la zona de firewall backend con las IPs de los servidores de backend para la comunicación interna de los servicios. Sobre esta red se habilitan los puertos para la comunicación de postgres, patroni y etcd 


### Zona Frontend

- Se crea la zona de firewall fontend para la recepción de los requerimientos, se asume que hay una interfaz de red dedicada al frontend
- Se configurar SSL para keycloak, se habilita el puerto 443/tcp para recibir los requerimientos
- Se configura el certificado SSL en la instalación de haproxy
- Si se requiere hacer cambios revisar la instalación de haproxy


## Servicios

### Servicio etcd

### Servicio postgres + patroni

### Servicio haproxy

### Servicio keycloak







