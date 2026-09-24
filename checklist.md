# Checklist de entrega

Antes de entregar tu proyecto al laboratorio, verifica que se cumplan todos los siguientes puntos:

## Construcción y Configuración Local
- [ ] El proyecto incluye un `Dockerfile`.
- [ ] Se incluyó un archivo `.dockerignore` (excluyendo `node_modules`, `.git`, `.env`, etc.).
- [ ] La imagen se construye correctamente sin errores (`docker build`).
- [ ] No hay credenciales, claves API ni secretos hardcodeados en el Dockerfile o la imagen.

## Prueba y Ejecución Local
- [ ] El contenedor inicia correctamente (`docker run`).
- [ ] La aplicación responde y funciona navegando desde `http://localhost:PUERTO`.
- [ ] Los puertos están documentados en el README o expuestos en el Dockerfile.
- [ ] La persistencia de datos (volúmenes) está configurada y probada (si corresponde).
- [ ] El archivo `compose.yaml` funciona con `docker compose up` (si corresponde).

## Publicación y Entrega al Laboratorio
- [ ] La imagen fue etiquetada y publicada exitosamente en Docker Hub (`docker push`).
- [ ] La imagen publicada puede descargarse (`docker pull`) y ejecutarse correctamente.

### Verificación de la imagen publicada

Se recomienda realizar esta prueba en una computadora diferente o en otro entorno:

```bash
docker pull USUARIO/PROYECTO:TAG
docker run ...
```

Si no es posible utilizar otra computadora, puede realizarse la prueba en la misma máquina eliminando previamente la imagen local:

```bash
docker image rm USUARIO/PROYECTO:TAG
docker pull USUARIO/PROYECTO:TAG
docker run ...
```