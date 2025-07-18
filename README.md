# Playbooks Ansible que instalan una estación de trabajo

A partir de una instalación base de Ubuntu se pueden agregar las herramientas populares para trabajar.

# 1. Cómo usar este repositorio sobre un nodo físico

Cómo instalar una notebook o PC con estas herramientas.

## 1.1. Instale y configure Git

```bash
sudo apt update
sudo apt install git
```

## 1.2. sudo configurado para correr sin pedir contraseña
```bash
sudo visudo
```
modificando la línea:

```yml
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

## 1.3 Verificar la existencia del archivo de Swap
```bash
ls /swap.img
```
En el caso de que no exista, [crearlo](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04).

## 1.3. Clone de los repositorios

```bash
git clone https://github.com/CesarBallardini/ansible-devops-workstation.git
git clone https://github.com/al3camarasa/ansible-workstation.git
```

## 1.3. Instale los requisitos para que funcione Ansible 2.15
Si se desea instalar la versión de Ansible 2.15, ingrese [aquí](python_virtual_ansible_2_15.md).


## 1.4. Instale los requisitos para que funcione Ansible superior a 2.15
Agregar la variable con el path del virtualenv de Python en el `.barhrc`
```bash
PYTHON_VENV_ANSIBLE_2_18=/usr/local/venv-ansible-2.18
```

Luego, cierra y vuelve a abrir la terminal para que se carguen las variables de entorno.

```bash
PYTHON_VENV_ANSIBLE_PATH=$PYTHON_VENV_ANSIBLE_2_18
PYTHON_VENV_ANSIBLE="$PYTHON_VENV_ANSIBLE_2_18/bin"
sudo apt-get install -y python3-pip python3.12-venv
sudo mkdir $PYTHON_VENV_ANSIBLE_PATH
[ -d "${PYTHON_VENV_ANSIBLE}" ] || sudo python3 -m venv "${PYTHON_VENV_ANSIBLE_PATH}"
sudo chown -R "${USER}" "${PYTHON_VENV_ANSIBLE_PATH}"
${PYTHON_VENV_ANSIBLE}/python -m pip install --upgrade pip setuptools wheel github3.py shyaml
${PYTHON_VENV_ANSIBLE}/python -m pip install ansible-core==2.18
${PYTHON_VENV_ANSIBLE}/ansible-galaxy collection install community.general --force
${PYTHON_VENV_ANSIBLE}/ansible-galaxy collection install ansible.posix
${PYTHON_VENV_ANSIBLE}/ansible --version
```

Añadir el path al entorno global
Edita el archivo `/etc/environment` y añade la siguiente línea para incluir el path del entorno virtual de Ansible que se utilizará por defecto:
```yml
PATH="....:/usr/local/venv-ansible-2.18/bin"
```

## 1.5. Instalar con Ansible el resto del software

```bash
cd ansible-devops-workstation/
${PYTHON_VENV_ANSIBLE}/ansible-galaxy install -r requirements.yml -p roles/
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

Modificar en el archivo ```inventario/host_vars/localhost``` con las siguientes variables:

- ```devops_user_name```: con nuestro propio nombre de usuario.
- ```devops_user_uid```: si nuestro uid es dististo de 1000.
- ```organizacion```: Nombre-empresa.
- En caso que se utilice ```proxy```, de lo contrario comentar las siguientes variables:
  - all_proxy: 'http://IP:3128'.
  - http_proxy: 'http://IP:3128'.
  - https_proxy: 'http://IP:3128'.
  - ftp_proxy: 'http://IP:3128'.
  - no_proxy: ''.
  - soap_use_proxy: ''.
  
Luego ejecutamos Ansible:

```bash
time ${PYTHON_VENV_ANSIBLE}/ansible-playbook -vv -i inventario/hosts site.yml --limit localhost --tags proxy,performance,locales,snap,git,virtualbox,vagrant,docker,microsoft_visualstudio_code,packer
```

Si se da el error: `"msg": "[Errno 2] No existe el archivo o el directorio: b'/usr/local/bin/pip'"` aplicar:
```bash 
sudo ln -s /home/[devops_user_name]/.local/bin/pip /usr/local/bin/pip
```

# 2. Instalación de los aplicativos custom

```bash
cd ..
cd ansible-workstation/
${PYTHON_VENV_ANSIBLE}/ansible-galaxy install -r requirements.yml -p roles/
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

* Crear el directorio para el inventario:

```bash
mkdir -p inventario/{group_vars,host_vars}
```

* Crear el archivo ```inventario/host_vars/office``` con las siguientes variables; `python_venv` y `python_path` dependerán de la versión de Ansible instalada.

- ```devops_user_name```: con nuestro propio nombre de usuario.
- ```devops_shell```: bash
- ```dock_icon_size```: 34
- ```mouse_speed```: 0.25735294117647056
- ```color_scheme```: dark
- ```office_ntp_servers```: [ ntp1, ntp2, ... ]
- ```office_git```: nombre del repositorio git.
- ```firewall_ip: IP_number
     firewall_enabled_at_boot: true
     firewall_enable_ipv6: false
     firewall_allowed_tcp_ports: []
     firewall_additional_rules:
      - "iptables -I INPUT -p tcp --dport 3389 -s {{ firewall_ip_vpn  }} -j ACCEPT -m comment --comment 'Regla de entrada para RDP'"
      ...
      ...
     firewall_deny_tcp_ports: false
- ```### Python virtualenv name ANSIBLE_2_15
     #python_venv: "/usr/local/venv-ansible-2.15"
     #python_path: "/usr/local/venv-ansible-2.15/lib/python3.11/site-packages"
- ```### Python virtualenv name ANSIBLE_2_18
     python_venv: "/usr/local/venv-ansible-2.18"
     python_path: "/usr/local/venv-ansible-2.18/lib/python3.12/site-packages"
     ```

Luego ejecutamos Ansible:

```bash
time ${PYTHON_VENV_ANSIBLE}/ansible-playbook -vv -i inventario/hosts site.yml --extra-vars "@inventario/host_vars/[nombre-host]" --limit localhost
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
