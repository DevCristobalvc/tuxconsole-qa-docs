# Configuración

Conexiones del panel: Proxmox, gateway, proxy, host de despliegues, licencia y publicación (`/configuracion`).

## Qué dice la documentación

- Todo lo parametrizable: conexión con Proxmox, gateway, proxy inverso y host de despliegues, además de licencia y publicación a internet.
- Cada grupo trae un botón para **probar la conexión antes de guardar** (evita el fallo silencioso de listas vacías por una dirección mal puesta).
- Los secretos nunca se muestran: solo últimos caracteres. Dejar un campo de secreto vacío = "no lo cambies".

## Qué se hizo para probar

- Lectura de todos los grupos de configuración y su estado.

## Resultado

| Grupo | Valor / estado |
|---|---|
| Proxmox | `https://192.168.8.200:8006` · token `<id-de-token-proxmox>` · certificado autofirmado sin verificar (esperado) |
| Gateway OPNsense | vacío (sin conexión) |
| Proxy inverso | NPM `http://192.168.8.11:81` · credenciales guardadas (últimos chars) |
| Host de despliegues | `192.168.8.12` · root · `/srv/tuxconsole-apps` · rango puertos `20000–20999` |
| Seguridad | lista de hosts permitidos del panel |
| Publicación a internet | túnel con dominio del proveedor **tuxadvisor.net**; campo opcional de token Cloudflare para dominio propio |

Resultado: ✅ todos los grupos presentes y coherentes; botones "Probar conexión" por grupo.

## Hallazgos / notas

- Los secretos se muestran truncados (últimos caracteres), tal como documenta la guía.
- No se modificó ninguna configuración (lectura). El host de despliegues ya estaba conectado y operativo en el momento de la prueba.
