# Aplicación shield para el despliegue y automatización de tareas con Fabric, Ansible y Docker

A continuación vamos explicar como instalar el proyecto.

1. Descargar el repositorio con el código con el siguiente comando:

```
git clone https://github.com/raulgarcia07/shield.git
```

2. Entramos en la carpeta `shield` y creamos el entorno virtual con el comando:
```
python3 -m venv .venv 
```
3. Activar el entorno virtual con el comando:
```
source .venv/bin/activate for Linux

\path\to\env\Scripts\activate for Windows
```
4. Activar las librerias necesarias para este proyecto que se encuentra en el fichero `requirements.txt` con el comando:
```
pip install -r requirements.txt
```

5. Ejecutamos las migraciones iniciales:

```
python manage.py migrate
```

6. Cargamos los datos iniciales de los metahumans con:

```
python manage.py load_from_csv superheroes.csv
```

7. Configuramos el archivo `shield/settings.py` , añadiendo la dirreción de loopback `127.0.0.1 ` (en la linea 28) de forma que en la linea aparezca al menos esa dirreción ip:

```
ALLOWED_HOSTS = ['192.168.33.10', '127.0.0.1', 'localhost']
```
8. Creamos nuesto usuario superuser con:
```
python manage.py createsuperuser 
```

9. Arrancar el servidor para comprobar que se ha instaldo correctamente la aplicación:
```
python manage.py runserver
```

## Despliegue de la aplicación en varias máquinas con Fabric:

1. Instalar fabric en el entorno virtual con el comando:
```
pip install fabric
```

2. Cambiar los datos de acceso a la máquina remota, localizados en la función ´development(ctx)´ del archivo fabfile.py ,líneas 12-16:

```
@task
def development(ctx):
    ctx.user = 'vagrant'
    ctx.host = '192.168.33.10'
    ctx.connect_kwargs = {"password": "vagrant"}
 ``` 

3. Comentar las lineas 92 y 93 del archivo `fabfile.py` si no se quiere hacer un `sudo apt-get update` y un `sudo apt-get install python3-venv` porque ya está instalado.

4. Ejecutar el script con el comando con la estructura `fab nombre-de-la-función deploy`, en este caso:

 ```
 fab development deploy
```
(OPCIONAL)

5. Se pueden añadir varios entornos de configuración creando funciones del tipo del paso 2:
```
@task
def production(ctx):
    ctx.user = 'production'
    ctx.host = '192.168.33.10'
    ctx.connect_kwargs = {"password": "production"}
 ``` 

 6. Ejecutar el script con el comando con la estructura `fab nombre-de-la-función deploy`, en este caso:
    ```
    fab production deploy
    ```

## Despliegue de la aplicación en varias máquinas con Ansible:

1. Instalar Ansible en el sistema operativo:
```
En la terminal de Ubuntu:

sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

2. Instalar la libreia de Ansible en el entorno virutal:

```
pip install ansible
```
3. En el fichero ansible/vars.yml se encuentran las variables del despliegue, cambiar al menos la variable `ssh_private_key` por la ruta donde está la clave privada.

4. En el fichero `ansible/host` añadir o quitar las máquinas donde se quiere desplegar junto con `ansible_python_interpreter=/usr/bin/python3`:
```
192.168.33.10 ansible_python_interpreter=/usr/bin/python3
```

5. Dirigirnos a la carpeta `ansible` si  no lo estamos ya:
```
cd ansible
```
6. Ejecutar el script dentro de la carpeta `ansible` con:
```
ansible-playbook -i hosts provision.yml --user=vagrant --ask-pass # donde vagrant es el usuario de la máquina remota
```

7. Instalar sshpass si lo requiere y volver a ejecutar el paso 6:
```
sudo apt-get install sshpass
```

## Despliegue de la aplicación en varias máquinas con Docker:

1. Comprobar que nuestro proyecto funciona correctamente.

2. Instalar Docker en nuestra máquina local y en la remota:
```
En Windows: https://docs.docker.com/docker-for-windows/install/
En Mac: https://docs.docker.com/docker-for-mac/install/
En Ubuntu: https://docs.docker.com/engine/install/ubuntu/
```
3. Dirigirnos a la carpeta raiz de nuestro proyecto y  crear una imagen de docker con:
```
docker build -t shield .
```
  
4. Comprobar que funciona la imagen de docker con :
```
docker run --publish 8000:8000 shield 
o
docker run -d -p 8000:8000 shield # para ejecutar en segundo plano

y curl localhost:8000  # si nos devuelve código html funciona
```
5. Si funciona correctamente guardar un archivo comprimido .tar con la imagen:

```
docker save -o ./shield.tar shield   # donde ./shield.tar es la ruta donde queremos guardar el fichero en nuestra máquina local

```
6. Transferir el archivo por ssh con el comando:
```
scp ./shield.tar usuario@maquina:/home/usuario # sustituir usuario@maquina  y /home/usuario por la ruta donde queremos que se guarde el fichero en la máquina remota

```
7. Una vez transferido el archivo ejecutar el siguiente comando en la máquina remota para cargar la imagen:
```
docker load -i "path to image tar file"

```
8. Repetir el paso 4  en la máquina remota para comprobar que funciona la imagen de docker.