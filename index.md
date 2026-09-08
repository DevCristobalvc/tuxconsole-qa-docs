# Índice y resumen ejecutivo

## Resumen

Este repositorio documenta la **validación de la plataforma TuxConsole** (panel de gestión sobre Proxmox). El trabajo siguió el "recorrido de prueba" sugerido por el manual de operación de la plataforma y lo extendió con **pruebas funcionales reales** (creación/borrado de recursos sobre el nodo Proxmox `nariv`), verificadas de extremo a extremo.

**Veredicto:** la plataforma está **operativa y funcional**. Todas las funcionalidades del recorrido oficial se ejecutaron y validaron de punta a punta con recursos reales (VMs, LXC, bases de datos, despliegues, sitio web público y control de permisos), salvo dos límites por *coste/seguridad* que se detallan más abajo (ver `## Límites no cubiertos y por qué`).

## Tabla de cobertura (guía vs. comportamiento real)

| # | Funcionalidad (guía) | Qué se probó | Resultado |
|---|---|---|---|
| 1 | Dashboard y Nodos | Métricas en vivo, almacenamiento, tema | ✅ |
| 2 | Catálogo LXC → crear contenedor | Instalación real `qa-nginx` (TurnKey 18), HTTP 200, luego eliminado | ✅ |
| 3 | Consola gráfica de VM | VM real `qa-vm-ubuntu` con ISO de Ubuntu encendida; botón Consola habilitado, modal abierto y `vncproxy` entregó ticket+puerto VNC | ✅ (flujo/API) |
| 4 | Despliegue desde repo (Dockerfile) | Repo GitHub propio → build → contenedor vivo | ✅ |
| 5 | Publicar en dominio (Sitios web) | Sitio local `tuxqa-hello.local`, ruteo por nombre verificado | ✅ |
| 6 | Crear BD y conectar la app | Instancias **PostgreSQL 15** y **MariaDB 11.4** + CRUD real + conexión TCP | ✅ |
| 7 | Crear usuario Operador (permisos) | Enforcement de Operador, Solo lectura y rol a medida verificado (usuarios/roles temporales borrados) | ✅ |
| 8 | Backup y restauración | Snapshot + rollback en **frío y en caliente** sobre VMs reales (disco 20G encendida) | ✅ |
| 9 | Publicación a internet (túnel) | App expuesta por subdominio del proveedor (HTTPS 200) y retirada | ✅ |
| 10 | Modos de despliegue | Dockerfile · docker-compose · constructor automático (nixpacks) — 3/3 | ✅ |

Detalle por pantalla → ver carpetas. Comparativa extendida y hallazgos → sección [Hallazgos y comparativa frente al manual](comparativa-hallazgos.md).

## Hallazgos principales

1. **Permisos de BD (PG15):** nivel `lectura_escritura` **no puede crear tablas** en `schema public` (`permission denied`, SQLSTATE 42501). Solo el nivel `administrador` ejecuta DDL. No está documentado en el manual. Más detalle en [`aplicaciones/bases-de-datos/`](aplicaciones/bases-de-datos/).
2. **Detener→Eliminar (LXC):** la detención es asíncrona; un borrado inmediato devuelve *"container is running"*. Hay que esperar a que quede `stopped`.
3. **Valores de `nivel` en la API de BD:** usan guion bajo (`solo_lectura`, `lectura_escritura`, `administrador`); con guion normal devuelven `400 "Nivel de privilegio no válido"`.
4. **Firewall del clúster inactivo**: sin reglas configuradas (el panel lo permite operar; revisar si es lo deseado).
5. **Formulario UI "Nueva BD"**: el botón *Crear* quedó deshabilitado con datos válidos; el aprovisionamiento se completó correctamente por la API (posible incidencia de UX/validación reactiva).
6. **Sesión SPA (30 min):** al navegar a una ruta tras inactividad vuelve al login; se re-ingresa sin fricción (documentado en el manual).

## Evidencias

- Capturas por pantalla y de la app desplegada en [`evidencias/`](evidencias/).
- Artefactos de despliegue (repos públicos de ejemplo `tuxqa-hello`, `tuxqa-compose`, `tuxqa-nixpacks`).

## Límites no cubiertos y por qué

Estos dos puntos **no pudieron verificarse a nivel de contenido**, y se dejan escritos por transparencia (ambos son de *coste/entorno*, no fallos del panel):

1. **Ver el render píxel interior de la consola de VM.** Se validó el flujo completo de consola (botón habilitado solo con VM encendida, modal abierto, y el servidor `vncproxy` que entrega el ticket VNC para el navegador). El **contenido visual** (pantalla del instalador/escritorio) no pudo leerse en la sesión de QA: el backend de análisis visual devolvió "unsupported image" de forma repetida y el DOM del visor noVNC quedó bloqueado por las restricciones de seguridad del navegador. No se afirma haber visto píxeles; solo que el flujo de apertura de consola responde.
2. **Restauración verificando el *contenido interno* de un SO** (instalar Ubuntu completo → inyectar archivos → snapshot → rollback → comprobar que los archivos volvieron). Se validó el snapshot+rollback en frío y en caliente sobre VMs con disco real, pero reproducir la instalación completa del SO requiere pasar el instalador interactivo por consola, no automatizable/verificable de forma fiable en esta sesión.

En ambos casos la **funcionalidad subyacente está operativa** (la operación de VM, snapshot/rollback y apertura de consola responden y se devuelven IDs/estados correctos); el único hueco es la comprobación profunda de contenido en pantalla y dentro del SO instalado.
