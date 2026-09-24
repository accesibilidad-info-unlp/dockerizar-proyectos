# 6. Docker Hub

Hasta ahora trabajamos con imágenes construidas y utilizadas localmente.

Eso es suficiente para probar un proyecto en nuestra computadora, pero el laboratorio necesita poder obtener la imagen desde otro equipo para desplegarla.

Para eso necesitamos un **registro de imágenes**.

```text
Imagen local
     🠟
Registro
     🠟
Otro equipo
```

En este curso utilizaremos **Docker Hub**.

Docker Hub es un registro de imágenes de Docker que permite almacenar y distribuir imágenes. Docker utiliza Docker Hub como registro predeterminado cuando no indicamos otro registro. 

## 1. ¿Qué es Docker Hub?

Hasta este momento, nuestra imagen existe únicamente en nuestra computadora:

```text
┌─────────────────────┐
│    Computadora      │
│                     │
│  mi-proyecto:1.0    │
└─────────────────────┘
```

Si queremos que otra persona pueda utilizarla, necesitamos publicarla:

```text
┌─────────────────────┐
│    Computadora      │
│                     │
│  mi-proyecto:1.0    │
└──────────┬──────────┘
           │
         push
           ↓
┌─────────────────────┐
│     Docker Hub      │
│                     │
│  imagen publicada   │
└──────────┬──────────┘
           │
         pull
           ↓
┌─────────────────────┐
│   Otro equipo       │
│                     │
│  imagen descargada  │
└─────────────────────┘
```

La idea fundamental es:

```text
docker push
     🠟
publicar

docker pull
     🠟
descargar
```

## 2. Nombre de una imagen publicada

Una imagen que queremos publicar en Docker Hub debe identificarse mediante un nombre que incluya nuestro usuario.

Por ejemplo:

```text
user1/demo-app:1.0
```

Podemos dividir este nombre:

```text
user1
  ↓
usuario de Docker Hub

demo-app
  ↓
nombre del repositorio

1.0
  ↓
etiqueta
```

La estructura general puede representarse como:

```text
USUARIO/REPOSITORIO:TAG
```

Por ejemplo:

```text
user1/mi-proyecto:1.0
```

El nombre del repositorio y la etiqueta son importantes porque permiten identificar qué imagen estamos publicando y qué versión queremos utilizar. 

## 3. ¿Qué es un tag?

Una misma imagen puede tener diferentes etiquetas.

Por ejemplo:

```text
mi-proyecto:1.0
mi-proyecto:1.1
mi-proyecto:2.0
```

Las etiquetas nos permiten distinguir versiones de una imagen.

Para un proyecto que va a ser entregado al laboratorio, es conveniente utilizar etiquetas que permitan identificar claramente la versión publicada.

Por ejemplo:

```text
usuario/proyecto:1.0
```

o:

```text
usuario/proyecto:2026-09
```

Lo importante es establecer una convención y utilizarla de manera consistente.

### Sobre `latest`

Docker utiliza `latest` como etiqueta predeterminada cuando no se especifica otra, pero **`latest` no significa automáticamente “la versión más reciente”**.

Es solamente una etiqueta.

Por ejemplo:

```text
usuario/proyecto:latest
```

no se actualiza automáticamente cuando se publica otra imagen.

Además, el uso de `latest` es una convención, no una característica especial que obligue a Docker a actualizar la imagen. 

Para la entrega al laboratorio, recomendamos documentar explícitamente la etiqueta que debe utilizarse.

## 4. Preparar la imagen

Supongamos que tenemos una imagen local:

```text
mi-proyecto:1.0
```

Podemos crear otra referencia para esa misma imagen:

```text
docker tag mi-proyecto:1.0 usuario/mi-proyecto:1.0
```

Ahora podemos tener:

```text
mi-proyecto:1.0
       │
       └──────────────┐
                      ↓
             usuario/mi-proyecto:1.0
```

Las dos referencias apuntan a la misma imagen.

Esto nos permite mantener un nombre local cómodo y, al mismo tiempo, tener el nombre necesario para publicarla en Docker Hub.

También podemos construir directamente utilizando el nombre del repositorio:

```text
docker build -t usuario/mi-proyecto:1.0 .
```

La bibliografía muestra ambas posibilidades: construir la imagen con el nombre del repositorio de Docker Hub o utilizar `docker tag` para asignarle posteriormente ese nombre.  

## 5. Iniciar sesión

Antes de publicar una imagen necesitamos autenticarnos en Docker Hub.

Utilizamos:

```text
docker login
```

Docker solicitará las credenciales correspondientes.

Cuando la autenticación se completa correctamente, podemos publicar imágenes en los repositorios para los que tenemos autorización. 

No debemos colocar contraseñas ni tokens directamente en el repositorio, en el `Dockerfile` o en el archivo `compose.yaml`.

## 6. Publicar la imagen

Una vez autenticados y con la imagen correctamente etiquetada:

```text
docker push usuario/mi-proyecto:1.0
```

El flujo completo es:

```text
Imagen local
     🠟
docker tag
     🠟
usuario/mi-proyecto:1.0
     🠟
docker login
     🠟
docker push
     🠟
Docker Hub
```

Durante el `push`, Docker envía al registro las capas necesarias de la imagen. Al finalizar, Docker muestra información de la imagen publicada, incluido su digest. 

## 7. ¿Qué ocurre con las capas?

Una imagen Docker está formada por capas.

Por eso, al hacer `push`, no necesariamente tenemos que transferir una copia completamente independiente de cada imagen.

Docker puede reutilizar capas que ya existen en el registro.

Podemos pensar en:

```text
Imagen
 ├── Capa base
 ├── Dependencias
 ├── Código
 └── Configuración
```

