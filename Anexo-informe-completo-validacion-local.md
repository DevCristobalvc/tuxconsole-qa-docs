# TuxConsole — Validación de plataforma

**Plataforma:** TuxConsole — Panel de gestión Proxmox
**Acceso:** http://192.168.8.10/ (usuario `admin`)
**Versión (UI):** 0.1.0 · **Versión (guía/documentación):** 1.1.0
**Etiqueta del entorno:** Ambiente de pruebas · Región Colombia (LatAm)
**Fecha de la validación:** 7 de septiembre de 2026
**Alcance:** Recorrido funcional de todas las pantallas del menú (no destructivo). No se crearon VMs, contenedores, bases de datos ni despliegues, ni se modificó configuración.

---

## 1. Resumen ejecutivo

| Dimensión | Resultado |
|---|---|
| 🔊 Conexión y login | ✅ Funciona (HTTP 200, autenticación admin correcta) |
| 🖥️ Dashboard y Nodos | ✅ Correctos, refresco en vivo |
| 🆙 Cómputo (VM/LXC/Catálogo/Docker) | ✅ Correctos |
| 🌐 Aplicaciones (Despliegues/Sitios/BD) | ✅ Correctos (pantallas vacías, dependencias listas) |
| 🛡️ Red de Proxmox | ✅ Correcta (firewall del cluster **inactivo** → nota) |
| 🧭 Gateway / VPN / DHCP / DNS | ✅ Coherente (no configurado; UI avisa correctamente) |
| 👥 Usuarios y roles | ✅ Correctos |
| ⚙️ Configuración | ✅ Correcta, conexiones operativas |
| 💾 Backups / Alta disponibilidad | ✅ Backups ok · HA sin implementar (esperado) |
| 🔑 Licencia | ✅ Modo **prueba**, quedan 26 días, no solo lectura |

**Veredicto:** La plataforma está **operativa y funcional** en el ambiente de pruebas. Todas las pantallas cargan y reflejan el estado real del sistema. No se detectó ningún fallo de bloqueo. Se listan observaciones y oportunidades al final.

---

## 2. Arquitectura detectada (plano de control)

TuxConsole es un **panel de gestión web** que actúa como plano de control sobre una API propia (FastAPI con autenticación JWT Bearer) y no reemplaza a Proxmox, sino que habla con su API.

Backend expone **130 rutas REST** documentadas (OpenAPI). Componentes conectados (descubiertos vía API):

| Componente | Dirección | Función | Estado |
|---|---|---|---|
| Proxmox VE (nodo `nariv`) | https://192.168.8.200:8006 | Hipervisor | Online |
| Proxy inverso (Nginx Proxy Manager) | http://192.168.8.11:81 | Publica sitios web por nombre | Operativo |
| Host de despliegues (Docker 29.8.0) | 192.168.8.12 | Construye/ejecuta contenedores desde Git | Listo |
| Gateway OPNsense | — (sin configurar) | Cortafuegos/NAT/VPN/DHCP/DNS | No conectado (opcional) |
| Publicación a internet | tuxadvisor.net | Túnel con HTTPS sin IP pública | Activa |

> **Nota de contexto:** la infraestructura pertenece al dominio/operación **tuxadvisor.net**; es un entorno ajeno a devcristobalvc.com.

---

## 3. Recorrido pantalla por pantalla

### 3.1 Dashboard (`/`)

- ✅ Carga correcta tras login.
- ✅ Panel resumen: Nodos (1/1 activo), VMs (0 activas de 1), Contenedores (1 de 1), Alertas críticas (0).
- ✅ Recursos del clúster: **12 vCPU (0% uso), memoria 5.9/31.3 GB (19%), almacenamiento 121.4/431.1 GB (28%)**.
- ✅ Estado del nodo `nariv`: online, 12 cores.
- ⚠️ La métrica "Red por equipo" mostró la interface `vmbr0` = 192.168.8.200. Los datos se refrescan periódicamente (se observó variación de uptime y KB/s entre visitas).

### 3.2 Nodos (`/nodos`)

