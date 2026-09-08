# Plano de la API

Documentación técnica del backend REST que sustenta al panel (`http://192.168.8.10/api`).

## Autenticación

- **POST `/api/auth/login`** — form-urlencoded con `username` y `password`. Devuelve `access_token` + `refresh_token` + `token_type:bearer` (JWT).
- Cabecera de autorización: `Authorization: Bearer <access_token>`.
- El token caduca; ante `401` se reintenta con `POST /api/auth/login` (y existe `POST /api/auth/refresh`).
- `GET /api/auth/me` — perfil de la sesión actual (id, username, rol, permisos por área).

## Salud y licencia

- `GET /api/health` → `{"status":"ok"}`
- `GET /api/license` → modo `prueba`, `solo_lectura:false`, día restantes etc. En el ambiente: quedaban 26 días, no solo lectura.

## Superficie expuesta (documentada vía OpenAPI)

El backend expone **~130 rutas** organizadas por dominio. Según la versión bajo prueba, se detectaron (muestra representativa):

| Dominio | Rutas (prefijo) |
|---|---|
| Aplicaciones (catálogo) | `/api/apps` · `/api/apps/install`, `/api/apps/install/{job_id}` |
| Autenticación | `/api/auth/login|me|refresh` |
| Clúster / firewall | `/api/cluster/*` |
| Bases de datos | `/api/databases*` (instancias, bases, usuarios, grants, network, reset-password, jobs) |
| Despliegues | `/api/deploys/*` (apps, credenciales, jobs, setup, status) |
| Docker | `/api/docker/*` (status, containers, images, stacks) |
| Servicios de red | `/api/netservices/*` (DHCP, DNS) |
| Nodos Proxmox | `/api/nodes*` (qemu, lxc, storage, network, isos, firewall, hardware) |
| Gateway | `/api/opnsense/*` |
| Configuración | `/api/settings*` |
| Sitios web | `/api/sites*` (certificates, tunnel, targets, status) |
| Usuarios | `/api/users*` |
| VPN | `/api/vpn/*` |

> El archivo `openapi.json` del servicio documenta cada contrato; es la referencia fiable para construir llamadas (schemas, métodos y campos).

## Resumen de endpoints usados en las pruebas

| Función | Endpoint | Resultado clave |
|---|---|---|
| Crear instancia BD | `POST /api/databases` | job de aprovisionamiento → vmid/IP |
| Crear BD | `POST /api/databases/{id}/databases` | `{created:...}` |
| Crear usuario BD | `POST /api/databases/{id}/users` | pwd de una sola vez |
| Acceso remoto BD | `PUT /api/databases/{id}/network` | remote_access |
| Eliminar instancia BD | `DELETE /api/databases/{id}` | `{deleted:1}` |
| Instalar app del catálogo | `POST /api/apps/install` | job → LXC |
| Crear app de despliegue | `POST /api/deploys/apps` | app `status:nueva` |
| Desplegar | `POST /api/deploys/apps/{id}/deploy` | job → contenedor vivo |
| Crear sitio | `POST /api/sites` | proxy configurado |
| Detener LXC | `POST /api/nodes/nariv/lxc/{vmid}/status/stop` | async |
| Eliminar LXC | `DELETE /api/nodes/nariv/lxc/{vmid}` | `{deleted}` (tras stopped) |

## Buenas prácticas observadas (recomendadas para reproducir pruebas)

- Refrescar el token entre tandas largas (los `401` eran por caducidad, no por fallo del servicio).
- Los trabajos de aprovisionamiento/despliegue devuelven `job_id` y son **asíncronos**: hay que sondeando el job hasta `completed`/100 %.
- Toda la autenticación debe ir por HTTPS fuera del laboratorio (en este ambiente se accedió por HTTP de red local).