El registro almacena estas capas y las utiliza para reconstruir la imagen cuando alguien la descarga.

No necesitamos administrar las capas manualmente para nuestro proyecto. Lo importante es comprender que la imagen publicada puede ser recuperada posteriormente como una unidad.

## 8. Descargar la imagen

Una vez publicada, otra computadora puede descargarla mediante:

```text
docker pull usuario/mi-proyecto:1.0
```

Docker obtiene la imagen desde Docker Hub:

```text
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

Si el repositorio es público, cualquier usuario puede descargar la imagen.

Si el repositorio es privado, será necesario contar con las credenciales y permisos correspondientes. 

## 9. Probar la imagen publicada

Esta es una de las pruebas más importantes de todo el taller.

No deberíamos asumir que porque:

```text
docker push
```

terminó correctamente, la imagen está lista para ser utilizada por otra persona.

Tenemos que comprobar el recorrido completo:

```text
Código
   🠟
Build
   🠟
Imagen
   🠟
Push
   🠟
Docker Hub
   🠟
Pull
   🠟
Contenedor
   🠟
Aplicación funcionando
```

Podemos realizar una prueba eliminando la imagen local.

Por ejemplo:

```text
docker image rm usuario/mi-proyecto:1.0
```

Después:

```text
docker pull usuario/mi-proyecto:1.0
```

Y finalmente ejecutarla:

```text
docker run ...
```

De esta forma comprobamos que el proyecto puede ejecutarse utilizando realmente la imagen publicada en Docker Hub.

Este procedimiento aparece explícitamente en la bibliografía como una forma de verificar que la imagen publicada puede descargarse nuevamente desde el registro. 

## 10. Docker Hub no contiene el proyecto fuente

Es importante distinguir:

```text
Repositorio Git
      ↓
Código fuente
      ↓
Dockerfile
```

de:

```text
Docker Hub
      ↓
Imagen Docker
      ↓
Contenedor
```

Docker Hub almacena la **imagen**, no reemplaza nuestro repositorio de código fuente.

Por eso un proyecto puede tener:

```text
GitHub
  ↓
Código fuente
Dockerfile
compose.yaml

Docker Hub
  ↓
Imagen construida
```

Cada elemento cumple una función diferente.

## 11. ¿Qué debe recibir el laboratorio?

Para nuestro objetivo, la entrega debe permitir que el laboratorio sepa exactamente qué imagen utilizar.

Por ejemplo:

```text
Imagen:
usuario/mi-proyecto:1.0

Puerto:
8080

Variables de entorno:
DB_HOST
DB_PORT
DB_NAME

Volúmenes:
db-data

Servicios adicionales:
PostgreSQL
```

El laboratorio debería poder consultar el proyecto y determinar:

```text
¿Qué imagen debo descargar?
          🠟
¿Qué etiqueta debo utilizar?
          🠟
¿Qué puerto debo publicar?
          🠟
¿Qué variables necesito?
          🠟
¿Qué volúmenes necesito?
          🠟
¿Qué servicios adicionales necesito?
```

Esto conecta directamente con el checklist de entrega del proyecto.

## 12. El flujo completo del taller

Llegamos al final del recorrido:

```text
                    Proyecto
                       🠟
                   Dockerfile
                       🠟
                     Imagen
                       🠟
                  Contenedor
                       🠟
                 Prueba local
                       🠟
                    Tag
                       🠟
                  Docker Hub
                       🠟
                     Pull
                       🠟
               Prueba nuevamente
                       🠟
                  Entrega
```

O, si utilizamos Compose:

```text
Proyecto
   🠟
Dockerfile
   🠟
compose.yaml
   🠟
Imágenes
   🠟
Servicios
   🠟
Prueba local
   🠟
Docker Hub
   🠟
Entrega al laboratorio
```

## 13. Flujo recomendado para la entrega

Podemos reducir todo el proceso a una secuencia práctica:

```text
1. Construir la imagen
       🠟
2. Probarla localmente
       🠟
3. Etiquetarla correctamente
       🠟
4. Iniciar sesión en Docker Hub
       🠟
5. Publicarla con docker push
       🠟
6. Descargarla nuevamente con docker pull
       🠟
7. Probar la imagen descargada
       🠟
8. Documentar imagen, tag, puertos y configuración
```

Si el proyecto utiliza Compose, también debemos verificar:

```text
docker compose up -d
       🠟
prueba de la aplicación
       🠟
docker compose down
       🠟
docker compose up -d
       🠟
comprobar nuevamente
```

De esta manera verificamos tanto la imagen como la configuración necesaria para ejecutar el proyecto.

## Lo que necesitamos recordar

- Docker Hub es un registro donde podemos almacenar y distribuir imágenes Docker
- Para publicar una imagen necesitamos identificarla con nuestro usuario y repositorio
- Una referencia de imagen puede tener la forma `USUARIO/REPOSITORIO:TAG`
- Los tags permiten identificar diferentes versiones de una imagen
- `latest` es una etiqueta convencional y no significa automáticamente “última versión”
- `docker tag` permite asignar una nueva referencia a una imagen existente
- `docker login` permite autenticarnos en el registro
- `docker push` publica una imagen
- `docker pull` descarga una imagen
- Una imagen publicada debe probarse nuevamente mediante `pull` y ejecución
- Docker Hub distribuye imágenes, mientras que GitHub u otro repositorio Git almacena el código fuente
- La entrega al laboratorio debe indicar claramente la imagen y el tag que deben utilizarse
- También deben documentarse los puertos, variables de entorno, volúmenes y servicios adicionales necesarios
- La imagen publicada debe ser suficiente para reproducir el entorno de ejecución previsto, junto con la configuración externa que el proyecto necesite