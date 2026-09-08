# Sitios web (proxy inverso)

Publicación de servicios en un dominio con proxy reverso (`/sitios`).

## Qué dice la documentación

- El proxy inverso enruta **por nombre de dominio**: muchos sitios conviven en una misma IP.
- También puede exponerlos a internet por **túnel** (sin IP pública ni abrir puertos) con dos modalidades: subdominio del proveedor o dominio propio en Cloudflare.
- Exponer a internet es una decisión explícita con aviso.
- El panel **se niega a publicar su propia infraestructura** (así mismo, proxy, Proxmox, contenedores de BD).

## Qué se hizo para probar

1. Estado del servicio de sitios (`/api/sites/status`): proxy en `http://192.168.8.11:81` operativo.
2. **Publicación de un sitio local** (`POST /api/sites`): dominio `tuxqa-hello.local` → `forward_host 192.168.8.12` (host de despliegues) puerto `20000`, esquema http, websockets y bloqueo de exploits activados.
3. **Verificación del ruteo por nombre**: se consultó al proxy (`192.168.8.11`) con la cabecera `Host: tuxqa-hello.local`; el proxy devolvió la **app desplegada** (HTTP 200 con nuestro HTML). Contra la IP del panel la misma consulta devuelve el panel → confirma múltiples sitios en una IP diferenciados por nombre.

## Resultado

| Campo | Valor observado |
|---|---|
| Proxy | Nginx Proxy Manager · `192.168.8.11:81` · operativo |
| Sitio local creado | `tuxqa-hello.local` · enabled |
| Reenvío | → `192.168.8.12:20000` (http) |
| Ruteo por nombre | ✅ HTTP 200 con nuestra app usando `Host: tuxqa-hello.local` |
| Publicación a internet | No expuesto (decisión de no exponer al mundo sin confirmación) |

Resultado: ✅ publicación local y ruteo por nombre correctos.

## Comandos / llamadas de prueba

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/sites/status
curl -sk -X POST -H "Authorization: Bearer <token>" -H "Content-Type: application/json" \
  -d '{"domain_names":["tuxqa-hello.local"],"forward_host":"192.168.8.12","forward_port":20000,"forward_scheme":"http"}' \
  http://192.168.8.10/api/sites
# comprobar enrutamiento por nombre contra el proxy
curl -sk -H "Host: tuxqa-hello.local" http://192.168.8.11/
```

## Hallazgos / notas

- **Publicación a internet (túnel `tuxadvisor.net`): NO ejecutada — pendiente de autorización explícita.** Exponer un sitio abre al público un recurso local; la propia plataforma lo trata como decisión explícita y avisada. Al autorizar, se haría un túnel temporal sobre el dominio de prueba existente (`tuxqa-hello.local`) y luego se retiraría. La modalidad está disponible según `/api/sites/status` y `Configuración → Publicación en internet`.
- En paralelo se validó por completo la publicación **local** por proxy (ruteo por nombre en una única IP), ver `Resultado`.
