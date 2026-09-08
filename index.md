# Índice y resumen ejecutivo

## Resumen

Este repositorio documenta la **validación de la plataforma TuxConsole** (panel de gestión sobre Proxmox). El trabajo siguió el "recorrido de prueba" sugerido por el manual de operación de la plataforma y lo extendió con **pruebas funcionales reales** (creación/borrado de recursos sobre el nodo Proxmox `nariv`), verificadas de extremo a extremo.

**Veredicto:** la plataforma está **operativa y funcional**. Las 8 funcionalidades del recorrido oficial operan según lo documentado, salvo las que la propia guía condiciona a la infraestructura protegida del sistema o al estado de licencia/appliance. Se listan hallazgos y oportunidades al final.

## Tabla de cobertura (guía vs. comportamiento real)

| # | Funcionalidad (guía) | Qué se probó | Resultado |
|---|---|---|---|
| 1 | Dashboard y Nodos | Métricas en vivo, almacenamiento, tema | ✅ |
| 2 | Catálogo LXC → crear contenedor | Instalación real `qa-nginx` (TurnKey 18), HTTP 200, luego eliminado | ✅ |
| 3 | Consola gráfica de VM | VM única es infraestructura **protegida** → no arrancada | ⚠️ |
| 4 | Despliegue desde repo (Dockerfile) | Repo GitHub propio → build → contenedor vivo | ✅ |
| 5 | Publicar en dominio (Sitios web) | Sitio local `tuxqa-hello.local`, ruteo por nombre verificado | ✅ |
| 6 | Crear BD y conectar la app | Instancia PostgreSQL 15 + CRUD real + conexión TCP | ✅ |
| 7 | Crear usuario Operador (permisos) | Enforcement de Operador, Solo lectura y rol a medida verificado (usuarios temporales borrados) | ✅ |
| 8 | Backup y restauración | Snapshot + rollback real validados sobre VM de prueba (eliminada) | ✅ |
| 9 | Publicación a internet (túnel) | App expuesta por subdominio `tuxadvisor.net` (HTTPS 200) y retirada | ✅ |
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
- Artefactos de despliegue (repo público de ejemplo `tuxqa-hello`).
