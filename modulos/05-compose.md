# 5. Docker Compose

Hasta ahora trabajamos con contenedores individualmente.

Esto funciona bien cuando tenemos una aplicación sencilla, pero muchos proyectos necesitan varios servicios para funcionar.

Por ejemplo:

```text
┌─────────────────┐
│   Aplicación    │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│    PostgreSQL   │
└─────────────────┘
```

También podríamos tener:

```text
┌─────────────────┐
│   Frontend      │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│     Backend     │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│    PostgreSQL   │
└─────────────────┘
```

Cada componente puede ejecutarse en su propio contenedor.

El problema es que tendríamos que ejecutar, configurar, conectar, detener y eliminar varios contenedores manualmente.

**Docker Compose permite describir estos servicios en un único archivo y administrarlos como una aplicación.** 

## 1. ¿Qué es Docker Compose?

Docker Compose es una herramienta integrada en Docker que permite definir aplicaciones formadas por varios contenedores mediante un archivo YAML.

El archivo describe los elementos necesarios para ejecutar la aplicación, como:

- servicios (los contenedores a ejecutar)
- imágenes o contextos de construcción
- puertos publicados
- variables de entorno
- volúmenes 
- redes internas de comunicación

Luego podemos utilizar comandos como:

```text
docker compose up
```

para poner en funcionamiento toda la aplicación.

Compose se integra actualmente en el comando principal `docker`, por lo que utilizamos:

```text
docker compose
```

y no:

```text
docker-compose
```

Las versiones modernas de Docker incluyen Compose como parte del conjunto de herramientas de Docker. 

## 2. El archivo `compose.yaml`

La configuración de Compose se guarda normalmente en:

```text
compose.yaml
```

Podemos imaginar este archivo como una descripción de nuestra aplicación:

```text
compose.yaml
     │
     ├── services (definición de contenedores)
     ├── ports (mapeo de puertos)
     ├── environment (variables de entorno)
     ├── volumes (volúmenes compartidos/persistentes)
     └── networks (redes de comunicación)
```

Por ejemplo, una aplicación que utiliza un backend y PostgreSQL podría en `compose.yaml` tener:

```text
services:
  app:
    # configuración del backend
  
  database:
    # configuración de la base de datos
```

Cada entrada dentro de `services` representa un servicio de la aplicación.

En términos prácticos, cada servicio normalmente terminará ejecutándose como un contenedor.

## 3. Un primer ejemplo

Supongamos que tenemos una aplicación cuya imagen ya construimos en el módulo anterior:

```text
mi-proyecto:1.0
```

Podemos crear un `compose.yaml` sencillo:

```text
services:
  app:
    image: mi-proyecto:1.0
    ports:
      - "8080:80"
```

Aquí estamos diciendo:

```text
services
   🠟
app
   🠟
image: mi-proyecto:1.0
   🠟
ports: 8080:80
```

El servicio `app` utilizará la imagen `mi-proyecto:1.0` y publicará el puerto `80` del contenedor en el puerto `8080` del host.

La sección `ports` cumple la misma función conceptual que `-p` en `docker run`. 

## 4. Levantar la aplicación

Desde el directorio donde se encuentra `compose.yaml` ejecutamos:

```text
docker compose up
```

Compose lee el archivo y crea los recursos necesarios para ejecutar la aplicación.

Podemos visualizar el flujo así:

```text
compose.yaml
     🠟
Docker Compose
     🠟
Imagen
     🠟
Contenedor
     🠟
Aplicación activa
```

Si queremos dejar los contenedores ejecutándose en segundo plano:

```text
docker compose up -d
```

La opción `-d` (detach) permite ejecutar Compose en segundo plano. 

## 5. Ver el estado de los servicios y logs

Podemos consultar los contenedores administrados por Compose mediante:

```text
docker compose ps
```

Esto permite comprobar qué servicios están funcionando y consultar información como su estado y los puertos publicados. 

También podemos consultar los registros:

```text
docker compose logs
```

Esto resulta especialmente útil cuando una aplicación no inicia correctamente.

El flujo básico de comprobación es:

```text
docker compose up -d
       🠟
docker compose ps
       🠟
docker compose logs
       🠟
probar la aplicación
```

