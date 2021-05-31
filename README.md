# proxy-reverso

## Requisitos necesarios:

Para la realización de esta práctica necesitaremos tener previamente instalado las siguientes
herramientas:

**- Docker:** es un conjunto de productos de **plataforma como servicio (PaaS)** que utilizan la
virtualización a nivel de sistema operativo para entregar software en paquetes llamados
**contenedores**.
**- Docker-Compose:** es una herramienta para definir y ejecutar aplicaciones **Docker** de varios
contenedores. Con **Docker-Compose** , usa un archivo **YAML** para configurar los servicios de
su aplicación. Luego, con un solo comando, crea e inicia todos los servicios desde su
configuración.

Para instalar las herramientas solo tendremos que utilizar la siguiente sentencia:

```
“sudo apt install *Nombre de la herramienta*”
```
## Configuración contenedores:

Lo primero que haremos será crear un archivo **.yml** donde le daremos forma a lo que será el
**servidor** y las **dos páginas web** :
```
version: '3.5'

services:
  nginx-proxy:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - /etc/nginx/vhost.d
      - /usr/share/nginx/html
  
  sitio1.com:
    image: nginx
    restart: always
    expose:
      - "80"
    environment:
      - VIRTUAL_HOST=sitio1.com

  sitio2.com:
    image: nginx
    restart: always
    expose:
      - "80"
    environment:
      - VIRTUAL_HOST=sitio2.com
```

Ahora ejecutaremos el comando **“sudo docker-compose up - d”** para levantar los contenedores.


### Comprobación:

Ahora utilizaremos el siguiente comando para ver que se ha levantado los contenedores:

```
“sudo docker ps - a”
```

## Configuración páginas web:

Nos dirigiremos dentro del contendor de cada una de las páginas web mediante el siguiente
comando:

```
“sudo docker exec – it *ID contenedor* bin/bash”
```
Una vez dentro tendremos que actualizar los paquetes de herramientas (en cada contenedor en el
que nos metemos), para ello utilizaremos los siguientes comandos:

```
“apt update”
```
```
“apt upgrade”
```
También instalaremos un **editor de texto** para poder editar los archivos, en mí caso utilizaré nano
por lo que ejecutaré el siguiente comando:

```
“apt install nano”
```
Una vez actualizados los paquetes lo que haremos será **borrar** el archivo **index.html** mediante el
comando **“rm –rf *directorio donde se ubica el archivo*”:**

Y ahora crearemos uno nuevo en el que agregaremos unas pocas líneas para hacer una **página web
personalizada** :
```
<html>
<title>Sitio1</title>
<body>
Sitio1
</body>
</html>
```

```
*Esto lo haremos también con el contenedor de la otra página web*
```

## Configuración servidor proxy:

### IP:

Lo primero que haremos será saber **las IP** de los contenedores que contienen nuestras **páginas web** ,
para ello lo que haremos será utilizar el siguiente comando:

```
“sudo docker inspect *ID contenedor* | grep “IPAddress””
```
### Configuración:

Una vez conozcamos **las IP** , nos adentraremos en el contenedor del servidor y crearemos un nuevo
archivo el cual llamaremos **“load-balancing.conf”** , en el añadiremos las siguientes líneas:
```
upstream backend {
        server 192.168.10.11;
        server 192.168.10.12;
    }
	
    server {
        listen      80;
        server_name loadbalancing.example.com;

        location / {
	        proxy_redirect      off;
	        proxy_set_header    X-Real-IP $remote_addr;
	        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header    Host $http_host;
		proxy_pass http://backend;
	}
}
```
En la parte de “ **server *IP*** ” cambiaremos la **IP** que tenga nuestro **contenedor** de la página web.


Lo siguiente que haremos será borrar el archivo **“default.conf”** para que utilice el archivo que hemos
creado antes, y por último reiniciaremos el servidor, para ello utilizaremos el siguiente comando:

```
“/etc/init.d/nginx restart”
```
### Comprobar funcionamiento del contenedor:

Para comprobar que **todas las modificaciones son correctas** lo que haremos será utilizar la siguiente
sentencia:

```
“nginx -t”
```
## Comprobación final:

Ahora lo que haremos será dirigirnos al navegador y escribir la **IP** de nuestro servidor y **recargar** la
página para comprobar que cambia a la otra página web:

## Importante:

```
- A la hora de agregar los archivos .yml hay que tener especial cuidado a la hora de escribirlo
ya que si escribimos un espacio de más o de menos pues hacer que el archivo no se ejecute
correctamente, también cuidado a la hora de escribir una palabra mal.
- Cuidado con las rutas de los directorios, un fallo a la hora de escribir donde se sitúa nuestro
archivo puede darnos un error el cual luego no pensemos que pueda ser ese el error que nos
esté generando el fallo.
```
