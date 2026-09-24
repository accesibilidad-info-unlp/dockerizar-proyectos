# 1. Conceptos básicos

## ¿Qué vamos a hacer?

En este taller vamos a preparar un proyecto para que pueda ejecutarse mediante Docker y posteriormente publicarse en Docker Hub.

El objetivo es entender los conceptos necesarios para poder:

- construir una imagen
- ejecutar un contenedor
- probar nuestra aplicación
- conservar datos cuando sea necesario
- publicar la imagen para que otra persona pueda ejecutarla

## ¿Qué es Docker?

Docker es una plataforma que permite empaquetar aplicaciones junto con los elementos que necesitan para ejecutarse y distribuirlas como **imágenes**.

Una imagen puede utilizarse para crear uno o más **contenedores**, que son las instancias en ejecución de esas imágenes.

Podemos pensar el flujo de esta manera:

```text
Código de la aplicación
    🠟
Dockerfile
    🠟 docker build
Imagen
    🠟 docker run
Contenedor
    🠟
Aplicación funcionando
```

## Imagen

Una imagen Docker es un paquete de solo lectura que contiene lo necesario para ejecutar una aplicación:

- código de la aplicación
- dependencias
- componentes mínimos del sistema operativo
- metadatos

Una imagen funciona como una plantilla a partir de la cual podemos crear uno o varios contenedores.

Por ejemplo, podemos tener una imagen `mi-proyecto:1.0` y crear varios contenedores a partir de ella:

```text
            ┌── Contenedor 1
Imagen ─────┼── Contenedor 2
            └── Contenedor 3
```

La imagen representa lo que queremos ejecutar. El contenedor representa una instancia en ejecución.

### Contenedor

Un contenedor es una instancia de una imagen que se está ejecutando.

Una forma sencilla de pensarlo es:

```
Imagen
   🠟
Contenedor
```

La imagen es un artefacto de construcción y el contenedor es un artefacto de ejecución.

Por ejemplo:

`docker run mi-proyecto:1.0`

Docker utiliza la imagen `mi-proyecto:1.0` para crear y ejecutar un contenedor.

Podemos crear varios contenedores a partir de la misma imagen sin tener que construir la imagen nuevamente.

## ¿Docker es una máquina virtual?

No. Una máquina virtual incluye un sistema operativo completo que se ejecuta sobre un hipervisor.

A diferencia de una máquina virtual, los contenedores no virtualizan hardware ni incluyen un sistema operativo completo; comparten el kernel del sistema operativo host y ejecutan procesos aislados que comparten el kernel del sistema que los ejecuta. Esto permite que inicien en segundos y consuman muchos menos recursos.

De forma simplificada:

**Máquina virtual**

```
Aplicación
🠟
Sistema operativo invitado
🠟
Máquina virtual
🠟
Hipervisor
🠟
Sistema operativo host
```

**Contenedores**

```
Aplicación
🠟
Contenedor
🠟
Docker
🠟
Sistema operativo host
```

## Dockerfile

Para crear una imagen de nuestro proyecto necesitamos indicarle a Docker cómo construirla.

Para eso utilizamos un archivo llamado Dockerfile. Este contiene las instrucciones necesarias para construir la imagen.

Por ejemplo:

```
# FROM indica la imagen base
FROM nginx:alpine

# incorpora archivos del proyecto a la imagen
COPY index.html /usr/share/nginx/html/
```

Después podemos construir nuestra propia imagen:

`docker build -t mi-proyecto:1.0 .`

El punto (.) indica el contexto de construcción, es decir, la carpeta local donde están el Dockerfile y los archivos de tu proyecto que Docker utilizará para armar la imagen.

## Imagen, contenedor y Dockerfile

Podemos relacionar los tres conceptos principales de esta manera:

```
Dockerfile
    🠟 docker build
Imagen
    🠟 docker run
Contenedor
```

Cada elemento tiene una función diferente:

| Concepto | Función |
| --- | --- |
| Dockerfile | Describe cómo construir la imagen |
| Imagen | Contiene lo necesario para ejecutar la aplicación |
| Contenedor | Es una instancia ejecutándose a partir de una imagen |

## ¿Dónde obtenemos las imágenes?

No siempre tenemos que construir una imagen desde cero.

Docker puede descargar imágenes desde un registro.

Uno de los registros más utilizados es **Docker Hub**.

Por ejemplo:

`docker pull nginx`

Esto descarga la imagen de NGINX desde un registro para poder utilizarla localmente.

También podemos publicar nuestras propias imágenes:

```text
Dockerfile
   🠟
docker build
   🠟
Imagen
   🠟
Docker Hub
   🠟
docker pull
   🠟
Imagen local
   🠟
docker run
   🠟
Contenedor
```
## Lo que necesitamos recordar

Para este taller, alcanza con recordar estas ideas:

1. Una imagen contiene lo necesario para ejecutar una aplicación.
2. Un contenedor es una instancia de una imagen en ejecución.
3. Un Dockerfile describe cómo construir una imagen.
4. `docker build` construye una imagen.
5. `docker run` crea y ejecuta un contenedor a partir de una imagen.
6. Docker Hub permite almacenar y distribuir imágenes.
