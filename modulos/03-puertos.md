# 3. Puertos y pruebas locales

Una aplicación que se ejecuta dentro de un contenedor se encuentra aislada en su propia red.

Para acceder a una aplicación web desde nuestro navegador debemos **publicar un puerto del contenedor en un puerto de nuestra computadora (host)**.

## El mapeo de puertos

La opción `-p` (publish) de `docker run` permite realizar este mapeo:

```bash
docker run -p 8080:80 mi-proyecto:1.0
```

La estructura es:

```text
-p PUERTO_HOST:PUERTO_CONTENEDOR
```

En nuestro ejemplo:

```text
Computadora                         Contenedor
┌─────────────────┐                ┌─────────────────┐
│                 │                │                 │
│  localhost:8080 ├───────────────>│      :80        │
│                 │                │                 │
└─────────────────┘                └─────────────────┘
       8080                              80
```

El puerto `8080` pertenece a nuestra computadora y el puerto `80` pertenece al contenedor.

Por lo tanto, cuando abrimos:

```text
http://localhost:8080
```

Docker redirige la conexión hacia el puerto `80` del contenedor.


## ¿Por qué los números pueden ser diferentes?

El puerto utilizado por la aplicación dentro del contenedor no tiene por qué ser igual al puerto que utilizamos en nuestra computadora.

Por ejemplo:

```bash
docker run -p 8080:80 mi-proyecto:1.0
```

significa:

```text
8080 🠢 80
host    contenedor
```

También podríamos utilizar:

```bash
docker run -p 3000:80 mi-proyecto:1.0
```

En este caso:

```text
3000 🠢 80
host    contenedor
```

La aplicación sigue escuchando en el puerto `80` dentro del contenedor. Lo único que cambia es el puerto mediante el cual accedemos desde nuestra computadora:

```text
http://localhost:3000
```


## Probar un contenedor localmente

Una vez construida nuestra imagen, podemos iniciar un contenedor:

```bash
docker run -d -p 8080:80 --name prueba-local mi-proyecto:1.0
```

En este comando:

* `-d` (detach) ejecuta el contenedor en segundo plano
* `-p 8080:80` publica el puerto `80` del contenedor en el puerto `8080` de nuestra computadora
* `--name prueba-local` asigna un nombre al contenedor
* `mi-proyecto:1.0` es la imagen a partir de la cual se crea el contenedor

Después podemos abrir:

```text
http://localhost:8080
```

Podemos verificar que el contenedor se encuentra en ejecución mediante:

```bash
docker ps
```

La salida incluye una columna `PORTS` donde podemos comprobar el mapeo realizado por Docker.

Por ejemplo:

```text
CONTAINER ID   IMAGE              COMMAND       STATUS        PORTS                  NAMES
a1b2c3d4e5f6   mi-proyecto:1.0    ...           Up 2 minutes  0.0.0.0:8080->80/tcp  prueba-local
```

La expresión:

```text
0.0.0.0:8080->80/tcp
```

indica que el puerto `8080` del host está publicado hacia el puerto `80` del contenedor.


## `EXPOSE` no publica el puerto

Podemos encontrar una instrucción como esta en un `Dockerfile`:

```dockerfile
EXPOSE 80
```

`EXPOSE` documenta que la aplicación utiliza ese puerto dentro del contenedor.

No publica el puerto automáticamente en el host. Para acceder desde nuestra computadora seguimos necesitando la opción `-p`:

```bash
docker run -p 8080:80 mi-proyecto:1.0
```

Por lo tanto:

```text
EXPOSE 80 🠊 documenta el puerto del contenedor

-p 8080:80 🠊 publica el puerto para acceder desde el host
```

Esta distinción es importante porque `EXPOSE` por sí solo no hace que la aplicación sea accesible desde `localhost`.


## Si la aplicación no funciona

Cuando un contenedor inicia pero no podemos acceder a la aplicación, podemos comprobar algunas cosas básicas:

### 1. Comprobar que el contenedor está ejecutándose

```bash
docker ps
```

Si no aparece en la lista, puede haber fallado al iniciar. Para ver contenedores detenidos usar:

```bash
docker ps -a
```

### 2. Comprobar el mapeo de puertos

En la columna `PORTS` de `docker ps`, corroborar que figure la redirección esperada. Buscamos algo similar a:

```text
0.0.0.0:8080->80/tcp
```

### 3. Consultar los logs

```bash
docker logs prueba-local
```

Los logs muestran la salida estándar y de error de la aplicación, permitiendo detectar fallos de inicio o dependencias faltantes.


## Probar diferentes puertos

Podemos cambiar el puerto del host sin modificar la aplicación ni reconstruir la imagen:

```bash
docker run -d -p 8081:80 --name prueba-local-2 mi-proyecto:1.0
```

Ahora accederíamos mediante:

```text
http://localhost:8081
```

Mientras que dentro del contenedor la aplicación continúa utilizando el puerto `80`.

Esto nos permite tener varios contenedores que utilizan el mismo puerto interno, siempre que publiquemos cada uno mediante un puerto diferente del host.

```text
Host                         Contenedor

localhost:8080 🠢             contenedor1 (:80)
localhost:8081 🠢             contenedor2 (:80)
localhost:8082 🠢             contenedor3 (:80)
```


## El flujo completo

El proceso acumulado hasta este punto es:

```text
Código
  🠟
Dockerfile
  🠟
docker build
  🠟
Imagen
  🠟
docker run -p
  🠟
Contenedor
  🠟
http://localhost:8080
  🠟
Prueba local exitosa
```

La prueba local es un paso importante antes de publicar la imagen.

Primero comprobamos que la imagen funciona en nuestra máquina y posteriormente podremos comprobar que la misma imagen funciona después de publicarla en Docker Hub.

## Lo que necesitamos recordar

1. Una aplicación dentro de un contenedor puede escuchar en un puerto propio del contenedor
2. `-p HOST:CONTENEDOR` mapea el puerto para hacerlo accesible desde nuestra computadora
3. `http://localhost:8080` permite acceder a la aplicación mapeada en el puerto `8080` del host
4. `EXPOSE` documenta un puerto del contenedor pero no lo publica
5. `docker ps` permite comprobar los contenedores en ejecución y sus puertos publicados
6. `docker logs` permite consultar los mensajes generados por un contenedor
7. Antes de publicar una imagen debemos comprobar que la aplicación funciona localmente
