# Playbooks Ansible que instalan una estación de trabajo

A partir de una instalación base de Ubuntu se pueden agregar las herramientas populares para trabajar.

# 1. Cómo usar este repositorio sobre un nodo físico

Cómo instalar una notebook o PC con estas herramientas.

## 1.1. Instale desde DVD o mediante PXE un escritorio Ubuntu 20.04

1. `sudo` configurado para correr sin pedir contraseña con la cuenta que corre este script

Ejecutar:

```bash
  sudo visudo
```

Modificar el usuario con que va a correr Ansible y agregar: "NOPASSWD":

```yml
 %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

2. APT configurado (mirrors, acceso al mirror y acceso a fuentes de paquetes por internet)
3. Proxy configurado en `/etc/environment` (para curl, wget, etc.)

## 1.2. Instale y configure Git

```bash
sudo apt update
sudo apt install git
```

## 1.3. Clone de los repositorios

```bash
git clone https://github.com/CesarBallardini/ansible-devops-workstation.git
git clone https://github.com/al3camarasa/ansible-workstation.git
```

## 1.4. Instale los requisitos para que funcione ansible

```bash
sudo apt-get install -y python3-pip
sudo -H python3 -m pip install --upgrade pip setuptools wheel
sudo -H python3 -m pip install --upgrade ansible
ansible-galaxy collection install community.general
```

Verificado con Ansible versión 2.9.6, python version 2.7.18rc1

## 1.5. Instalar con Ansible el resto del software

```bash
cd ansible-devops-workstation/
ansible-galaxy install -r requirements.yml -p roles/
```

A partir de este momento, el resto de las actividades las realizaremos desde ese directorio.

## 1.6. Crear un inventario para su local

* el directorio para el inventario:

```bash
mkdir -p inventario/{group_vars,host_vars}
cp vagrant-inventory/host_vars/devopsws inventario/host_vars/localhost
```

* En el inventario ```inventario/hosts```, agregar la dirección ```localhost``` correspondiente al perfil del equipo.

```bash
cat - > inventario/hosts  <<EOF
localhost ansible_connection=local

[ejemplo]

[tinyproxy]

[servidor]
localhost

[escritorio]
localhost

[devops]
localhost

[utn]

[ocsinventoryagent]

[python3]
localhost

[python3:vars]
ansible_python_interpreter=/usr/bin/python3
EOF
```

Modificar en el archivo ```inventario/host_vars/localhost``` las variables:

- ```devops_user_name```: con nuestro propio nombre de usuario.
- ```devops_user_uid```: si nuestro uid es dististo de 1000.
- En caso que se utilice ```proxy```:
  - organizacion: nombre.
  - all_proxy: 'http://IP:3128'.
  - http_proxy: 'http://IP:3128'.
  - https_proxy: 'http://IP:3128'.
  - ftp_proxy: 'http://IP:3128'.
  - no_proxy: ''.
  - soap_use_proxy: ''.

Luego ejecutamos Ansible:

```bash
time ansible-playbook -vv -i inventario/hosts site.yml --limit localhost --tags proxy,performance,locales,snap,ansible,git,virtualbox,vagrant,docker,microsoft_visualstudio_code
```

# 2. Instalación de los aplicativos custom

```bash
cd ..
cd ansible-workstation/
ansible-galaxy install -r requirements.yml -p roles/
```

* Como en el anterior inventario ```inventario/hosts```, agregar la dirección **localhost** al perfil que corresponde al equipo.

```bash
cat - > inventario/hosts  <<EOF
# hosts
localhost ansible_connection=local

[custom]

[homeoffice]

[escritorio]
localhost

[devops]
localhost

[job]
localhost

[python3]
localhost

[servidor]

[python3:vars]
ansible_python_interpreter=/usr/bin/python3
EOF
```

Crear el archivo ```inventario/host_vars/office``` las variables:

- ```devops_user_name```: con nuestro propio nombre de usuario.
- ```devops_shell```: bashc
- ```dock_icon_size```: 34
- ```mouse_speed```: 0.25735294117647056
- ```office_ntp_servers```: [ ntp1, ntp2, ... ]
- ```office_git```: nombre del repositorio git.

Luego ejecutamos Ansible:

```bash
time ansible-playbook -vv -i inventario/hosts site.yml --extra-vars "@inventario/host_vars/[nombre-host]" --limit localhost
```

# 3. Cómo comprobar los playbooks mediante Vagrant y VirtualBox

## 3.1. Configure su estación de trabajo como en el punto 1. anterior

Si lo hace manualmente, debe tener instalados:

* Oracle Virtualbox y Oracle VM VirtualBox Extension Pack
* Vagrant
* Plugins de Vagrant:
  * vagrant-proxyconf y su configuracion si requiere de un Proxy para salir a Internet
  * vagrant-cachier
  * vagrant-disksize
  * vagrant-hostmanager
  * vagrant-share
  * vagrant-vbguest

## 3.2. Ejecute Vagrant

```bash
time vagrant up --provision
```

Se usarán las configuraciones de Proxy de la estación de trabajo y el inventario (con sus variables) disponible
en `vagrant-inventory`.

`vagrant-inventory` tiene los archivos:

```text
vagrant-inventory/
├── ansible.cfg
├── hosts
└── host_vars
    └── devopsws
```

```bash
cp hosts-vars.yml vagrant-inventory/host_vars/devopsws
time ansible-playbook -vv -i vagrant-inventory/hosts site.yml --limit devopsws
```
# 4. Referencias

[link to Repo CesarBallardini](https://github.com/CesarBallardini/ansible-devops-workstation).
