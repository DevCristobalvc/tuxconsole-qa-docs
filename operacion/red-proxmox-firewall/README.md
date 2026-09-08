# Red de Proxmox y firewall

Interfaces, puentes y reglas del cortafuegos de Proxmox (`/red`).

## Qué dice la documentación

- La pantalla protege el **hipervisor en sí**: interfaces/puentes y reglas del cortafuegos propio (nodo y cluster), independiente del router de la sede.
- Una regla mal puesta puede dejar el propio panel sin acceso; conviene probar teniendo a mano la consola física o la interfaz nativa.

## Qué se hizo para probar

- Lectura de la topología, interfaces, ruta por defecto y estado del firewall del cluster.

## Resultado

| Campo | Valor observado |
|---|---|
| Subred | `192.168.8.0/24` · 1 nodo |
| Interfaz activa | `vmbr0` (BRIDGE) · 192.168.8.200/24 · puerto `nic0` ACTIVO |
| Interfaces caídas | `nic1` (ETH) DOWN · `wlp5s0` (ETH) DOWN |
| Ruta por defecto | `0.0.0.0/0` → gateway `192.168.8.1` vía `vmbr0` |
| Firewall del cluster | **INACTIVO** · sin reglas configuradas |
| SD-WAN | "Próximamente" (requiere appliance de firewall) |

Resultado: ✅ lectura correcta.

## Hallazgo

- **El firewall del cluster está inactivo**, con el conjunto de reglas vacío. Recomendable revisar si es lo deseado en operación (aunque para pruebas no bloquea).

## Comandos

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes/nariv/network
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/cluster/firewall/options
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/cluster/firewall/rules
```
