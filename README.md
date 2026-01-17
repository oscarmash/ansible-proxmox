# Playbooks

Listado de repositorios de playbooks

* [Alta de VM en Proxmox](README.roles_alta_equipo.md): Alta de equipos virtuales de Proxmox
* [Update SO VM](README.update_so_vm.md): Actualización de equipos virtuales de Proxmox

# Creación de plantillas

Nomenclatura de VM en Proxmox:

| Num.   |  Servicio  |
|:------:|:----------:|
| 1xxx   | producción |
| 5xxx   | testing    |
| 9xxx   | plantillas |

* [Plantillas](#id10)
  * [Debian 13](#id11)

# Plantillas <div id='id10' />

## Debian 13 <div id='id11' />

```
root@host1:~# mkdir /tmp/template_debian13 && cd /tmp/template_debian13
root@host1:/tmp/template_debian13# wget https://cloud.debian.org/images/cloud/trixie/latest/debian-13-genericcloud-amd64.qcow2
```

```
virt-customize -a debian-13-genericcloud-amd64.qcow2 \
--update \
--install qemu-guest-agent,curl,wget,vim,rsync,htop,iputils-ping,net-tools,telnet,nmap,gpg,procps,sudo,ncdu,git,lsb-release \
--timezone "Europe/Madrid" \
--run-command 'sed -i s/^#PermitRootLogin.*/PermitRootLogin\ prohibit-password/ /etc/ssh/sshd_config' \
--run-command 'sed -i s/^PasswordAuthentication.*/PasswordAuthentication\ yes/ /etc/ssh/sshd_config' \
--run-command "sed -i 's/disable_root: [Tt]rue/disable_root: False/' /etc/cloud/cloud.cfg" \
--root-password password:r00tme \
--run-command "echo 'net.ipv6.conf.all.disable_ipv6=1' >> /etc/sysctl.conf" \
--run-command "echo 'net.ipv6.conf.default.disable_ipv6=1' >> /etc/sysctl.conf" \
--run-command "sysctl -p"
```

```
qemu-img convert -O qcow2 -c -o preallocation=off debian-13-genericcloud-amd64.qcow2 debian-13-genericcloud-amd64-shrink.qcow2
```

```
qm create 9013 \
--name debian13 \
--cores 2 \
--cpu "cputype=x86-64-v2-AES" \
--memory 4096 \
--net0 virtio,bridge=LAN_SFP \
--agent "enabled=1,fstrim_cloned_disks=1" \
--scsihw "virtio-scsi-pci" \
--scsi1 "host1-SSD-256GB:cloudinit" \
--boot "order=scsi0" \
--ciuser "oscar" \
--cipassword "C@dinor1988" \
--ipconfig0 "ip=172.26.0.22/24,gw=172.26.0.1" \
--nameserver "172.26.0.11 8.8.8.8" \
--searchdomain ilba.cat \
--sshkeys <(echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCc8E249Z9GKz63xcozQ3siA+/SXnw2TrhbMigJiESh2PYDC2CiqUFaSM07BM9bGwDbtomAzilKSigifNmzV1jlKgRe9165ngidmOJ+rEa7WwTXV3gl0AYArx6dmxkG2KTckSFhzEiPdiSvlb/0UswXiN/OWyPsjrcFZBewnpIKhrvYYsAN/u2vJjG1x74Oc4jdmId7jZMAdhImz0eE1UMpOiOUHZMAJ8W1BK/4T/PU1pzulDRi2qltWX21kQnQkT1fe7BSxXQUx4Ea80H5MszhuCJn8IbPSUFniq3hYNwNz6dzpoQJ7o9kMB8CeI+WBRCoBM1vGkWZB2+IaVqzAbLlBTeVPSEzhtVndVMI0gbCa9AxzWMG6+PxWQjFUFlDxuSkrs+3cavG41PsqQMpug6Kf4o5qINPblSaEOkuJLpaQDb4AhCCSFpK/aH7szOpRmwLGlqJNtMnL9Hf1+mNNymVjyF8TUOmLKPqyyDvkFFo3SSx/KHWr1EslHhSKW/HKy0= oscar.mas@ilimit.net")
```

```
qm importdisk 9013 debian-13-genericcloud-amd64-shrink.qcow2 host1-SSD-256GB --format qcow2
qm set 9013 --virtio0 host1-SSD-256GB:vm-9013-disk-0,discard=on --boot order=virtio0
qm resize 9013 virtio0 60G
```

```
qm template 9013
```