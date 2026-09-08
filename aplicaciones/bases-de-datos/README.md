# Bases de datos (instancias PostgreSQL / MariaDB)

Crea instancias en contenedores dedicados y las administra (`/bases-de-datos`).

## Qué dice la documentación

- Instancia aislada en su propio contenedor con dirección fija.
- Motores: PostgreSQL y MariaDB. Permisos de solo lectura o lectura-escritura. Acceso remoto controlado.
- Las contraseñas se generan en el panel y **se muestran una sola vez** (no se guardan en ningún sitio).
- Los contenedores de BD son infraestructura protegida y no aparecen en Contenedores LXC (se gestionan desde esta pantalla).

## Qué se hizo para probar (ciclo completo real)

1. **Crear instancia PostgreSQL 15** (`POST /api/databases`): nodo `nariv`, 1 core / 1 GB / 8 GB disco, IP fija `192.168.8.50/24`. Job de aprovisionamiento → `vmid 106` → estado final *"postgresql 15 listo en 192.168.8.50"*.
2. **Crear una base** (`appdata`).
3. **Crear usuarios por nivel** y probar la semántica de permisos.
4. **Habilitar acceso remoto**.
5. **Conectarse por TCP** a `192.168.8.50:5432` con un cliente PostgreSQL nativo y ejecutar CRUD real.
6. **Eliminar la instancia** al terminar y confirmar que la IP dejó de responder.

## Resultado

| Acción | Resultado |
|---|---|
| Instancia PG15 creada | ✅ (LXC 106, IP 192.168.8.50, motor instalado) |
| Base `appdata` | ✅ `{"created":"appdata"}` |
| Usuario por nivel | ✅ solo_lectura / lectura_escritura / administrador |
| Acceso remoto | ✅ habilitado |
| Conexión TCP real | ✅ PostgreSQL 15.19 (Debian) alcanzable |
| CRUD (CREATE/INSERT/SELECT) | ✅ con rol `administrador` |
| Vuelta del contenedor a `local-ai` | ✅ |

### Prueba de permisos (hallazgo relevante)

- Rol `app_user` (**lectura_escritura**): se conecta, pero **`CREATE TABLE` falla** en `schema public` → `permission denied for schema public` (SQLSTATE 42501). Realiza DML sobre objetos existentes, pero **no DDL**.
- Rol `app_owner` (**administrador**): `CREATE TABLE` + `INSERT` + `SELECT` correctos. ✅

Conclusión: PG 15 deja de conceder `CREATE` en `public` por defecto → un rol "lectura-escritura" no puede migrar/crear tablas. No está documentado en el manual. Recomendación: otorgar `CREATE ON SCHEMA public` a los roles de lectura-escritura si deben poder migrar esquemas.

## Comandos / llamadas de prueba

```bash
TOK="Authorization: Bearer <token>"; API=http://192.168.8.10/api
# crear instancia
curl -sk -X POST -H "$TOK" -H "Content-Type: application/json" \
  -d '{"name":"qa-postgres01","engine":"postgresql","version":"15","node":"nariv","cores":1,"memory":1024,"disk_gb":8,"ip_mode":"static","ip_cidr":"192.168.8.50/24","gateway":"192.168.8.1"}' $API/databases
# job / estado
curl -sk -H "$TOK" $API/databases/jobs/<job_id>
# crear base / listar
curl -sk -X POST -H "$TOK" -H "Content-Type: application/json" -d '{"name":"appdata"}' $API/databases/1/databases
curl -sk -H "$TOK" $API/databases/1/databases
# usuario/permisos y acceso remoto
curl -sk -X POST -H "$TOK" -H "Content-Type: application/json" -d '{"username":"app_owner","database":"appdata","nivel":"administrador"}' $API/databases/1/users
curl -sk -X PUT -H "$TOK" -H "Content-Type: application/json" -d '{"remote_access":true}' $API/databases/1/network
# eliminar
curl -sk -X DELETE -H "$TOK" $API/databases/1
```

## Hallazgos (resumen)

1. **`nivel` en la API usa guion bajo**: `solo_lectura` | `lectura_escritura` | `administrador`. Con guion normal devuelve `400 "Nivel de privilegio no válido"`.
2. **`lectura_escritura` no hace DDL** en `public` (PG15). Ver arriba.
3. La contraseña de cada usuario se genera y **se devuelve una sola vez** (evidencia del flujo "una sola vez"): se pierde → regenerar (endpoint `reset-password`).
