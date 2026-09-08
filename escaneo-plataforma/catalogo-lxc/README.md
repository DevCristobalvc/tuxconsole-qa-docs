# Catálogo de contenedores LXC

Catálogo de servicios listos para instalar (`/catalogo`) y operación de contenedores LXC (`/contenedores`).

## Qué dice la documentación

- Los contenedores LXC arrancan en segundos y consumen mucho menos que una VM (comparten el núcleo).
- El catálogo crea contenedores desde plantillas preparadas (TurnKey Linux).
- Descargar plantillas nuevas requiere internet; sin conexión la pantalla lo advierte y ofrece solo las ya descargadas.
- Los 3 contenedores de infraestructura de TuxConsole (`tuxconsole-proxy`, `tuxconsole-builder`, `tuxconsole`) **no aparecen** en el listado (etiqueta protegida); se listan las acciones de ciclo de vida y consola de texto.

## Qué se hizo para probar

1. **Escaneo del catálogo**: plantillas por categoría (Web, Bases de datos, Seguridad, Productividad, Desarrollo, Redes) con buscador y botón *Instalar*. Disponibles: Nginx, PostgreSQL, MySQL, Redis, Samba, Nextcloud, Gitea, WordPress, WireGuard.
2. **Instalación real de un LXC desde el catálogo** (`qa-nginx`):
   - `POST /api/apps/install` con `app_id:"nginx"`, nodo `nariv`, hostname `qa-nginx`.
   - El panel **descargó la plantilla** `debian-12-turnkey-nginx-php-fastcgi_18.0-1_amd64.tar.gz` desde internet (job: fase descargando → creando) y creó el contenedor.
   - El contenedor **arrancó** (IP detectada) y respondió **HTTP 200** con la página de inicialización TurnKey.
   - Se **detuvo** y **eliminó** al terminar (ciclo de vida completo).
3. Estado residual del nodo: solo el LXC `local-ai` (104) original.

## Resultado

| Acción | Resultado |
|---|---|
| Listar catálogo | ✅ (9 plantillas agrupadas por categoría) |
| Instalar LXC desde plantilla | ✅ job completado (100 %): "Nginx instalado como contenedor 106" |
| Contenedor arrancado | ✅ `running`, IP 192.168.8.225, HTTP 200 |
| Detener | ✅ (`Contenedor detenido`) |
| Eliminar (tras parada) | ✅ |
| Consola | disponible como botón por contenedor |

Resultado: ✅ funcionalidad completa de instalar/arrancar/detener/borrar un LXC desde el catálogo.

## Comandos / llamadas de prueba

```bash
# ver apps del catálogo
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/apps
# instalar una app del catálogo como LXC
curl -sk -X POST -H "Authorization: Bearer <token>" -H "Content-Type: application/json" \
  -d '{"app_id":"nginx","node":"nariv","hostname":"qa-nginx","cores":1,"memory":512,"disk":8,"password":"<clave-temporal-pruebas>"}' \
  http://192.168.8.10/api/apps/install
# ver job y listar LXC
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/apps/install/<job_id>
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes/nariv/lxc
```

## Hallazgos / notas

- Requiere **salida a internet** para descargar la plantilla TurnKey (coherente con la guía).
- El `DELETE` inmediato tras `stop` devuelve *"container is running"* (detención asíncrona → esperar a `stopped`). Más detalle en el índice de hallazgos.
