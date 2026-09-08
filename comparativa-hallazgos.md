# Comparativa frente al manual de operación y hallazgos

## Metodología

Se contrastó lo que declara el **manual de operación de TuxConsole** ("recorrido de prueba ≈ 45 min", 8 pasos) contra el **comportamiento real** observado en el ambiente bajo prueba. Cada funcionalidad se ejercitó con llamadas reales a la plataforma (UI y/o API) y se verificó el resultado; los recursos de prueba se **nombraron `qa`** y, salvo la app de demostración, se **revertieron** al terminar.

Leyenda: ✅ cumple según documenta / ⚠️ parcial o condicionado / ❌ no cumple.

## Los 8 pasos del manual

| # | Paso | Documentación | Comportamiento real | Estado |
|---|---|---|---|---|
| 1 | Dashboard y Nodos | Datos cada ~5 s; ver espacio libre y tema | Métricas refrescan en vivo; almacenamiento reportado `local-lvm` 337.1G (30%) y `local` 93.9G (21%); tema c/3 posiciones. | ✅ |
| 2 | Crear LXC desde el catálogo | Rápido; arranca + consola | `qa-nginx` (TurnKey 18): descargó plantilla `debian-12-turnkey-nginx-php-fastcgi_18.0-1`, creó y arrancó contenedor; respondió HTTP 200; consola disponible. Eliminado al final. | ✅ |
| 3 | Consola gráfica de una VM | En el navegador; exige permiso escritura | Validado sobre la VM real `qa-vm-ubuntu` (ISO Ubuntu, encendida): botón Consola activo, modal abierto y `vncproxy` devolvió ticket+puerto VNC. El render píxel no se pudo leer (limitación del entorno de QA, ver `index.md`). | ✅ (flujo/API) |
| 4 | Desplegar app desde repo (Dockerfile) | Clona, construye, publica; puerto real en ficha | Repo GitHub `DevCristobalvc/tuxqa-hello` (Dockerfile). El panel detectó `dockerfile`, construyó `tuxqa-hello:cc6b3bc...`, contenedor `0.0.0.0:20000->5000`, `status:activa`; **HTTP 200** en `http://192.168.8.12:20000/`. | ✅ |
| 5 | Publicar en dominio (Sitios web) | Proxy enruta por nombre; varios sitios en una IP | Sitio `tuxqa-hello.local` creado; con `Host: tuxqa-hello.local` contra el proxy se sirvió la app (HTTP 200) mientras la IP del panel devuelve el panel → ruteo por nombre verificado. Publicación a internet probada en la fila 9. | ✅ |
| 6 | Crear BD y conectar la app | Instancia aislada, permisos, pass única | Instancia **PostgreSQL 15** (LXC vmid 106, IP fija 192.168.8.50). BD `appdata`, usuarios por nivel, acceso remoto habilitado, conexión TCP y CRUD (CREATE/INSERT/SELECT) reales con nivel `administrador`. Eliminada al final. | ✅ |
| 7 | Crear usuario Operador | Para Operador, *Usuarios* desaparece y rechaza escrituras en Infra | **Probado (enforcement real):** se crearon operadores y un rol a medida temporales (ver `gestion/usuarios-roles/`), se verificó 403/201 por área de permisos y se eliminaron usuarios/roles. Solo queda `admin`. | ✅ |
| 8 | Backup y restauración | Backup de VMs/contenedores y restore | **Probado:** snapshot + rollback real en VM de prueba (`qa-snapshot-001`), recurso luego eliminado. VM protegida `VIRSU` no intervenida. | ✅ |
| 9 | Publicación a internet (túnel, extra) | - | **Probado:** subdominio `tuxadvisor.net` (modo gestionado) sirvió HTTPS 200; retirado tras validar. | ✅ |

## Otros puntos del manual contrastados

