# 02. Dockerfile

El `Dockerfile` es el archivo que describe **cómo construir una imagen Docker** a partir de nuestro proyecto.

Docker lee sus instrucciones en orden y las utiliza para construir la imagen paso a paso.

## Estructura básica

Un `Dockerfile` puede comenzar con una imagen base y luego incorporar los archivos y las dependencias de nuestra aplicación.

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["node", "app.js"]
```

En este ejemplo aparecen algunas de las instrucciones más habituales:

| Instrucción | Función                                                                    |
| --- | --- |
| `FROM`      | Define la imagen base                                                      |
| `WORKDIR`   | Define el directorio de trabajo                                            |
| `COPY`      | Copia archivos desde el contexto de construcción a la imagen               |
| `RUN`       | Ejecuta comandos durante la construcción                                   |
| `CMD`       | Define el comando predeterminado que se ejecutará al iniciar el contenedor |

Estas instrucciones cubren una parte central del proceso de construcción de una aplicación Dockerizada.


## `FROM`

Indica qué imagen se utilizará como punto de partida.

```dockerfile
FROM node:22-alpine
```

En este caso utilizamos una imagen que ya contiene Node.js.

También podríamos utilizar una imagen de NGINX:

```dockerfile
FROM nginx:alpine
```

La elección de la imagen base depende de las necesidades de nuestra aplicación.


## `WORKDIR`

Establece el directorio de trabajo para las instrucciones que aparecen después.

```dockerfile
WORKDIR /app
```

A partir de ese momento, instrucciones como `COPY`, `RUN` y `CMD` pueden trabajar tomando `/app` como directorio de referencia.

Por ejemplo:

```dockerfile
WORKDIR /app

COPY package.json ./
```

El archivo se copiará dentro de `/app`


## `COPY`

Incorpora archivos del proyecto dentro de la imagen.

```dockerfile
COPY package.json ./
```

También podemos copiar el proyecto completo.

```dockerfile
COPY . .
```

Los dos puntos tienen significados diferentes:

- El primer (.) representa la carpeta origen en nuestra máquina (contexto de construcción).

- El segundo (.) representa el directorio destino definido por WORKDIR dentro del contenedor.

El contexto de construcción es importante porque Docker solo puede utilizar los archivos incluidos en él durante la construcción de la imagen .


## `RUN`

Ejecuta un comando **durante la construcción de la imagen**.

Por ejemplo:

```dockerfile
RUN npm install
```

Docker ejecuta ese comando mientras construye la imagen y el resultado queda incorporado en una nueva capa de la imagen.

Es importante distinguirlo de `CMD`:

```text
docker build 🠊 RUN 🠊 Construye la imagen (Build time)
docker run   🠊 CMD 🠊 Inicia el contenedor (Runtime)
```

## `CMD`

Indica qué comando se ejecutará por defecto cuando se inicie un contenedor a partir de la imagen

```dockerfile
CMD ["node", "app.js"]
```

`CMD` no se ejecuta cuando hacemos `docker build`.

Se ejecuta cuando iniciamos un contenedor con `docker run`.

Esta diferencia es fundamental

```text
Dockerfile
    │
    ├── RUN ──🠢 durante docker build
    │
    └── CMD ──🠢 durante docker run
```

## El orden de las instrucciones importa

Consideremos este ejemplo:

```dockerfile
FROM node:22-alpine

WORKDIR /app

# 1. Copiar manifiestos e instalar dependencias
COPY package*.json ./
RUN npm install

# 2. Copiar el resto del código
COPY . .

CMD ["node", "app.js"]
```

Primero copiamos únicamente los archivos que describen las dependencias (`package*.json`) y las instalamos (`RUN npm install`). Después copiamos el resto del código fuente (`COPY . .`).

Esto permite aprovechar la caché de construcción: si solo modificamos el código fuente de la aplicación, Docker reutilizará la capa donde se instalaron las dependencias sin tener que ejecutarlas de nuevo.

> **Regla clave**: Colocar primero aquello que cambia con menor frecuencia evita trabajos innecesarios en re-construcciones


## `.dockerignore`

El contexto de construcción puede contener archivos que no necesitamos dentro de la imagen.

Para evitar enviarlos al proceso de construcción podemos crear `.dockerignore`.

Por ejemplo:

```text
node_modules
.git
.env
npm-debug.log
```

Esto permite excluir archivos innecesarios o sensibles del contexto de construcción.

En particular, debemos evitar incorporar carpetas pesadas y archivos que contengan credenciales o secretos.


## Construir la imagen

Una vez creado el `Dockerfile`, podemos construir nuestra imagen

```bash
docker build -t mi-proyecto:1.0 .
```

El comando puede entenderse así

```text
docker build
     🠟 lee el Dockerfile y utiliza el contexto de construcción
construye la imagen
     🠟
mi-proyecto:1.0
```

La opción `-t` permite asignar un nombre y una etiqueta (tag) a la imagen (`nombre:etiqueta`). En este caso `mi-proyecto` es el nombre y `1.0` la etiqueta

El `.` final indica que el directorio actual será utilizado como contexto de construcción.


## Comprobar la imagen

Después de construirla podemos comprobar que existe localmente con:

```bash
docker images
```
o bien:

```bash
docker image ls
```

Deberíamos encontrar nuestra imagen `nombre:etiqueta`:

| IMAGE | ID | DISK USAGE | CONTENT SIZE | EXTRA |
| --- | --- | ---: | ---: | --- |
| `ubuntu:latest` | `5ba9dab47459` | `78.1MB` | `78.1MB` | |
| `redis:7.0` | `0256c63af7db` | `117MB` | `117MB` | |
| `mi-proyecto:1.0` | `a1b2c3d4e5f6` | `182MB` | `182MB` | `U` |


## Lo que necesitamos recordar

1. `FROM` define la base sobre la cual construimos
2. `WORKDIR` fija la carpeta de trabajo
3. `RUN` se ejecuta al construir (docker build)
4. `CMD` se ejecuta al iniciar el contenedor (docker run)
5. Estructurar el `Dockerfile` optimizando el caché acelera las builds
6. `.dockerignore` excluye archivos pesados o sensibles. 