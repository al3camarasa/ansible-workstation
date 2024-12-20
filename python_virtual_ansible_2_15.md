# Instalación de Ansible 2.15 con Python 3.11 usando pyenv

Este documento describe los pasos para instalar y configurar `Ansible 2.15` utilizando `Python 3.11` a través de `pyenv`, sin depender de la versión de Python del sistema.

## 1. Instalar `pip` para Python 3.11.10

```bash
sudo apt-get install -y python3-pip 
```

## 2. Instalación de Python 3.11 y dependencias
Si se desea la versión de `Ansible 2.15`, esta requiere una versión de `Python inferior a 3.12` (The deprecated key_file, cert_file, This was fixed in ansible-core version 2.16.0 or above)

```bash
sudo apt-get install -y python3-venv
```
# Instalación de Python con `pyenv`
Si prefieres no instalar Python a través de apt y utilizar una versión específica, puedes usar `pyenv` para gestionar diferentes versiones de Python. Para ello, instala las dependencias necesarias:

```bash
sudo apt-get install -y python3-venv curl python3-openssl make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev 
```

Luego, instala `pyenv`:

```bash
curl https://pyenv.run | bash
```

Configura tu entorno de shell:
```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Verifica que pyenv se haya instalado correctamente:
```bash
pyenv versions
```

## 3. Crear un entorno virtual
Si necesitas instalar una versión específica de Python, como la 3.11, usa `pyenv` para instalarla:

```bash
pyenv install -s 3.11.10 # install preferred version if needed
```

A continuación, crea un entorno virtual con la versión de Python deseada:
```bash
PYENV_VERSION=3.11.10 python -m venv /usr/local/venv-ansible-2.15 ; source /usr/local/venv-ansible-2.15/bin/activate
sudo chown -R usuario:usuario /usr/local/venv-ansible-2.15/
```

Edita el archivo de configuración del entorno virtual (pyvenv.cfg) para incluir los paquetes del sistema:
```bash
sudo nano  /usr/local/venv-ansible-2.15/pyvenv.cfg 
```
Añade lo siguiente:
```yml
include-system-site-packages = true
version = 3.11.10
```

## 4. Configuración en .bashrc

Agrega las siguientes líneas a tu archivo `~/.bashrc` para configurar las variables de entorno de pyenv:

```bash
# python virtualenv name
PYTHON_VENV_ANSIBLE_2_15=/usr/local/venv-ansible-2.15/bin
```

Luego, cierra y vuelve a abrir la terminal para que se carguen las variables de entorno.


## 5. Instalar componentes de Ansible
Con el entorno virtual activado, instala y actualiza los componentes necesarios:

```bash
"${PYTHON_VENV_ANSIBLE_2_15}"/python -m pip install --upgrade pip setuptools wheel github3.py
"${PYTHON_VENV_ANSIBLE_2_15}"/python -m pip install ansible-core==2.15.12
${PYTHON_VENV_ANSIBLE_2_15}/ansible-galaxy collection install community.general --force
${PYTHON_VENV_ANSIBLE_2_15}/ansible --version
```

## 6. Añadir el path al entorno global
Edita el archivo `/etc/environment` y añade la siguiente línea para incluir el path del entorno virtual de Ansible que se utilizará por defecto:
```yml
PATH="....:/usr/local/venv-ansible-2.15/bin"
```

## 7. Ejecutar los playbooks de Ansible
Para ejecutar un playbook en tu estación de trabajo `ansible_dev_workstation`, configura la variable `ansible_version_deseada` en el archivo de inventario:
```yml
ansible_version_deseada: '8.7.0'
```
Luego, ejecuta el playbook:
```bash
time "${PYTHON_VENV_ANSIBLE_2_15}"/ansible-playbook -i inventario  site.yml --limit localhost --tags proxy,performance,locales,snap,ansible,git,virtualbox,vagrant,docker,microsoft_visualstudio_code,packer
```

En `hosts_vars/office`, configura las rutas del entorno virtual:
```yml
# Python virtualenv name                                                                                      
python_venv: "/home/alejandro/.pyenv/versions/3.11.10/envs/venv-ansible-2.15"
python_path: "/home/alejandro/.pyenv/versions/3.11.10/envs/venv-ansible-2.15/lib/python3.11/site-packages"
```

Ejecuta el playbook con los parámetros de configuración específicos:
```bash
time  ${PYTHON_VENV_ANSIBLE_2_15}/ansible-playbook -vv -i inventario/hosts site.yml --extra-vars "@inventario/host_vars/office" --limit localhost
```

## 8. Volver a la versión de Python del sistema
```bash
pyenv local  system
```

¡Listo! Ahora tienes `Ansible 2.15` corriendo con `Python 3.11` en tu entorno virtual gestionado por `pyenv`.

