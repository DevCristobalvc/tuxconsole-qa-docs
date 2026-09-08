# Contenedores Docker

Administración del Docker del host de despliegues (`/docker`), estilo Portainer.

## Qué dice la documentación

- El panel administra contenedores del host de despliegues: listado en marcha con consumo en vivo, registros, arranque/parada, imágenes, y creación por formulario o por pilas (`docker-compose`).
- Antes de crear se puede pedir la **vista previa del comando** que se ejecutará.
- Los contenedores de una **aplicación desplegada** quedan marcados (`managed_by:"app"`) y **no se pueden borrar desde aquí**: se retiran desde *Despliegues* (evitar huérfanos en el proxy).
- Opciones tipo `--privileged` o montar el disco del host equivalen a control total; la pantalla agrupa y advierte.

## Qué se hizo para probar

1. Estado del host Docker (`/api/docker/status`): host `192.168.8.12`, **Docker 29.8.0**, "Servidor de contenedores operativo".
2. Listado de contenedores e imágenes: vacío al inicio (servidor sin cargas).
3. Tras desplegar una app (ver `aplicaciones/despliegues/`), se **verificó en este listado** la aparición del contenedor de la app con su marcado interno.

## Resultado

| Campo | Valor observado |
|---|---|
| Host | `192.168.8.12` · Docker 29.8.0 · operativo |
| Estado inicial | sin contenedores ni imágenes |
| Estado tras despliegue | contenedor `tuxqa-hello` `running` |
| Marcado interno | `managed_by:"app"` ✅ (coherente: no borrable desde este panel) |
| Puertos | `0.0.0.0:20000->5000/tcp` |

Resultado: ✅ administración y listado correctos; el marcado de apps desplegadas coincide con la documentación.

## Comandos / llamadas de prueba

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/docker/status
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/docker/containers
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/docker/images
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/docker/stacks
```

## Hallazgos / notas

- No se crearon contenedores Docker sueltos (el host está previsto como host de despliegues; la creación de apps va por *Despliegues*). La vista previa de comandos y la creación por composición quedan para cubrir con una pila de ejemplo.
