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

Si no tienes llave
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

Instalar colecciones de ansible
```text
$ ansible-galaxy collection install ansible.posix
$ ansible-galaxy collection install containers.podman
```


### Configuraciones de secretos y contraseñas

Editar el archivo vars/secrets.yml y configurar las contraseñas deseadas.
```text
---

# password usuario keycloak
keycloak_user_password_hash: "$6$bJMuTHP0yU4lWSnV$jv5E7NPdWhimgw.Liu.r.OKjwv5vwEVlI3vHjG.cih4ulAVWptKTct36Sz8evozqnQ5l6mCbmm//RuLJHD9xk1"

# password keepalived
keepalived_auth_pass: StrongVRRPPass

# postgres passwords
postgres_superuser_password: StrongPass
postgres_replicator_password: ReplicatorPass

# keycloak postgres database password
keycloak_db_password: kcpass

# keycloak admin password
keycloak_admin_password: admin
```


### Configurar networking

Editar el archivo vars/servers.yml y configurar los valores adecuados para cada servidor.
```text
---
servers:
  server1:
    backend_ip: 10.10.10.21
    frontend_ip: 192.168.56.21
    keepalived_priority: 200

  server2:
    backend_ip: 10.10.10.22
    frontend_ip: 192.168.56.22
    keepalived_priority: 150

  server3:
    backend_ip: 10.10.10.23
    frontend_ip: 192.168.56.23

network:
  backend_zone: backend
  frontend_zone: frontend
  frontend_interface: ens224
  frontend_nm_connection: ens224
  frontend_vip: 192.168.56.20
  keycloak_domainname: auth.turrillas.net

```

- backend_ip: IP del servidor para la red interna
- frontend_ip: IP del servidor para la red publica
- frontend_interface: interfaz de red de la red publica
- frontend_nm_connection: nombre de la conexión de Network Manager para la red de frontend
- frontend_vip: Direccion IP Virtual para el servicio
- keycloak_domainname: FQDN del servicio keycloak. en el DNS debe apuntar a la IP frontend_vip



## Instalación

### Configuración de zonas de firewall

```text
$ ansible-playbook playbooks/10-firewalld-zones.yml 
[WARNING]: Collection ansible.posix does not support Ansible version 2.14.18

PLAY [Configurar firewalld] ***********************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************
ok: [server3]
ok: [server2]
ok: [server1]

TASK [Crear zonas] ********************************************************************************************************************************************************
changed: [server2] => (item=frontend)
changed: [server3] => (item=frontend)
changed: [server1] => (item=frontend)
changed: [server2] => (item=backend)
changed: [server1] => (item=backend)
changed: [server3] => (item=backend)

TASK [Recargar firewalld] *************************************************************************************************************************************************
changed: [server2]
changed: [server3]
changed: [server1]

TASK [Set frontend zone in NetworkManager] ********************************************************************************************************************************
changed: [server3]
changed: [server1]
changed: [server2]

TASK [reload connection] **************************************************************************************************************************************************
changed: [server1]
changed: [server3]
changed: [server2]

TASK [Asociar interfaz frontend] ******************************************************************************************************************************************
changed: [server1]
changed: [server2]
changed: [server3]

TASK [Asociar IPs a zona backend] *****************************************************************************************************************************************
changed: [server1] => (item={'key': 'server1', 'value': {'backend_ip': '10.10.10.21', 'frontend_ip': '192.168.56.21', 'keepalived_priority': 200}})
changed: [server2] => (item={'key': 'server1', 'value': {'backend_ip': '10.10.10.21', 'frontend_ip': '192.168.56.21', 'keepalived_priority': 200}})
changed: [server3] => (item={'key': 'server1', 'value': {'backend_ip': '10.10.10.21', 'frontend_ip': '192.168.56.21', 'keepalived_priority': 200}})
changed: [server1] => (item={'key': 'server2', 'value': {'backend_ip': '10.10.10.22', 'frontend_ip': '192.168.56.22', 'keepalived_priority': 150}})
changed: [server2] => (item={'key': 'server2', 'value': {'backend_ip': '10.10.10.22', 'frontend_ip': '192.168.56.22', 'keepalived_priority': 150}})
changed: [server3] => (item={'key': 'server2', 'value': {'backend_ip': '10.10.10.22', 'frontend_ip': '192.168.56.22', 'keepalived_priority': 150}})
changed: [server3] => (item={'key': 'server3', 'value': {'backend_ip': '10.10.10.23', 'frontend_ip': '192.168.56.23'}})
changed: [server1] => (item={'key': 'server3', 'value': {'backend_ip': '10.10.10.23', 'frontend_ip': '192.168.56.23'}})
changed: [server2] => (item={'key': 'server3', 'value': {'backend_ip': '10.10.10.23', 'frontend_ip': '192.168.56.23'}})

TASK [Recargar firewalld] *************************************************************************************************************************************************
changed: [server1]
changed: [server2]
changed: [server3]

PLAY RECAP ****************************************************************************************************************************************************************
server1                    : ok=8    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server2                    : ok=8    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server3                    : ok=8    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Validar configuracion
```text
$ ansible -m command -a "firewall-cmd --list-all --zone=backend" server1
server1 | CHANGED | rc=0 >>
backend (active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 10.10.10.21 10.10.10.22 10.10.10.23
  services: 
  ports: 
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
$ ansible -m command -a "firewall-cmd --list-all --zone=frontend" server1
server1 | CHANGED | rc=0 >>
frontend (active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: ens224
  sources: 
  services: 
  ports: 
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```




