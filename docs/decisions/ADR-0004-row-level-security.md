# ADR-0004: Row-Level Security con un rol de aplicación sin privilegios

- **Estado:** Propuesta
- **Fecha:** 30/10/2026
- **Specs relacionadas:** `specs/database.md` (DP-DB-06), `specs/security.md`, `specs/features/accounts/spec.md` (RNF-ACC-01), `specs/features/accounts/plan.md` §5 y §8, [ADR-0001](ADR-0001-stack-tecnologico.md)

## Contexto

DP-DB-06 (ADR-0001) aprobó aislar los datos financieros con dos capas: el filtro obligatorio por `user_id` en el código y Row-Level Security (RLS) de PostgreSQL como defensa en profundidad. `accounts` es la primera tabla financiera, así que aquí se decide cómo funciona RLS.

Hoy `finora-back` se conecta con un único rol, dueño de las tablas y superusuario en desarrollo y en las pruebas (imagen oficial de PostgreSQL). **Un superusuario, o un rol con `BYPASSRLS`, ignora RLS siempre**, incluso con `FORCE ROW LEVEL SECURITY`. Activar las políticas sin cambiar el rol no protegería nada y las pruebas pasarían igual.

Además, los servicios hacen varios `commit` dentro de una misma petición. Una variable de sesión fijada solo al principio de la petición se perdería después del primer `commit` si es local a la transacción.

## Decisión

1. **Rol de aplicación `finora_app`**: sin `LOGIN`, sin `SUPERUSER`, sin `BYPASSRLS` y sin ser dueño de ninguna tabla. Lo crea una migración (si no existe) con permisos `SELECT, INSERT, UPDATE, DELETE` sobre todas las tablas y uso de las secuencias, más `ALTER DEFAULT PRIVILEGES` para las tablas futuras.
2. **La API trabaja con ese rol**: el motor de la aplicación ejecuta `SET ROLE finora_app` al abrir cada conexión. Alembic y las tareas de mantenimiento usan el rol dueño, sin `SET ROLE`. Así basta una sola `FINORA_DATABASE_URL`.
3. **Usuario actual en la base de datos**: tras autenticar la petición, se guarda `user_id` en `Session.info` y un evento `after_begin` de SQLAlchemy ejecuta `SELECT set_config('finora.user_id', :id, true)` al comenzar **cada** transacción. El valor es local a la transacción, así que nunca se filtra a otra petición que reutilice la conexión del pool.
4. **Políticas**: cada tabla financiera tiene `ENABLE ROW LEVEL SECURITY` y una política `USING` y `WITH CHECK` de la forma `user_id = nullif(current_setting('finora.user_id', true), '')::uuid`. Sin usuario fijado, la consulta no ve ni escribe ninguna fila (falla cerrado).
5. **Las tablas de `authentication` no usan RLS** (el inicio de sesión busca al usuario antes de saber quién es), pero `finora_app` sí necesita permisos sobre ellas.
6. El filtro por `user_id` en el código **se mantiene**: RLS es la segunda capa, no la reemplaza.

## Alternativas consideradas

- **RLS sin cambiar de rol**: no tiene efecto con un superusuario o con el dueño (sin `FORCE`). Descartada.
- **Dos URL de conexión (rol dueño para migraciones y rol con `LOGIN` para la API)**: es lo más común en producción, pero exige crear y administrar credenciales nuevas en cada entorno antes de decidir el hosting (DP-ARQ-06). Se puede adoptar más adelante sin tocar las políticas: solo cambia quién ejecuta `SET ROLE`.
- **Fijar el usuario con `SET` (sin `LOCAL`) una vez por petición**: si una petición falla antes de limpiarlo, la conexión vuelve al pool con el usuario anterior. Descartada.
- **Solo filtro en el código**: contradice DP-DB-06.

## Consecuencias

- Un olvido del filtro por `user_id` en una consulta ya no expone datos de otro usuario: RLS devuelve cero filas.
- Las pruebas de integración ejercen RLS de verdad, porque la aplicación de prueba también hace `SET ROLE`. Se agrega una prueba que consulta sin usuario fijado y espera cero filas.
- Crear un rol requiere el permiso `CREATEROLE` al migrar. Si el proveedor de hosting no lo permite, el rol se crea a mano una vez y la migración solo lo verifica (riesgo registrado para DP-ARQ-06).
- Los roles son globales del servidor de PostgreSQL: varias bases de datos (desarrollo, E2E) comparten `finora_app`, con permisos por base de datos.
- Las consultas que necesiten ver datos de todos los usuarios (mantenimiento, futuras tareas programadas) deben ejecutarse con el rol dueño de forma explícita.