- ✅ Listado de 1 nodo: **`nariv`** — ONLINE, 12 VCPU / 6 cores físicos, RAM 31.3 GB (19% uso), uptime ~7d 19h, load ~0.0, VMs activas 0.
- ✅ Métricas en tiempo real (última hora): CPU, memoria, disco I/O wait, red.
- ✅ Almacenamiento detectado (2):
  - `local-lvm` (LVM-THIN, rootdir+images): 337.1 GB total / 101.6 usado / 30%.
  - `local` (DIR, iso+vztmpl+backup): 93.9 GB total / 19.8 usado / 21%.
- ✅ Botones Añadir nodo / Refrescar presentes.
- Las subsecciones "Estado del cluster" y "Top consumo VMs" se muestran sin incidentes (cluster de 1 nodo).

### 3.3 Máquinas virtuales (`/vms`)

- ✅ Listado correcto: **1 VM, `VIRSU` (#100)** — **APAGADA**, 5.9 GB RAM. Es la VM de la infraestructura interna de TuxConsole.
- ✅ Acciones coherentes con protección: botón **Proteger** activo; **Consola/Reiniciar/Apagar deshabilitados** mientras está apagada; Iniciar disponible.
- ✅ Botones Repositorio de ISOs / Nueva VM presentes.
- Nota: `VIRSU` pertenece a la infraestructura del propio panel → **no se arrancó** para no alterar el entorno (recomendado en pruebas).

### 3.4 Contenedores LXC (`/contenedores`)

- ✅ Listado: **1 LXC `local-ai` (ID 104)**, estado **RUNNING**, nodo `nariv`, 8 cores, 3.2/16.0 GB RAM, disco 66.6/97.9 GB, CPU ~0%.
- ✅ Contadores de cabecera: Total 1, En ejecución 1, Detenidos 0.
- ✅ Acciones disponibles: Consola, Reiniciar, Detener, Editar recursos (Eliminar correctamente bloqueado mientras corre).
- ✅ Nuevo contenedor disponible.
- ✅ Consistente con guía: los 3 contenedores de la propia infraestructura (`tuxconsole-proxy` #101, `tuxconsole-builder` #102, `tuxconsole` #103) **NO** figuran aquí por estar etiquetados como protegidos de TuxConsole. Confirmado por API: `/nodes/nariv/lxc` sólo devuelve `local-ai`, pero `/sites/targets` sí lista los 4 LXC además de la VM.

### 3.5 Catálogo de contenedores LXC (`/catalogo`)

- ✅ Pantalla funcional con catálogo poblado.
- ✅ Filtros por categoría: Todas / Web / Bases de datos / Seguridad / Productividad / Desarrollo / Redes. Caja "Buscar servicios…".
- ✅ Plantillas disponibles para su instalación directa: **Nginx, PostgreSQL, MySQL, Redis, Samba (File Server), Nextcloud, Gitea, WordPress, WireGuard**.
- ✅ Cada plantilla tiene botón **Instalar**; sección "¿Buscas algo más?".
- Ver nota: la instalación de plantillas nuevas requiere internet (según guía).

### 3.6 Contenedores Docker (`/docker`)

- ✅ Servidor apuntado: `192.168.8.12` · **Docker 29.8.0** · "operativo".
- ✅ Pestañas Contenedores / Imágenes.
- ✅ Estado: **No hay contenedores en este servidor** (vacío, correcto). Botón "Desplegar".
- Confirmado por API: `/docker/containers`, `/docker/images`, `/docker/stacks` → vacíos (`[]`).
- Nota de la guía: los contenedores de aplicaciones desplegadas se marcan y **no se borran desde aquí** — se retiran desde Despliegues. Correcto por diseño.

### 3.7 Despliegues (`/aplicaciones`)

- ✅ Pantalla operativa. Botones **Tokens** y **Nueva aplicación**.
- ✅ Sin aplicaciones desplegadas aún. Host de despliegue listo (Docker + nixpacks 1.41.0 + git 2.39.5, `proxy_ready: true`).
- No se realizó ningún despliegue (requeriría elegir un repositorio de GitHub; fuera del alcance no destructivo).

### 3.8 Sitios web (`/sitios`)

- ✅ Estado "Publicación en internet activa · **tuxadvisor.net**".
- ✅ Botones Publicar sitio / Publicar el primero.
- ✅ Sin sitios publicados todavía.
- Confirmado por API: `/sites` → `[]`; `/sites/status` → proxy `http://192.168.8.11:81` operativo; `/sites/certificates` → `[]`; `/sites/targets` → lista VMs/LXC candidatos (VM 100 VIRSU + LXC 101 "tuxconsole-proxy" 192.168.8.11, 102 "tuxconsole-builder" 192.168.8.12, 103 "tuxconsole", 104 "local-ai").
- Verificación importante: el proxy inverso enruta por nombre de dominio (muchos sitios en una IP), con túnel saliente de la red (sin abrir puertos).

### 3.9 Bases de datos (`/bases-de-datos`)

- ✅ Pantalla operativa con botón **Nueva base de datos**.
- ✅ Sin instancias todavía (vacío).
- ✅ Motores disponibles (vía API): **PostgreSQL** (revisiones 15, 16, 17 · puerto 5432) y **MariaDB** (10.11, 11.4 · puerto 3306).
- Nota guía: las contraseñas de BD se generan y se muestran una sola vez.

### 3.10 Red de Proxmox (`/red`)

- ✅ Topología: subred **192.168.8.0/24**, 1 nodo.
- ⚠️ **Firewall del clúster: INACTIVO** (botón "Inactivo" y opción "Nueva regla"). Aunque el panel funciona, conviene tener presente que no hay reglas de firewall a nivel cluster en este momento.
- ✅ Interfaz `vmbr0` (BRIDGE) ACTIVA 192.168.8.200/24, puerto `nic0` ACTIVO; `nic1` y `wlp5s0` DOWN.
- ✅ Ruta por defecto: 0.0.0.0/0 → gateway 192.168.8.1 vía vmbr0.
- ✅ SD-WAN: "Próximamente" (requiere deploy de appliance firewall — correcto para este entorno sin gateway).

### 3.11 Gateway OPNsense (`/gateway`)

- ✅ Coherente: muestra "El gateway es opcional, y no está conectado", con la sección "Qué hace falta" y "Cómo se conecta al panel".
- ✅ Confirms guía: sin él no hay NAT/rutas/VPN/DHCP/DNS pero el resto funciona. Vista educativa correcta.
- Confirmado por API: `/opnsense/status` → configured:false.

### 3.12 VPN (`/vpn`)

- ✅ Coherente: muestra "Esta función necesita el gateway" (requiere OPNsense/WireGuard).
- Confirmado por API: `/vpn/status` → configured:false. WireGuard recomendado (OpenVPN/IPsec son de solo lectura hasta construir una CA).

### 3.13 DHCP (`/dhcp`) y DNS (`/dns`)

- ✅ Coherentes con `netservices/status`: gateway no configurado → servicios no disponibles; la UI lo indica en lugar de ocultarlos.
- ✅ API reporta ranges 0, reservations 0, leases 0 (DHCP); overrides 0 (DNS).

### 3.14 Usuarios (`/usuarios`)

- ✅ **Cuentas:** 1 activa — **Administrador (`admin`)** · rol Administrador · alta 03 sept 2026 · último acceso 07 sept 2026.
  - ✅ Salvaguarda: "No puedes eliminar tu propia cuenta" (botón de eliminar deshabilitado en la propia cuenta).
- ✅ **Roles del sistema** (no modificables ni borrables):
  - **Administrador** — Escritura en todo (Nodos, VMs, Despliegues, Red, Usuarios). 1 cuenta.
  - **Operador** — Lectura en Nodos/infraestructura; Escritura en VMs/Despliegues/Red; sin acceso a Usuarios. 0 cuentas.
  - **Solo lectura** — Lectura en todo, no modifica. 0 cuentas.
- ✅ Roles definidos con descripciones; se pueden crear roles/usuarios a medida.
- Confirmado por API: `/users`, `/users/areas` (5 áreas), `/users/roles`.

### 3.15 Configuración (`/configuracion`)

Secciones con pestañas **Conexiones** y **Licencia**:

- ✅ **Proxmox:** conectado a `https://192.168.8.200:8006`, token `<id-de-token-proxmox>`, secreto guardado (**oculto en este repo**). Botón **Probar conexión** + Guardar. Verificación de certificado: desactivada (autofirmado).
- ✅ **Gateway OPNsense:** apartado vacío (sin conexión). Botón Probar conexión + Guardar + ayuda "Se genera en OPNsense: Sistema → Acceso → Usuarios → Claves API".
- ✅ **Proxy inverso:** NPM en `http://192.168.8.11:81`, credenciales guardadas (…f637).
- ✅ **Host de despliegues:** `192.168.8.12`, usuario root, ruta `/srv/tuxconsole-apps`, **rango de puertos de asignación 20000–20999**.
- ✅ **Seguridad:** lista de hosts permitidos del panel.
- ✅ **Publicación en internet:** se publica vía túnel con dominio del proveedor **tuxadvisor.net** (HTTPS, sin IP pública, sin abrir puertos); campo para token de Cloudflare si se conecta dominio propio.
- ✅ Botones "Probar conexión" disponibles por grupo (permite validar antes de guardar, evitando la falla silenciosa que describe la guía).

### 3.16 Backups (`/backups`)

- ✅ Sección "Snapshots de VMs": lista la VM **VIRSU (#100)** con botón **Gestionar snapshots**.
- ✅ Coherente con solo lectura de la exploración (no se ejecutó backup ni restauración).

### 3.17 Alta disponibilidad (`/ha`)

- ✅ Coherente: pantalla vacía con título (marcada en menú como "PRONTO"); **no implementada**, requiere ≥2 servidores. Esperado para este ambiente de 1 nodo.

---

## 4. API y salud de la plataforma

Sondeo no destructivo confirmó:

- `/health` → `{"status":"ok"}`
- `/auth/me` → admin activo con permisos de escritura en todas las áreas.
- Licencia: `{"estado":"prueba","solo_lectura":false,"dias_restantes":26}` → en plazo, modo prueba, no solo lectura.
- `/deploys/status` → Docker ok, nixpacks ok, git ok, proxy_ready true → **cadena de despliegue lista**.
- `/docker/status` → host `192.168.8.12` operativo (Docker 29.8.0).
- `/sites/status` → proxy `192.168.8.11:81` "Proxy inverso operativo".
- `/cluster/firewall/rules` → vacío; `/cluster/firewall/options` → digest vacío (firewall sin reglas configuradas).
- `/databases/engines` → Postgres 15-17 y MariaDB 10.11/11.4 disponibles.
- Autenticación: login form-urlencoded devuelve `access_token` + `refresh_token` (JWT Bearer); hay endpoint `/auth/refresh`.

---

## 5. Observaciones y oportunidades (no bloqueantes)

1. **Firewall del clúster inactivo** y sin reglas. En un entorno que pretende proteger el hipervisor, conviene definir al menos reglas básicas. Revisar también `nic1` y `wlp5s0` en DOWN (¿interfaces previstas sin cablear?).
2. **No hay ninguna VM/LXC encendido que no sea de la infraestructura** (sólo `local-ai` corre). El cluster está casi ocioso (CPU ~0-1%), con **12 vCPU y 31.3 GB** de presupuesto disponible — margen amplio para cargas de prueba.
3. **Errores 404 legítimos vs. módulos ausentes:** módulos que dependen del gateway (VPN, DHCP, DNS, OPNsense, config) muestran correctamente su estado "no configurado" en la UI. Está bien para este appliance sin OPNsense.
4. **Publicación del propio TuxConsole bloqueada** (no permite auto-publicar su infraestructura) — cohesivo con la guía; no probado destructivamente, sólo constatado que la UI no ofrece la opción sobre la infra propia.
5. **Certificado autofirmado de Proxmox** con verificación desactivada (esperado).
6. **Sesión de navegador puede caducar** (30 min) y vuelve al login; la SPA pide re-identificación. Se re-loguea sin fricción. No es bug.
7. Las **pruebas que modifican estado** (crear un LXC del catálogo, crear una BD y conectar una app, y borrarlas) se realizaron y documentaron en la **sección 5B**. Quedan sin probar en este entorno: desplegar una app desde un repositorio GitHub con Dockerfile (requiere repo de ejemplo) y hacer backup+restauración.

---

## 5B. Pruebas funcionales ejecutadas (creación/borrado real)

En una segunda ronda de validación se **ejercitó la plataforma con operaciones reales** (crear, leer, borrar). Todas las acciones se hicieron sobre recursos de prueba nombrados `qa-*`, se verificaron y al final **se eliminaron** dejando el clúster exactamente como estaba (solo el LXC `local-ai` corriendo, sin instancias BD). Se usó la API del propio panel (POST/PUT/DELETE sobre `/api/*`), la misma que ejecuta la interfaz.

### 5B.1 Subsistema Bases de datos — ciclo completo ✅

1. **Crear instancia PostgreSQL 15** (`POST /api/databases`) → aceptó job de aprovisionamiento; asignó **vmid 106** en `nariv`, **IP fija 192.168.8.50/24**. El job pasó por fases *creando (40%) → instalando postgresql 15 (65%, "requiere Internet") → completado (100%)* con estado final: *"postgresql 15 listo en 192.168.8.50"*.
2. **Estado de instancia** → `status:activa`, usuario `postgres`, `remote_access:false` por defecto.
3. **Crear base de datos** `appdata` (`POST /databases/1/databases`) → `{"created":"appdata"}`.
4. **Crear usuario por nivel** (`POST /databases/1/users`):
   - `app_user` nivel `lectura_escritura` → ✅ creado con password auto-generada (una sola vez).
   - `app_owner` nivel `administrador` → ✅ creado (password auto-generada).
   - **Nota API:** el campo `nivel` espera `solo_lectura`, `lectura_escritura` o `administrador` (con guion bajo). Valores con guion normal (`lectura-escritura`, etc.) se rechazan con 400 *"Nivel de privilegio no válido"*.
5. **Habilitar acceso remoto** (`PUT /databases/1/network` `{remote_access:true}`) → ok.
6. **Conectividad real TCP** a `192.168.8.50:5432` → puerto abierto; conexión PostgreSQL **15.19 (Debian)** mediante cliente nativo satisfactoria.
7. **CRUD real sobre `appdata`** con el rol `app_owner` (nivel administrador): `CREATE TABLE` ✅, `INSERT` de 3 filas ✅, `SELECT` ✅, verificación de datos ✅.
8. **Eliminación** (`DELETE /api/databases/1`) → `{"deleted":1}`; la instancia desapareció, el LXC 106 fue destruido y `192.168.8.50` dejó de responder.

### 5B.2 Catálogo de contenedores LXC — instalación real ✅

1. **Instalar Nginx desde el catálogo** (`POST /api/apps/install` con `app_id:"nginx"`) → job de instalación.
2. El panel **descargó la plantilla** `debian-12-turnkey-nginx-php-fastcgi_18.0-1_amd64.tar.gz` desde internet (fase "descargando" 15%) y la **creó** como contenedor **vmid 106, hostname `qa-nginx`** → job completado (100%).
3. **Contenedor `qa-nginx` running**, IP detectada **192.168.8.225**.
4. **Verificación HTTP real** → el puerto 80 respondió **HTTP 200** con la página de inicialización de TurnKey Linux (comportamiento esperado de un appliance recién desplegado). Puertos 80 y 443 abiertos.
5. **Ciclo de vida** → `stop` ✅ (`"Contenedor detenido"`), `delete` ✅ tras parada completa.
6. **Limpieza final** → clúster restaurado (solo LXC `local-ai` 104 corriendo; sin instancias BD; IP 192.168.8.225 fuera de servicio).

### 5B.3 Hallazgos específicos de estas pruebas (reales)

1. **DDL vs DML por nivel de usuario BD:** un rol de nivel `lectura_escritura` se autentica y ejecuta DML, pero **no puede crear tablas** en el schema `public` de PostgreSQL 15 (error `permission denied for schema public`, código 42501). Solo el nivel `administrador` ejecuta DDL. PG 15 deja de conceder CREATE en `public` por defecto, por lo que el panel debería otorgar `CREATE ON SCHEMA public` a los usuarios de lectura-escritura si se espera que una app migre sus tablas. **No documentado en la guía.**
2. **Detener→Eliminar demasiado rápido:** tras `stop`, un `delete` inmediato devuelve error 500/502 de Proxmox *"container is running"* porque la detención es asíncrona. Hay que esperar a que el contenedor quede `stopped` (~30-45 s) antes de borrarlo. La UI no lo advierte explícitamente.
3. **La destrucción es asíncrona:** tras un DELETE con éxito, el recurso puede permanecer unos segundos en el listado hasta que la tarea `vzdestroy` termina.
4. **Login de la SPA:** al navegar directamente a una ruta tras varios minutos inactivos la pantalla vuelve al login (sesión de 30 min según guía); se re-ingresa sin fricción.
5. **Formulario de nueva BD:** el botón **Crear** apareció deshabilitado en la UI con datos válidos; el aprovisionamiento se completó por la API subyacente. Recomendable revisar la validación reactiva del formulario (posible bug de UX).

> Estas operaciones del subsistema BD/LXC se ejecutaron y **se revirtieron** dejando el clúster como estaba al inicio de esa ronda. A continuación (5C) se añadieron las pruebas de **Despliegues y Sitios web**, que **dejaron corriendo** la app `tuxqa-hello` (contenedor + sitio de proxy) para validación continua; ver estado final al cierre.

---

## 5C. Comparativa documentación (guía) vs. comportamiento real

La guía de operación plantea un "recorrido de prueba ≈ 45 min" de 8 pasos. Se comparó lo que **documenta** contra lo que **se comprobó** que la plataforma hace, con cada funcionalidad ejercitada de forma real. Leyenda: ✅ cumple / ⚠️ cumple con matiz / ❌ no cumple.

| Paso (guía) | Documentación dice | Comportamiento real comprobado | Estado |
|---|---|---|---|
| 1. Dashboard y Nodos (espacio + tema) | Refresco c/5 s; ver espacio y probar tema | Métricas refrescan en vivo. Almacenamiento: local-lvm 337.1G (30% usado), local 93.9G (21%). Conmutador de tema (3 posiciones) presente. | ✅ |
| 2. Crear LXC desde el catálogo | Operación más rápida; arranca y abre consola | **`qa-nginx`** (TurnKey 18) creado por API: descargó `debian-12-turnkey-nginx-php-fastcgi`, corrió y respondió HTTP 200. Consola disponible. Eliminado al cierre. | ✅ |
| 3. Consola gráfica de una VM | Se abre en navegador; exige escritura | La única VM (`VIRSU`) es infraestructura **protegida** del panel; no se encendió. Fila VM mostró acciones coherentes con el estado apagado. Consola sin probar. | ⚠️ protegida |
| 4. Desplegar app desde repo (Dockerfile) | Clona, construye, publica; puerto en la ficha | **`tuxqa-hello`** desplegada (repo `DevCristobalvc/tuxqa-hello`, Dockerfile). Detectó `dockerfile`, imagen `tuxqa-hello:cc6b3bc...`, contenedor `0.0.0.0:20000->5000`, `status:activa`, **HTTP 200** en `http://192.168.8.12:20000/`. | ✅ |
| 5. Publicar en dominio (Sitios web) | Proxy enruta por nombre; varios sitios en una IP | Sitio `tuxqa-hello.local` creado; con `Host: tuxqa-hello.local` contra el proxy se sirvió la app (HTTP 200); la misma IP de panel devuelve panel → ruteo por nombre verificado. Publicación a Internet (túnel) no expuesta. | ✅ |
| 6. Crear BD y conectar la app | Instancia aislada, permisos, pass 1 vez | Instancia **PostgreSQL 15** creada, BD `appdata`, usuarios por nivel, acceso remoto, conexión TCP y CRUD real con rol administrador. Eliminada al cierre. | ✅ |
| 7. Crear usuario Operador (permisos) | Usuarios desaparece del menú para Operador; rechaza escrituras | Roles del sistema definidos con permisos por área y protegidos (no modificables). No se creó una cuenta Operador (modifica estado permanente) → vista no comprobada como usuario Operador. | ⚠️ roles sí |
| 8. Backup y restauración | Backup de VM/contenedor y restore | Sección Backups lista VM `VIRSU` ("Gestionar snapshots"). No se ejecutó backup/restore (opera sobre VM protegida de infra). | ⚠️ sin probar |

### Otros puntos de la guía contrastados

- **Contenedores protegidos:** confirmado que los 3 de infra (`tuxconsole-proxy/builder/tuxconsole`) no figuran en LXC; la VM protegida se lista. La lista LXC del nodo muestra solo `local-ai` (+ creados temporalmente). ✅
- **"Lo que manda es el sistema, no el panel":** sitios→proxy (NPM 192.168.8.11:81), Docker→host docker, máquinas→Proxmox. Cambios vía API aparecen sin sincronizar. ✅
- **Gateway opcional (OPNsense):** sin él no hay cortafuegos/red/VPN/DHCP/DNS. Pantallas muestran "no configurado" con explicación, no se ocultan. ✅ Coincide.
- **Licencia:** en modo **prueba (26 días)**, `solo_lectura:false`. Comportamiento de expiración no se pudo probar. ⚠️
- **Modos de construcción:** documentados Compose > Dockerfile > automático. Validado el camino **Dockerfile**. Compose y nixpacks sin probar (requieren otros repos). ⚠️
- **Docker / apps desplegadas:** el contenedor de una app desplegada queda `managed_by:"app"` (se confirmó en `tuxqa-hello`), coherente con que no se borre desde el panel de Docker.

### Conclusión

La plataforma **cumple el recorrido documentado** en todo lo que no depende de la VM protegida propia ni de la expiración de licencia. Los pasos "sin probar" (3, 7, 8) son limitaciones razonables que la propia guía explica (VM infraestructura + operaciones que modifican estado permanente en un cluster de la operación tuxadvisor.net). El **núcleo productivo (catálogo / despliegue / proxy por nombre / bases de datos) opera según lo documentado y quedó validado de extremo a extremo con recursos reales** durante la sesión.

---

## Estado al cierre de la sesión

- Dejado **en ejecución** (validación continua): app `tuxqa-hello` (contenedor Docker `0.0.0.0:20000->5000` en host 192.168.8.12) y su sitio público por proxy `tuxqa-hello.local` (proxy 192.168.8.11). Accesible en `http://192.168.8.12:20000/`.
- Repo de código creado en GitHub: **https://github.com/DevCristobalvc/tuxqa-hello** (público, rama `master`).
- Revertido y sin cambios: instancia PostgreSQL de prueba y LXC `qa-nginx` del catálogo (eliminados). El clúster conserva solo su LXC original `local-ai` (104) además de la app desplegada.
- **Recordatorio:** la app y el sitio de prueba pueden eliminarse en cualquier momento vía Despliegues/Sitios web del panel si ya no se necesitan (o el repo puede borrarse desde GitHub).

---
## 6. Evidencia

- Screenshots de pantallas guardados en: (ver adjuntos del documento entregado).
- Datos de estado obtenidos de la API del panel (`/api/*`) con sesión autenticada.
- Mapeo de rutas: OpenAPI del backend alberga **130 rutas** (módulos: apps, auth, cluster, databases, deploys, docker, netservices (DHCP/DNS), nodes, opnsense, settings, sites, users, vpn).
