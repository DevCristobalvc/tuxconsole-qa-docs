# Backups y Alta disponibilidad

Copias de seguridad y restauración, y disponibilidad del clúster.

## Qué dice la documentación

- **Backups**: copias de seguridad de máquinas y contenedores y su restauración. "Una copia sin restauración probada no es una copia": conviene incluir en las pruebas al menos una restauración completa.
- **Alta disponibilidad (HA)**: prevista para configurar y vigilar la HA del clúster; **aún no implementada** y requiere al menos dos servidores.

## Qué se hizo para probar

- Lectura de la sección *Backups*: lista la VM `VIRSU` (#100) con botón "Gestionar snapshots".
- Lectura de la sección *Alta disponibilidad*: pantalla presente y vacía (marcada "PRONTO" en el menú).

## Resultado

| Recurso | Estado |
|---|---|
| Backups | ✅ pantalla operativa (lista VM #100) |
| Snapshot/restauración real | ⚠️ NO ejecutado — recae sobre la VM `VIRSU`, que es infraestructura protegida del panel |
| Alta disponibilidad | ⚠️ No implementada (esperado; requiere ≥2 nodos) |

## Qué queda pendiente

- Ejecutar un **backup y restauración real** sobre un recurso propio (una VM/LXC de prueba, no la VM protegida). Requiere crear primero un recurso de prueba y, según el manual, validar el restore.

## Comandos / referencia

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes/nariv/qemu   # VMs candidatas a backup
```