## 6. Una aplicación con dos servicios

Ahora podemos aprovechar lo aprendido en el módulo anterior.

Supongamos que nuestro proyecto necesita una aplicación web y PostgreSQL:

```text
┌─────────────────┐
│       app       │
│                 │
│  Aplicación web │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│    database     │
│                 │
│   PostgreSQL    │
└─────────────────┘
```

El `compose.yaml` puede describir ambos servicios:

```text
services:
  app:
    image: mi-proyecto:1.0
    ports:
      - "8080:80"

  database:
    image: postgres:17-alpine
    environment:
      POSTGRES_PASSWORD: "contraseña_segura"
```

Cuando ejecutamos:

```text
docker compose up -d
```

Compose inicia automáticamente ambos contenedores coordinados sin necesidad de escribir múltiples comandos individuales.

Esto es justamente una de las ventajas principales de Compose: podemos describir la aplicación completa en lugar de mantener una serie de comandos independientes. 

## 7. Servicios y comunicación

Los servicios definidos en un mismo proyecto Compose pueden comunicarse entre sí.

Por ejemplo:

```text
        ┌──────────────┐
        │     app      │
        └──────┬───────┘
               🠟 (red interna)
        ┌──────────────┐
        │   database   │
        └──────────────┘
```

La aplicación puede conectarse al servicio de base de datos utilizando el nombre del servicio:

```text
database
```

Por ejemplo, conceptualmente:

```text
DB_HOST=database
```

No necesitamos averiguar la dirección IP del contenedor.

Compose crea la infraestructura de red necesaria para que los servicios puedan comunicarse. En el ejemplo de referencia de Poulton, los servicios de la aplicación están conectados a una red común creada por Compose. 

## 8. Volúmenes en Compose

También podemos utilizar los volúmenes que vimos en el módulo anterior.

Por ejemplo:

```text
services:
  database:
    image: postgres:17-alpine
    environment:
      POSTGRES_PASSWORD: "contraseña_segura"
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

Aquí tenemos dos partes:

```text
services
   🠟
database
   🠟
volumes
   🠟
db-data:/var/lib/postgresql/data
```

y:

```text
volumes
   🠟
db-data
```

La primera parte indica dónde se monta el volumen dentro del contenedor.

La segunda declara el volumen que Compose administrará.

De esta manera podemos combinar los conceptos de los módulos 4 y 5:

```text
Compose
   🠟
Servicio PostgreSQL
   🠟
Volumen
   🠟
Datos persistentes
```

Compose puede crear y administrar estos volúmenes junto con los demás recursos de la aplicación. 

## 9. Detener la aplicación

Para detener los servicios podemos utilizar:

```text
docker compose stop
```

Los contenedores se detienen pero no se eliminan.

Podemos volver a iniciarlos posteriormente.

También podemos utilizar:

```text
docker compose restart
```

para reiniciar los servicios.

Estos comandos forman parte del ciclo habitual de trabajo con Compose. 

## 10. Eliminar la aplicación

Cuando queremos detener y eliminar los recursos creados por Compose utilizamos:

```text
docker compose down
```

Por defecto, `docker compose down` elimina los contenedores y las redes creadas para la aplicación, pero no elimina los volúmenes ni las imágenes. 

Esto es importante cuando tenemos datos persistentes.

Podemos pensar en:

```text
docker compose down
        🠟
contenedores eliminados
        +
red creada por Compose eliminada
        +
volúmenes conservados
```

Si posteriormente ejecutamos:

```text
docker compose up -d
```

Compose puede volver a crear los contenedores utilizando los mismos volúmenes.

## 11. `down` no significa borrar todo

Es importante distinguir:

```text
docker compose stop
```

Detiene los contenedores.

```text
docker compose down
```

Detiene y elimina los contenedores y las redes del proyecto.

Y una opción como:

```text
docker compose down --volumes
```

también elimina los volúmenes administrados por Compose.

Esta última opción debe utilizarse con cuidado cuando los volúmenes contienen datos que queremos conservar. 

## 12. `image` o `build`

Hasta ahora utilizamos:

```text
image: mi-proyecto:1.0
```

Esto significa que Compose utilizará una imagen existente.

Pero también podemos indicar que la imagen debe construirse utilizando un `Dockerfile`:

```text
services:
  app:
    build: .
    ports:
      - "8080:80"
