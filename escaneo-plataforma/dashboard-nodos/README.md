# Dashboard y Nodos

Pantalla de entrada del panel (`/`) y detalle del servidor físico (`/nodos`).

## Qué dice la documentación

- El Dashboard es la primera pantalla tras entrar: estado del servidor, consumo de CPU/memoria/disco, recuento de VMs y contenedores encendidos/apagados. Los datos se refrescan cada ~5 segundos.
- La pantalla *Nodos* muestra el detalle del servidor físico (procesador, memoria, almacenamiento con espacio libre, versión de Proxmox, uptime). Si el servidor está en cluster, aparecen todos los miembros.

## Qué se hizo para probar

- Acceso autenticado y lectura de la vista Dashboard y Nodos.
- Verificación de datos reales del hardware y del almacenamiento (contra la API `/api/nodes`).
- Comprobación de que las métricas se actualizan (uptime y tráfico de red variaron entre dos visitas).
- Verificación del conmutador de tema (3 posiciones: sistema/claro/oscuro) en la barra superior.

## Resultado

| Campo | Valor observado |
|---|---|
| Nodo | `nariv` · ONLINE |
| CPU | 12 vCPU / 6 núcleos · uso ≈ 0–1 % |
| Memoria | 5.9 / 31.3 GB (19 %) |
| Uptime | ~7 d (crece con el refresco) |
| Almacenamiento 1 | `local-lvm` (LVM-THIN) · 337.1 GB total / 101.6 usado / 30 % |
| Almacenamiento 2 | `local` (DIR) · 93.9 GB total / 19.8 usado / 21 % |
| VMs activas | 0 de 1 |
| Contenedores activos | 1 de 1 (`local-ai`) |
| Alertas críticas | 0 |

Resultado: ✅ funciona según documenta.

## Comandos / llamadas de prueba

```bash
# Nodos (estado real de Proxmox expuesto por el panel)
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/cluster/status
```

## Hallazgos / notas

- Distribución física → `/operacion/red-proxmox-firewall/`.
- Nada reseñable: funcionalidad correcta.
