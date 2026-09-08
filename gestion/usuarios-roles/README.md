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
- **Prueba funcional de permisos con cuentas temporales** (eliminadas al terminar). Se creó un usuario con rol **Operador** y otro con rol **Solo lectura**, se hizo login con cada uno y se probó que el backend **aplica de verdad** los permisos declarados (no solo la UI los muestra). Luego se borraron ambas cuentas, dejando de nuevo solo `admin`.

## Resultado

| Rol | Infrastr. | Cómputo | Aplicaciones | Red | Usuarios | Cuentas |
|---|---|---|---|---|---|---|
| Administrador | Escritura | Escritura | Escritura | Escritura | Escritura | 1 |
| Operador | Lectura | Escritura | Escritura | Escritura | — | (0 tras prueba) |
| Solo lectura | Lectura | Lectura | Lectura | Lectura | — | (0 tras prueba) |

### Verificación del enforcement (prueba real)

Con el token de un usuario **Operador**:
- `GET /api/users` → **403** *"Solo un administrador puede gestionar usuarios y roles"* ✅
- `GET /api/users/roles` → **403** ✅
- `GET /api/nodes` → **200** (puede leer infraestructura) ✅
- `GET /api/sites` → **200** (opera aplicaciones/sitios) ✅

Con el token de un usuario **Solo lectura**:
- `POST /api/databases` (crear BD) → **403** *"Tu rol no permite modificar «Despliegues, sitios web y bases de datos»"* ✅
- Detener un LXC → **403** *"Tu rol no permite modificar «Máquinas virtuales…»"* ✅
- Crear usuario → **403** ✅
- `GET /api/nodes` → **200** ✅

Los tres roles del sistema se comportan según la documentación: el control de permisos se aplica en el **backend**, no es solo visual.

## Qué queda pendiente

- La prueba confirmó el control de acceso server-side para los **3 roles del sistema** y para un **rol a medida**. Nota: el rol a medida (`QA_Solo_Apps`: escritura `solo` en aplicaciones, resto `ninguno`) se creó, se le asignó un usuario temporal, se validó y se **eliminó** tras la prueba (usuario + rol borrados). No se dejó ningún rol/usuario extra; el ambiente queda con solo los 3 roles del sistema y el usuario `admin`.

### Detalle de la prueba del rol a medida

1. `POST /api/users/roles` creó `QA_Solo_Apps` (permisos: infraestructura/cómputo/red/usuarios = `ninguno`; aplicaciones = `escritura`).
2. Se creó un usuario temporal con ese rol y se hizo login.
3. Enforcement verificado:
   - `GET /api/sites` → **200**; `POST /api/sites` (crear sitio) → **201** ✅ (puede escribir en aplicaciones).
   - `GET /api/nodes` → **403** ✅ (infraestructura `ninguno`).
   - Detener un LXC → **403** "Tu rol no permite modificar «Máquinas virtuales…»" ✅ (cómputo `ninguno`).
   - `GET /api/users` → **403** ✅ (usuarios `ninguno`).
4. Limpieza: sitio de prueba, usuario y rol eliminados (204).

## Comandos

```bash
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/users
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/users/roles
curl -sk -H "Authorization: Bearer <token>" http://192.168.8.10/api/users/areas
```