```

En este caso:

```text
build: .
   🠟
Dockerfile
   🠟
imagen
   🠟
contenedor
```

La ruta `.` indica que el contexto de construcción se encuentra en el directorio actual.

Compose puede utilizar `build` o `image` según cómo esté definido el servicio. 

Para nuestros proyectos, esto resulta especialmente útil porque podemos mantener juntos:

```text
Proyecto
├── Dockerfile
├── compose.yaml
├── código fuente
└── configuración
```

El `Dockerfile` define cómo construir la imagen.

El `compose.yaml` define cómo ejecutar los servicios que forman la aplicación.

## 13. El flujo completo

Podemos reunir lo aprendido en los módulos anteriores:

```text
Código
   🠟
Dockerfile
   🠟
Imagen
   🠟
compose.yaml
   🠟
Contenedores
   🠟
Aplicación funcionando
```

Cuando existen varios servicios:

```text
                 compose.yaml
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
       Servicio                Servicio
          ↓                       ↓
     Contenedor              Contenedor
          │                       │
          └───────────┬───────────┘
                      ↓
                 Aplicación
```

Y si existen datos persistentes:

```text
                  compose.yaml
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
          Servicio            Servicio
             ↓                   ↓
        Contenedor           Contenedor
                                 │
                                 ↓
                              Volumen
                                 │
                                 ↓
                                Datos
```

## 14. Flujo de trabajo recomendado

Para trabajar con un proyecto que utiliza Compose:

```text
1. Revisar compose.yaml
       🠟
2. docker compose up -d
       🠟
3. docker compose ps
       🠟
4. Probar la aplicación
       🠟
5. docker compose logs
       🠟
6. Realizar cambios
       🠟
7. Reconstruir si corresponde
       🠟
8. Volver a probar
```

Si modificamos el `Dockerfile` o archivos que forman parte de la imagen, será necesario reconstruirla.

Podemos hacerlo con:

```text
docker compose build
```

y después:

```text
docker compose up -d
```

Compose también puede encargarse de construir las imágenes cuando corresponde al utilizar `up`, pero conocer `build` explícitamente resulta útil cuando estamos desarrollando y queremos controlar cuándo se reconstruye una imagen. 

## 15. ¿Cuándo necesitamos Compose?

No todos los proyectos necesitan Compose.

Para una aplicación muy sencilla puede ser suficiente:

```text
Dockerfile
   🠟
Imagen
   🠟
Contenedor
```

Compose resulta especialmente útil cuando el proyecto necesita varios componentes:

```text
Frontend
   +
Backend
   +
Base de datos
   +
Otros servicios
```

En esos casos, `compose.yaml` permite documentar y reproducir la forma en que deben ejecutarse juntos.

Además, el archivo Compose debería formar parte del repositorio del proyecto. Esto permite que la configuración pueda versionarse junto con el código. 

## Lo que necesitamos recordar

- Docker Compose permite definir y administrar aplicaciones formadas por varios contenedores
- La configuración se guarda normalmente en `compose.yaml`
- Cada entrada de `services` representa un servicio de la aplicación
- `docker compose up` crea y ejecuta la aplicación definida en Compose
- `docker compose up -d` ejecuta la aplicación en segundo plano
- `docker compose ps` permite consultar el estado de los servicios
- `docker compose logs` permite consultar los registros
- `docker compose stop` detiene los servicios sin eliminarlos
- `docker compose restart` reinicia los servicios
- `docker compose down` elimina los contenedores y redes del proyecto
- `docker compose down --volumes` también elimina los volúmenes, por lo que debe utilizarse con cuidado
- `image` permite utilizar una imagen existente
- `build` permite construir una imagen a partir de un `Dockerfile`
- Compose permite declarar puertos, variables de entorno, volúmenes y otros recursos
- Los servicios de una misma aplicación pueden comunicarse entre sí
- `compose.yaml` debe formar parte del repositorio cuando describe cómo ejecutar el proyecto