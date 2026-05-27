# Instalacion de keycloak en cluster

## Arquitectura y Prerequisitos

Revisar antes:
- Arquitectura
- Prerequisitos

## Configuraciones previas

### Usuario de instalación

Previo a la instalación es necesario tener creado un usuario en cada servidor con permisos de sudo, lo mas simple es que tenga tambien NOPASSWD: ALL en su configuración. Después de la instalación se pueden eliminar estos permisos.

También se requiere copiar la llave de ssh para que no pida password durante la instalación

El nombre de este usuario debe ser configurado en el archivo ansible.cfg en la opción remote_user. Por omisión está configurado el usuario ansible.

Si no tienes llave1
```text
ssh-keygen -t dsa
```

Copia la llave a cada uno de los servidores
```text
ssh-copy-id ansible@server1
ssh-copy-id ansible@server2
ssh-copy-id ansible@server3
```

### Configuraciones Ansible

Debe configurarse apropiadamente el archivo ansible.cfg e inventory. En ansible.cfg por omisión el remote_user es ansible, tiene llave de ssh instalada y el usuario remoto tiene permisos de sudo sin contraseña

En el archivo de inventory van los servidores: server2, server2 y server3. Lo ideal es mantener los nombres de los servidores, ya que se usan en la configuracion de  algunos playbooks. Si no se pueden configurar los nombres en el /etc/hosts. se pueden configurar las IPs en el inventory, por ejemplo:

```text
[servers]
server1 ansible_host=192.168.202.136
server2 ansible_host=192.168.202.149
server3 ansible_host=192.168.202.150
```

Probar que ansible está correctamente configurado
```text
$ ansible -m command -a "id" all
server2 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
server3 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
server1 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```


