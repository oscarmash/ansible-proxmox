# Playbook: roles_alta_equipo.yml

## Funcionamiento

Tan simple como lanzarlo:

```bash
$ ansible-playbook roles_alta_equipo.yml
Nombre del equipo [borrame]:
IP del equipo [22]:
Promox de destino [host2]:
Memoria equipo [2048]:
Sockets del equipo [1]:
Cores del equipo [2]:
Equipo a clonar [9999]:
ID del equipo:
```

## Prerequisitos

Hay que instalar en nuestro equipo el paquete de pip para que pueda acceder a Vault

```
pip install hvac
```

## Install other host

Para instalar uno de los roles en un determinado host:

Para Debian:

```
ansible-playbook roles_alta_equipo.yml -i 172.26.0.58, -t neofetch
```

Para Ubuntu:

```
ansible-playbook roles_alta_equipo.yml -i 172.26.0.3, -k -u oscar --tags neofetch --become --ask-become-pass
```