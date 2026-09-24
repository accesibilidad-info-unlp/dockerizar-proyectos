# 4. Persistencia de datos

Hasta ahora trabajamos con contenedores que ejecutan una aplicación. Pero ¿qué ocurre con los datos que genera esa aplicación?

Por ejemplo:

- una base de datos guarda registros
- una aplicación genera archivos o archivos de registro (logs)
- un usuario carga datos o documentos
- un sistema almacena configuraciones persistentes

Si esos datos quedan únicamente dentro del sistema de archivos del contenedor, quedan ligados al ciclo de vida de ese contenedor y se perderán al eliminarlo.

Docker proporciona **volúmenes** para almacenar datos fuera del contenedor y permitir que esa información sobreviva al reemplazo del mismo.


## 1. El problema: contenedor y datos no son lo mismo

Un contenedor tiene su propio sistema de archivos.

Podemos imaginarlo así:

```text
┌─────────────────────────┐
│       Contenedor        │
│                         │
│  Aplicación             │
│  Código                 │
│  Dependencias           │
│                         │
│  Datos                  │
│      🠟                  │
│  /app/data              │
└─────────────────────────┘
```

Si la aplicación escribe datos dentro del contenedor, esos datos forman parte del estado temporal de ese contenedor.

Esto resulta problemático cuando necesitamos actualizar o reemplazar el contenedor.

Por ejemplo:

```text
Contenedor
    🠟
se elimina
    🠟
se crea otro contenedor
    🠟
los datos que estaban dentro del anterior ya no están disponibles
```

Por eso conviene separar los componentes:

```text
Aplicación 🠊 Contenedor (Efímero)

Datos 🠊 Volumen (Persistente)
```

El contenedor puede desaparecer y volver a crearse mientras los datos permanecen en el volumen.

Los volúmenes están desacoplados de los contenedores y pueden utilizarse posteriormente con otros contenedores. 

## 2. ¿Qué es un volumen?

Un **volumen** es un espacio de almacenamiento administrado por Docker que existe de forma independiente y puede montarse dentro de uno o varios contenedores.

Podemos representarlo así:

```text
┌──────────────────┐
│    Contenedor    │
│                  │
│  /app/data ──────┼──────┐
└──────────────────┘      │
                          │
                    ┌────────────┐
                    │  Volumen   │
                    │            │
                    │   Datos    │
                    └────────────┘
```

La aplicación continúa trabajando con una ruta dentro del contenedor, pero esa ruta está conectada con un volumen.

Esto permite que los datos permanezcan aunque el contenedor sea eliminado o reemplazado.

## 3. Crear y gestionar un volumen

Podemos crear un volumen explícitamente:

```text
docker volume create datos
```

Para comprobar que existe:

```text
docker volume ls
```

Docker también permite inspeccionarlo:

```text
docker volume inspect datos
```

Docker administra este almacenamiento en una ruta interna del host. Lo importante es entender que **Docker gestiona la ubicación y el ciclo de vida de este almacenamiento por nosotros**.

## 4. Utilizar un volumen con un contenedor

Un volumen debe montarse en una ruta determinada dentro del contenedor.

Por ejemplo:

```text
docker run -d \
  --name mi-app \
  --mount source=datos,target=/app/data \
  mi-proyecto:1.0
```

Aquí aparecen tres elementos importantes:

```text
source=datos 🠊 nombre del volumen

target=/app/data 🠊 ruta dentro del contenedor
```

Es decir:

```text
Volumen "datos"
    🠟
/app/data
    🠟
Aplicación
```

Docker permite montar un volumen mediante opciones como `--mount`. También existe la forma abreviada mediante `-v`. 

## 5. ¿Qué ocurre si eliminamos el contenedor?

Este es el punto más importante del módulo.

Supongamos que tenemos:

```text
Contenedor A
     🠟
Volumen "datos"
```

La aplicación guarda información en `/app/data`. Si eliminamos el contenedor:

```text
Contenedor A 🠊 eliminado

Volumen "datos" 🠊 intacto
```

Podemos crear posteriormente otro contenedor y conectarlo al mismo volumen:

```text
Volumen "datos"
       🠟
Contenedor B
```

La aplicación en el Contenedor B accederá inmediatamente a los datos almacenados por el Contenedor A.

Los volúmenes están desacoplados del ciclo de vida de los contenedores. Por eso eliminar un contenedor no implica necesariamente eliminar el volumen que utiliza. 

## 6. Un ejemplo con una base de datos

Las bases de datos son uno de los casos más claros para utilizar persistencia.

Supongamos una aplicación que utiliza PostgreSQL:

```text
┌──────────────────┐
│    Aplicación    │
│                  │
│       API        │
└────────┬─────────┘
         🠟
┌──────────────────┐
│   PostgreSQL     │
│                  │
│   /var/lib/      │
│   postgresql/    │
│      data        │
└────────┬─────────┘
         🠟
   Volumen "db-data"
```