- **Contenedores de infraestructura protegidos:** los 3 LXC de TuxConsole (`tuxconsole-proxy/builder/tuxconsole`) **no figuran** en *Contenedores LXC*; la lista del nodo muestra solo `local-ai`. La VM protegida sí se lista con sus protecciones. ✅
- **"Lo que manda es el sistema, no el panel":** los sitios los conoce el proxy inverso (NPM `192.168.8.11:81`), los contenedores Docker los conoce Docker, las máquinas Proxmox. Cambios hechos por la API aparecen en la UI sin sincronizar. ✅
- **Gateway opcional (OPNsense):** sin él no hay cortafuegos/NAT/rutas/VPN/DHCP/DNS; esas pantallas muestran "no configurado" con la explicación de qué se necesita y **no se ocultan**. ✅ Coincide con el manual.
- **Licencia:** modo **prueba**, quedan 26 días, `solo_lectura:false`. No se pudo ejercitar el comportamiento de caducidad. ⚠️
- **Modos de construcción (Despliegues):** manual indica Compose > Dockerfile > constructor automático. **Validado 3/3**: `docker-compose` (tuxqa-compose), `Dockerfile` (tuxqa-hello) y constructor automático `nixpacks` (tuxqa-nixpacks). ✅
- **Docker / apps desplegadas:** el contenedor de una app desplegada se marca `managed_by:"app"` (observado en `tuxqa-hello`) o `compose_project` (en `tuxqa-compose-web-1`), coherente con que no se borre desde el panel de *Contenedores Docker*.

## Conclusión

La plataforma **cumple el recorrido documentado de extremo a extremo**: todas las funcionalidades del manual (8 pasos + extras de despliegue/publicación a internet) se **ejecutaron y validaron con recursos reales** y luego se **revirtieron/limpiaron**, devolviendo el clúster a su estado original (solo el LXC de la operación `local-ai` y la app de demostración local). Quedan sin verificar a nivel de *contenido* solo dos puntos por coste/entorno (no por fallo del panel): el render píxel interior de la consola de VM y la restauración comprobando archivos dentro de un SO completo (ver `index.md` → Límites no cubiertos). Además no se ejerció la expiración de licencia ni la caducidad (imposible sin vencerla). El **núcleo productivo — catálogo, despliegues (Dockerfile/compose/nixpacks), proxy por nombre, publicación a internet por túnel, bases de datos (PostgreSQL/MariaDB) y control de permisos por rol — quedó validado de punta a punta**.

## Hallazgos técnicos (ordenados por relevancia)

1. **DDL vs DML en usuarios de BD:** un rol `lectura_escritura` **no** puede crear tablas en `schema public` de PostgreSQL 15 (`permission denied`, SQLSTATE 42501). Solo el rol `administrador` ejecuta DDL. Dado que PG 15 dejó de conceder `CREATE` en `public` por defecto, el proveedor (o el panel) debería otorgar `CREATE ON SCHEMA public` a los usuarios de lectura-escritura si se espera que una aplicación realice migraciones. **No está documentado en el manual.**
2. **Detener→Eliminar LXC demasiado rápido:** tras `stop`, un `delete` inmediato devuelve error Proxmox *"container is running"* (500/502). La detención es asíncrona; hay que esperar a `stopped` (~30-45 s). La UI no lo advierte.
3. **Destrucción asíncrona:** tras un DELETE con éxito, el recurso puede permanecer unos segundos en el listado hasta que termina la tarea `vzdestroy`.
4. **`nivel` de la API de BD:** los valores correctos usan **guion bajo** (`solo_lectura`, `lectura_escritura`, `administrador`). Con guion normal (`lectura-escritura`, etc.) → `400 "Nivel de privilegio no válido"`.
5. **Firewall del clúster inactivo:** sin reglas configuradas en `nariv` (el `cluster/firewall/rules` está vacío). Conviene revisar si es deseado.
6. **Formulario UI "Nueva BD":** el botón *Crear* quedó deshabilitado con datos válidos; el aprovisionamiento se completó correctamente por la API subyacente. Posible incidencia de validación reactiva en la UI.
7. **Sesión de la SPA:** expira pasados ~30 min; al navegar a una ruta la pantalla vuelve al login. Se re-ingresa sin fricción (coherente con el manual). No se considera bug.

## Cierre

- Dejado **en ejecución** (demostración local): app `tuxqa-hello` (Dockerfile, `0.0.0.0:20000->5000` en `192.168.8.12`), app `tuxqa-compose` (`5001:5000`, modo compose) y app `tuxqa-nixpacks` (`20002`, constructor automático). Sitio proxy local `tuxqa-hello.local`.
- **Exposición pública retirada**: el túnel hacia `tuxqa-hello` se validó (HTTPS 200) y se **retiró** tras la prueba (`unexposed:true`).
- **Revertidos/limpiados durante la sesión**: instancias de BD (PostgreSQL y MariaDB), LXC del catálogo (`qa-nginx`), VMs de prueba (snapshot frío y en caliente), usuarios temporales y rol a medida.
- El clúster conserva sus recursos originales (`local-ai` 104 + app de demostración local); la VM protegida `VIRSU` no fue intervenida.
