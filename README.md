# TuxConsole QA — Documentación de pruebas

> Validación de la plataforma **TuxConsole** (panel de gestión Proxmox + servicios) accesible en el ambiente de evaluación. Documentación del **escaneo completo de la plataforma**, las **pruebas funcionales ejecutadas** y la **comparativa contra el manual de operación**.

## Acceso al entorno bajo prueba

| Campo | Valor |
|---|---|
| Interfaz | `http://192.168.8.10/` (TuxConsole — Panel de gestión Proxmox) |
| Rol usado en la prueba | `Administrador` |
| Nodo Proxmox | `nariv` · 12 vCPU · 31.3 GB RAM · online |
| Backend | API REST (FastAPI, autenticación JWT), **~130 rutas** documentadas |
| Licencia | Modo prueba (26 días), no solo lectura |

## Estado de la prueba

- **Escaneo de todas las pantallas del menú** → `escaneo-plataforma/`
- **Pruebas funcionales con creación real** (catálogo LXC, bases de datos, despliegue desde GitHub, publicación en proxy) → `aplicaciones/`, `operacion/`
- **Comparativa contra el manual de operación** → `index.md` (# resumen)

## Estructura del repositorio

```
tuxconsole-qa/
├── README.md                    # Este archivo: resumen y cómo leer
├── index.md                     # Índice + resumen ejecutivo + tabla de cobertura
├── escaneo-plataforma/          # Validación visual/lectura de cada pantalla
│   ├── dashboard-nodos/
│   ├── maquinas-virtuales/
│   ├── catalogo-lxc/
│   └── contenedores-docker/
├── aplicaciones/
│   ├── despliegues/             # Deploy real desde GitHub (Dockerfile)
│   ├── sitios-web/              # Proxy inverso, ruteo por nombre
│   └── bases-de-datos/          # Instancia PG15 real + CRUD
├── operacion/
│   ├── red-proxmox-firewall/
│   └── gateway-vpn-dhcp-dns/
├── gestion/
│   ├── usuarios-roles/
│   ├── configuracion/
│   └── backups-ha/
├── planos-api/                  # Mapa de la API: rutas, auth, health, licencia
└── evidencias/                  # Capturas y artefactos de las pruebas
```

## Cómo leer

- Empieza por [`index.md`](index.md): resumen ejecutivo, tabla de cobertura y comparativa contra el manual.
- Cada `*/README.md` de feature describe: **qué dice la doc → qué se hizo → resultado → evidencias → hallazgos/conclusiones**.
- Los pasos de cada prueba incluyen el comando/llamada utilizada para poder **reproducirla**.

## Resumen corto de cobertura

| Funcionalidad | Resultado |
|---|---|
| Login + API | ✅ operativo |
| Dashboard y Nodos | ✅ operativo |
| Máquinas virtuales | ✅ listado/protección (VM infraestructura, sin arrancar) |
| Catálogo LXC | ✅ instalación real (qa-nginx), ciclo completo, revertido |
| Contenedores Docker | ✅ host operativo |
| Despliegue desde GitHub | ✅ **app tuxqa-hello desplegada y verificada (HTTP 200)** |
| Sitios web / proxy | ✅ publicación local + ruteo por nombre |
| Bases de datos | ✅ instancia PG15, usuarios, CRUD real, revertido |
| Red Proxmox / firewall | ✅ lectura (firewall inactivo → nota) |
| Gateway / VPN / DHCP / DNS | ⚠️ no configurados (correcto, dependen de appliance) |
| Usuarios y roles | ✅ roles verificados |
| Configuración | ✅ operativo |
| Backups / Alta disponibilidad | ⚠️ no ejecutados (VM protegida / no implementado) |

## ⚠️ Notas de seguridad / alcance

- Entorno de **evaluación de un tercero** (operación tuxadvisor.net). No se realizaron acciones destructivas permanentes; los recursos de prueba se revirtieron salvo la app `tuxqa-hello` que quedó en ejecución para validación.
- No se expuso ningún servicio a internet.
- Credenciales de la documentación: solo de referencia del ambiente de pruebas; no publicar hacia producción.
