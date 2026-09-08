# Usuarios y roles

Cuentas del panel y permisos por área (`/usuarios`).

## Qué dice la documentación

- Permisos por área; las áreas coinciden con las categorías del menú.
- Roles del sistema (no modificables ni borrables):
  - **Administrador**: acceso total, incluida gestión de usuarios y roles.
  - **Operador**: opera el día a día (máquinas/apps/red); **no** gestiona usuarios.
  - **Solo lectura**: consulta y auditoría; no modifica.
- Nadie puede cambiar su propio rol ni desactivarse (para no dejar el panel sin administradores).
- Desactivar conserva cuenta/historial; borrar no. Para baja temporal, desactivar.

## Qué se hizo para probar

- Lectura de cuentas y roles (`/api/users`, `/api/users/roles`, `/api/users/areas`).
- Verificación de las salvaguardas de la UI (roles del sistema bloqueados; no puedes eliminar tu propia cuenta).

## Resultado

| Rol | Infrastr. | Cómputo | Aplicaciones | Red | Usuarios | Cuentas |
|---|---|---|---|---|---|---|
| Administrador | Escritura | Escritura | Escritura | Escritura | Escritura | 1 |
| Operador | Lectura | Escritura | Escritura | Escritura | — | 0 |
| Solo lectura | Lectura | Lectura | Lectura | Lectura | — | 0 |

Cuenta actual: `admin` (Admon., Activa, rol Administrador). Salvaguardas presentes. Resultado: ✅.

## Qué queda pendiente

- **No se creó una cuenta Operador** para validar en vivo que *Usuarios* desaparece de su menú y que las escrituras en Infraestructura se rechazan. Es una prueba valiosa del manual (paso 7) pero modifica estado permanente (cambia la lista de cuentas); a ejecutar solo si se autoriza y, en ese caso, con una cuenta claramente desechable.

## Comandos

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/users
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/users/roles
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/users/areas
```
