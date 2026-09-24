# Dockerizar proyectos

Taller práctico para aprender a contenerizar y publicar proyectos usando Docker.

El objetivo es que cada equipo pueda llevar su proyecto desde el código fuente hasta una imagen Docker lista para publicar y desplegar.

## Objetivo

Al finalizar el taller, deberías poder:

- crear un `Dockerfile` básico
- construir una imagen Docker
- ejecutar un contenedor localmente
- publicar puertos para probar la aplicación
- utilizar volúmenes cuando el proyecto necesite persistencia
- utilizar Docker Compose para proyectos con varios servicios
- publicar una imagen en Docker Hub
- comprobar que la imagen puede ejecutarse desde Docker Hub

## Recorrido

1. [Conceptos básicos](./01-conceptos/)
2. [Dockerfile](./02-dockerfile/)
3. [Puertos y pruebas locales](./03-puertos/)
4. [Persistencia](./04-persistencia/)
5. [Docker Compose](./05-compose/)
6. [Docker Hub](./06-docker-hub/)

## Requisitos

Antes del taller:

- Docker instalado y funcionando.
- Una cuenta de Docker Hub.
- El proyecto del equipo disponible localmente.

Comprobar Docker:

```bash
docker run hello-world
```

## Flujo de trabajo

```text
Proyecto
   🡇
Dockerfile
   🡇
docker build
   🡇
Imagen
   🡇
docker run
   🡇
Prueba local
   🡇
Docker Hub
   🡇
Despliegue
```