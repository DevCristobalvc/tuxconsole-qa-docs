# Máquinas virtuales (VM)

Listado, ciclo de vida y consolas de las VMs (`/vms`).

## Qué dice la documentación

- El panel permite encender, apagar y reiniciar; crear máquinas nuevas; editar CPU/memoria/disco; gestionar ISO, instantáneas y hardware; abrir la consola gráfica en el navegador (sin cliente VNC aparte).
- Apagar/reiniciar siempre piden confirmación. Un apagado desde el panel equivale al botón físico (si el huésped no responde, no hay apagado ordenado).
- Abrir una consola exige permiso de **escritura** en Cómputo.
- Las VMs protegidas de la infraestructura del panel se listan con insignia "Protegida" y acciones deshabilitadas salvo retirar protección.

## Qué se hizo para probar

- Lectura del listado de VMs y de las restricciones sobre la VM de infraestructura.
- El cluster de evaluación contiene **una única VM: `VIRSU` (#100)**.

## Resultado

| Campo | Valor observado |
|---|---|
| VM | `VIRSU` · ID 100 · nodo `nariv` · RAM 5.9 GB · **apagada** |
| Protección | Sí — pertenece a la infraestructura interna de TuxConsole |
| Acciones | Iniciar disponible; consola/reiniciar/apagar deshabilitados por apagada; "Proteger" activo |
| Repositorio de ISOs | Contiene 3 ISOs (lubuntu 26.04, ubuntu 22.04.5 server, Win11_25H2 Spanish) |

Resultado: ✅ listado y protecciones correctas.

## Qué se decidió NO hacer y por qué

- **No** se arrancó la VM `VIRSU`: es de infraestructura interna del propio panel y la documentación indica que no debe alterarse. En consecuencia, la consola gráfica de VM (paso 3 del manual) no se pudo probar en este ambiente sin intervenir la infra ajena.

## Comandos / llamadas de prueba

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes/nariv/qemu   # estados de VM
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/nodes/nariv/isos   # imágenes ISO
```

## Hallazgos / notas

- Sub-sección pendiente de validación: creación y consola gráfica de una VM nueva (requiere arrancar un instalador de SO desde la ISO, ~lento y pesado; no ejecutado por mantener el ambiente mínimo).