El directorio donde PostgreSQL almacena sus datos puede estar asociado con un volumen.

Por ejemplo:

Creamos el volumen para la base de datos:

```text
docker volume create db-data
```

Luego podemos utilizarlo con el contenedor de PostgreSQL:

```text
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=contraseña \
  --mount source=db-data,target=/var/lib/postgresql/data \
  postgres
```

**Nota de seguridad:** Nunca debemos colocar contraseñas reales hardcodeadas directamente en comandos públicos o repositorios. Este ejemplo utiliza una variable de entorno únicamente con fines demostrativos.

La misma idea aparecerá nuevamente cuando trabajemos con Docker Compose, donde podremos declarar los volúmenes junto con los servicios de la aplicación.

## 7. Volúmenes y `docker compose`

Cuando una aplicación está formada por varios servicios, Compose permite declarar los volúmenes necesarios junto con ellos.

Por ejemplo, conceptualmente:

```text
compose.yaml

Aplicación
     │
     └🠊 PostgreSQL
            🠟
         db-data
```

Esto permite que la configuración del almacenamiento forme parte de la definición del proyecto.

Docker Compose puede administrar volúmenes junto con los servicios, redes y demás recursos necesarios para una aplicación. 

Lo veremos con mayor detalle en el **Módulo 5 — Docker Compose**.

## 8. ¿Volumen o carpeta del proyecto?

Docker también permite utilizar un directorio específico del equipo mediante un **bind mount**.

| Tipo | Definición | Uso principal | 
| --- | --- | --- |
| Volumen | Administrado completamente por Docker en un área reservada | Bases de datos, datos persistentes de producción | 
| Bind Mount | Vincula una carpeta específica de nuestra computadora (/mi/codigo) dentro del contenedor | Desarrollo local (para ver cambios de código en vivo sin reconstruir la imagen) |

## 9. El ciclo de vida

Podemos resumir la idea de persistencia de esta manera:

```text
              ┌───────────────┐
              │   Imagen      │
              └───────┬───────┘
                      🠟
              ┌───────────────┐
              │   Contenedor  │
              └───────┬───────┘
                      │
                  usa │
                      🠟
              ┌───────────────┐
              │    Volumen    │
              │               │
              │     Datos     │
              └───────────────┘
```

El contenedor puede detenerse, eliminarse y reemplazarse.

El volumen puede permanecer:

```text
Contenedor A
     🠟
eliminado contenedor A
     🠟
Volumen
     🠟
Contenedor B
     🠟
eliminado contenedor B
     🠟
Volumen
     🠟
Contenedor C
```

Por eso, cuando diseñamos un proyecto que utiliza datos persistentes, debemos preguntarnos:

> **¿Qué datos deben sobrevivir si reemplazo el contenedor?**

Esos datos son candidatos a almacenarse en un volumen.

## 10. Limpieza

Docker permite listar los volúmenes:

```text
docker volume ls
```

Eliminar un volumen específico:

```text
docker volume rm datos
```

También existe:

```text
docker volume prune
```

pero debemos utilizarlo con cuidado porque puede eliminar volúmenes que ya no estén siendo utilizados por contenedores. Se recomienda especial precaución con `prune`, precisamente porque puede afectar otros volúmenes existentes. 

Para nuestro trabajo, es preferible eliminar explícitamente el volumen que sabemos que ya no necesitamos:

```text
docker volume rm datos
```

## 11. Flujo de trabajo

Cuando un proyecto necesita almacenar datos:

```text
Proyecto
   🠟
Identificar datos persistentes
   🠟
Crear volumen
   🠟
Montar volumen en el contenedor
   🠟
Probar escritura de datos
   🠟
Eliminar/recrear contenedor
   🠟
Comprobar que los datos continúan
```

Esta última prueba es especialmente importante.

No alcanza con declarar un volumen. Tenemos que comprobar que realmente cumple su propósito.

## Lo que necesitamos recordar

- Un contenedor tiene su propio sistema de archivos
- Los datos que necesitan sobrevivir al reemplazo del contenedor deben almacenarse fuera de su ciclo de vida
- Un **volumen*- permite almacenar datos de forma independiente del contenedor
- Un volumen puede utilizarse nuevamente con otro contenedor
- `docker volume create` permite crear un volumen
- `docker volume ls` permite listar los volúmenes
- `docker volume inspect` permite consultar información sobre un volumen
- `docker volume rm` permite eliminar un volumen específico
- `docker volume prune` debe utilizarse con precaución
- Las bases de datos son un caso típico de uso de volúmenes
- En Docker Compose también podemos declarar y administrar volúmenes
- La persistencia debe **probarse**, no solamente configurarse