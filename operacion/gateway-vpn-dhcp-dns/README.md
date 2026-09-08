# Gateway OPNsense · VPN · DHCP · DNS

Cortafuegos/red perimetral gestionable y servicios de red local.

## Qué dice la documentación

- El gateway (OPNsense) es **opcional**: sin él no hay cortafuegos/NAT/rutas/VPN/DHCP/DNS, pero el resto del panel funciona.
- Las pantallas de VPN/DHCP/DNS siguen visibles y **explican qué hardware hace falta** en lugar de esconderse.
- VPN: WireGuard es la vía recomendada (OpenVPN e IPsec son de solo lectura hasta construir una CA). Las claves privadas se muestran una sola vez.
- Con una sola interfaz de red el gateway no filtra nada (requiere dos interfaces); la pantalla lo advierte.

## Qué se hizo para probar

- Verificación del estado real de cada subsistema en este ambiente (que no dispone de appliance OPNsense conectado).

## Resultado

| Subsistema | Estado observado (API) | Coherente con doc |
|---|---|---|
| `/api/opnsense/status` | `configured:false` "El cortafuegos no está configurado" | ✅ |
| `/api/netservices/status` (DHCP/DNS) | `configured:false`; DHCP ranges 0 / reservations 0 / leases 0; DNS overrides 0 | ✅ |
| `/api/vpn/status` | `configured:false`; WireGuard no habilitado | ✅ |
| Vista Gateway en la UI | Explica que el gateway "no está conectado", qué se necesita y cómo vincularlo | ✅ |

Resultado: las pantallas de Gateway/VPN/DHCP/DNS están **operativas estructuralmente y muestran correctamente el estado "sin gateway"**, tal como documenta la guía (no se ocultan, explican el requerimiento).

## Qué queda pendiente

- Probar la administración completa de VPN/DHCP/DNS requiere conectar un appliance **OPNsense** (cortafuegos + router de la sede). No disponible en este ambiente → sin más validación posible.

## Comandos

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/opnsense/status
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/netservices/status
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/vpn/status
```
