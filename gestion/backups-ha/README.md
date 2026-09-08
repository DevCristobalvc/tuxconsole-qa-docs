# Backups y Alta disponibilidad

Copias de seguridad y restauración, y disponibilidad del clúster.

## Qué dice la documentación

- **Backups**: copias de seguridad de máquinas y contenedores y su restauración. "Una copia sin restauración probada no es una copia": conviene incluir en las pruebas al menos una restauración completa.
- **Alta disponibilidad (HA)**: prevista para configurar y vigilar la HA del clúster; **aún no implementada** y requiere al menos dos servidores.

## Qué se hizo para probar

- Lectura de la sección *Backups*: lista la VM `VIRSU` (#100) con botón "Gestionar snapshots".
- Lectura de la sección *Alta disponibilidad*: pantalla presente y vacía (marcada "PRONTO" en el menú).
- **Prueba funcional de snapshot + rollback (restauración) real** sobre un recurso de prueba propio (no la VM protegida): se creó una VM títere temporal vía API, se le creó un snapshot y se ejecutó el **rollback** (restauración), comprobando el ciclo de vida completo. Al terminar se eliminó la VM de prueba.
- Nota: la API de esta versión expone snapshots para **VMs (qemu)** — `snapshots` y `rollback`. No expone rutas de `vzdump` a nivel panel (el "Backup" de la UI usa snapshots de VM).

## Resultado

| Recurso | Estado |
|---|---|
| Backups (pantalla) | ✅ operativa (lista VM #100) |
| Crear snapshot (VM de prueba) | ✅ `qa-snapshot-001` persistente |
| Rollback / restauración | ✅ `{"restored":"qa-snapshot-001"}` |
| Alta disponibilidad | ⚠️ No implementada (esperado; requiere ≥2 nodos) |
| VM protegida `VIRSU` | NO intervenida ✅ |

### Detalle de la prueba de snapshot/rollback

1. `POST /api/nodes/nariv/qemu` creó una VM temporal (`qa-vm-backup`, 1 core / 512 MB / 4 GB en `local-lvm`, apagada).
2. `POST /api/nodes/nariv/qemu/106/snapshots` creó `qa-snapshot-001` → `upid: qmsnapshot...`.
3. `POST /api/nodes/nariv/qemu/106/snapshots/qa-snapshot-001/rollback` restauró la VM desde el snapshot → `{"restored":"qa-snapshot-001", "upid": "qmrollback..."}`.
4. La VM de prueba quedó intacta tras el rollback y se eliminó (`{"deleted":106}`), devolviendo el clúster a su estado original (solo `VIRSU`).

Conclusión: **snapshot y restauración funcionan** vía el panel/API. En una VM sin SO el snapshot del disco es un punto de no-retorno válido; en producción se probaría el restore completo de un sistema con datos.

## Qué queda pendiente

- Realizar un **backup/restauración completo probando el contenido** (datos) de una VM con sistema operativo. Requiere instalar/preparar una VM con datos antes de snapshot+rollback — pendiente del paso "consola/VM con SO completo", no ejecutado por coste, no por limitación del panel.

## Comandos / referencia

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes/nariv/qemu   # VMs candidatas a backup
```
