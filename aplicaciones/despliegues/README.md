# Despliegues (Deploy desde repositorio Git)

Da una dirección de repositorio; el panel clona, construye y levanta la aplicación (`/aplicaciones`).

## Qué dice la documentación

- Se indica un repositorio de GitHub (o Git); el panel lo clona, construye la app, la arranca y —si se puso dominio— la publica en el proxy. Puede quedar pendiente de cambios en la rama para redesplegar sola (`auto_deploy`).
- Modo de construcción elegido automáticamente en este orden: `docker-compose.yml` → Compose; `Dockerfile` → Dockerfile (vía más predecible) → "Constructor automático" (detecta lenguaje, nixpacks).
- Hay **dos puertos**, pero el formulario pide uno: el *interno* donde escucha la app en el contenedor. La dirección pública la asigna el panel en el rango **30000–30999** (en el despliegue real se observó host_port 20000 en el rango configurado 20000–20999) y aparece en la ficha.
- Guardar cambios **no** los aplica: quedan "Requiere redesplegar" hasta relanzar.
- No todo repositorio es desplegable: hace falta un Dockerfile/Compose/script de arranque (o el constructor automático lo detecta).

## Qué se hizo para probar

1. **Se creó un repositorio real** `tuxqa-hello` (GitHub, público) con un `Dockerfile` (Python `http.server` en puerto 5000) e `index.html`.
2. **Se registró la app** en el panel (`POST /api/deploys/apps`): `repo_url`, branch `master`, `app_port:5000`, `auto_deploy:true`, `build_mode:"dockerfile"`.
3. **Se disparó el despliegue** (`POST /api/deploys/apps/1/deploy`) y se monitoreó el job.
4. **Se verificó** el resultado por varios caminos independientes (panel, Docker y HTTP).

## Resultado

| Campo | Valor observado |
|---|---|
| App | `tuxqa-hello` · id 1 |
| Modo detectado | `dockerfile` |
| Imagen | `tuxqa-hello:cc6b3bc839f5` (from nuestro commit) |
| Contenedor | `running` · `0.0.0.0:20000->5000/tcp` · `managed_by:"app"` |
| Estado app | `activa` · last_commit `cc6b3bc...` |
| URL de acceso | **`http://192.168.8.12:20000/`** → **HTTP 200** (HTML de nuestra app) |

Resultado: ✅ despliegue de extremo a extremo funcionando (clone → build → run → servir).

**Repositorio fuente:** https://github.com/DevCristobalvc/tuxqa-hello (público, rama `master`).

## Comandos / llamadas de prueba

```bash
# crear la app
curl -sk -X POST -H "Authorization: Bearer <token>" -H "Content-Type: application/json" \
  -d '{"name":"tuxqa-hello","repo_url":"https://github.com/DevCristobalvc/tuxqa-hello.git","branch":"master","app_port":5000,"auto_deploy":true,"build_mode":"dockerfile"}' \
  http://192.168.8.10/api/deploys/apps
# desplegar
curl -sk -X POST -H "Authorization: Bearer <token>" http://192.168.8.10/api/deploys/apps/1/deploy
# estado / logs
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/deploys/apps
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/deploys/apps/1/logs
# comprobar acceso real
curl -s http://192.168.8.12:20000/
```

## Hallazgos / notas

- **Modos de construcción validados (3/3):**
  - **Dockerfile** → `tuxqa-hello` (HTTP 200 en `http://192.168.8.12:20000/`).
  - **docker-compose** → `tuxqa-compose` (repo sin Dockerfile+compose; `detected_mode:compose`; HTTP 200 en `http://192.168.8.12:5001/`, puerto que define el propio compose `5001:5000`). Coincide con la guía: en modo Compose **los puertos los define el fichero**, no el rango del panel.
  - **Constructor automático (nixpacks)** → `tuxqa-nixpacks` (repo Node/Express **sin** Dockerfile ni compose; `detected_mode:nixpacks`; HTTP 200 en `http://192.168.8.12:20002/`).
- El `access_token` del panel caduca (~minutos); refrescarlo con `/api/auth/login` antes de cada tanda (los `401 "Credenciales inválidas"` eran por token antiguo, no por fallo del host).
- Repos de ejemplo creados en GitHub: `DevCristobalvc/tuxqa-hello`, `DevCristobalvc/tuxqa-compose`, `DevCristobalvc/tuxqa-nixpacks`.
