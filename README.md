# B

A continuación vamos explicar como instalar el proyecto.

1. Descargar el repositorio con el código con el siguiente comando:

```
git clone https://github.com/raulgarcia07/shield.git
```

2. Entramos en la carpeta `shield` y crear el entorno virtual con el comando:
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

1. Asegurarse de que en la máquina donde se vaya a instalar el proyecto este instalado `python3` y  `python3-venv`:


2. Instalar fabric en el entorno virtual con el comando:
```
pip install fabric
```

3. Cambiar los datos de acceso a la máquina remota, localizados en la función ´development(ctx)´ del archivo fabfile.py ,líneas 12-16:

```
@task
def development(ctx):
    ctx.user = 'vagrant'
    ctx.host = '192.168.33.10'
    ctx.connect_kwargs = {"password": "vagrant"}
 ``` 

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