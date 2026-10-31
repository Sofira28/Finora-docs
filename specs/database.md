# Base de datos

| Campo | Valor |
|---|---|
| Estado | `Draft` |
| Última actualización | 30/10/2026 |

> Motor seleccionado: PostgreSQL ([ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md)). La §2 describe los **dominios de datos** de los módulos planificados (orientativos). La §5 registra los esquemas **aprobados** por feature.

## 1. Principios (confirmados)

- Todo registro financiero pertenece a un único usuario y se consulta siempre filtrado por él.
- Los montos se almacenan con precisión monetaria segura (sin punto flotante binario).
- Toda transacción tiene una clasificación explícita: ingreso, gasto, ahorro o transferencia.
- Una transferencia entre cuentas propias se modela de forma que no afecte los totales de ingresos ni de gastos.

## 2. Dominios de datos (orientativos)

| Dominio | Módulo origen | Descripción |
|---|---|---|
| Usuarios y perfiles | `authentication`, `settings` | Identidad, perfil y preferencias. |
| Cuentas y tarjetas | `accounts` | Cuentas bancarias, efectivo, tarjetas de crédito, etc. |
| Transacciones | `transactions` | Movimientos con clasificación y categoría. |
| Categorías y presupuestos | `budgets` | Categorías de gasto e ingreso; límites por período. |
| Metas de ahorro | `savings-goals` | Objetivos, montos y aportes. |
| Deudas y préstamos | `debts` | Obligaciones, cuotas y saldos. |
| Pagos recurrentes | `recurring-payments` | Reglas de recurrencia y próximos vencimientos. |
| Alertas y predicciones | `predictions-alerts` | Reglas, notificaciones y proyecciones. |
| Documentos | `documents`, `capture-center` | Archivos y sus metadatos. |
| Conversaciones del asistente | `ai-assistant` | Historial (si se persiste). |

## 3. Decisiones resueltas

| ID | Resolución | Fuente |
|---|---|---|
| DP-DB-01 | PostgreSQL (versión estable vigente). | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-DB-02 | `NUMERIC(19,4)`; `decimal.Decimal` en el dominio; redondeo `ROUND_HALF_EVEN`. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-DB-06 | Filtro obligatorio por `user_id` en la capa de repositorio + Row-Level Security como defensa en profundidad. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-DB-07 | Instantes en `timestamptz` (UTC); fechas contables en `DATE` interpretadas en `America/Bogota`. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-DB-03 | Un movimiento por fila, también las transferencias (`account_id` de origen y `destination_account_id`); saldos calculados al leer, sin columnas de saldo. | [ADR-0005](../docs/decisions/ADR-0005-modelo-de-movimientos.md) |

## 4. Decisiones pendientes

| ID | Decisión |
|---|---|
| DP-DB-04 | Borrado lógico vs. físico; retención de datos. |
| DP-DB-05 | Auditoría e historial de cambios de movimientos. |

## 5. Esquemas aprobados

### 5.1 Autenticación (`authentication`)

Aprobado el 29/10/2026 en [`features/authentication/plan.md`](features/authentication/plan.md) §5. Migraciones `0001_auth` y `0002_google`, implementadas (30/10/2026).

- Todos los instantes son `timestamptz` en UTC (DP-DB-07).
- Todas las claves foráneas a `users` tienen `ON DELETE CASCADE`, salvo en `security_events` (RN-AUTH-07).
- Los tokens, códigos e identificadores de sesión se guardan **solo como hash**. El secreto TOTP se guarda cifrado.

| Tabla | Columnas principales | Restricciones / índices |
|---|---|---|
| `users` | `id` (UUIDv7), `email`, `email_verified_at`, `name`, `password_hash` (nullable), `created_at` | Único en `lower(email)`; el correo se guarda normalizado (`email = lower(btrim(email))`); `name` sin espacios en los extremos y de 1 a 100 caracteres |
| `external_identities` | `id`, `user_id`, `provider` (`google`), `subject`, `created_at` | Único en (`provider`, `subject`) |
| `auth_sessions` | `id`, `token_hash` (`bytea`), `user_id`, `stage`, `csrf_token`, `created_at`, `last_activity_at`, `expires_at`, `reauthenticated_at`, `google_reauth_at` (nullable: paso por Google pendiente del código MFA), `pending_google_subject` (nullable) | Único en `token_hash`; índice en `user_id`; `stage` limitado a `active`, `mfa_pending` y `link_pending`; `pending_google_subject` solo y siempre en `link_pending` |
| `auth_tokens` | `id`, `user_id`, `purpose` (`verify_email`, `reset_password`), `token_hash` (`bytea`), `created_at`, `expires_at`, `used_at` | Único en `token_hash`; índice en (`user_id`, `purpose`) |
| `mfa_credentials` | `user_id` (PK), `secret_encrypted` (`bytea`), `confirmed_at` (nullable mientras no se confirma), `last_used_step`, `created_at` | — |
| `mfa_recovery_codes` | `id`, `user_id`, `code_hash`, `used_at` | Índice en `user_id` |
| `oauth_flows` | `id`, `state_hash` (`bytea`), `nonce`, `code_verifier`, `intent` (`login`, `reauth`), `session_id` (nullable, `ON DELETE CASCADE` a `auth_sessions`), `privacy_policy_version` (nullable), `created_at`, `expires_at` | Único en `state_hash`; expira a los 10 minutos |
| `login_attempts` | `id` (`bigint` identidad), `key`, `kind`, `created_at` | Índice en (`key`, `kind`, `created_at`) |
| `data_consents` | `id`, `user_id`, `policy_version`, `accepted_at` | Índice en `user_id` |
| `security_events` | `id` (`bigint` identidad), `user_id` (nullable, **`ON DELETE SET NULL`**), `type`, `ip` (`inet`), `created_at` | Índice en `user_id`, `created_at` |

**RLS (DP-DB-06):** estas tablas no la usan. El inicio de sesión necesita buscar al usuario por su correo antes de saber quién es, y solo el módulo `auth` accede a ellas. RLS empieza con la primera tabla de datos financieros (`accounts`), que tendrá `users.id` como referencia.

**Limpieza:** `purge_expired()` borra tokens, flujos OAuth, intentos y sesiones vencidos.

### 5.2 Cuentas y tarjetas (`accounts`)

Aprobado el 30/10/2026 en [`features/accounts/plan.md`](features/accounts/plan.md) §5. Rol `finora_app` en la migración `0003_app_role` (implementada, 30/10/2026); tabla en `0004_accounts` (implementada, 30/10/2026).

| Tabla | Columnas principales | Restricciones / índices |
|---|---|---|
| `accounts` | `id` (UUIDv7), `user_id` (FK `users`, `ON DELETE CASCADE`), `name`, `type`, `institution` (nullable), `opening_balance` (`numeric(19,4)`), `credit_limit` (`numeric(19,4)`, nullable), `statement_day` y `payment_due_day` (`smallint`, nullable), `archived_at` (nullable), `created_at`, `updated_at` | `type` en `bank`, `wallet`, `cash`, `savings`, `investment`, `credit_card`; campos de tarjeta todos llenos si es `credit_card` y todos vacíos si no; `credit_limit > 0`; días de 1 a 31; `name` de 1 a 60 caracteres sin espacios en los extremos; único en (`user_id`, `lower(name)`), que también sirve para buscar por `user_id` |

**RLS (DP-DB-06, [ADR-0004](../docs/decisions/ADR-0004-row-level-security.md)):** `accounts` es la primera tabla con RLS. Política `accounts_owner` (`USING` y `WITH CHECK`): `user_id = nullif(current_setting('finora.user_id', true), '')::uuid`. La API trabaja con el rol `finora_app` (sin `SUPERUSER` ni `BYPASSRLS`, no dueño de las tablas), que la migración crea si no existe; `finora.user_id` se fija al comenzar cada transacción de una petición autenticada.

### 5.3 Movimientos (`transactions`)

Aprobado el 30/10/2026 en [`features/transactions/plan.md`](features/transactions/plan.md) §5 y [ADR-0005](../docs/decisions/ADR-0005-modelo-de-movimientos.md). Migración `0005_transactions` (implementada, 30/10/2026).

| Tabla | Columnas principales | Restricciones / índices |
|---|---|---|
| `accounts` (cambio) | — | Nueva única (`id`, `user_id`), destino de las FK compuestas de `transactions` |
| `transactions` | `id` (UUIDv7), `user_id` (FK `users`, `ON DELETE CASCADE`), `type`, `amount` (`numeric(19,4)`), `occurred_on` (`date`), `account_id`, `destination_account_id` (nullable), `category` (nullable), `description` (nullable), `note` (nullable), `search_text`, `created_at`, `updated_at` | `type` en `expense`, `income`, `transfer`, `adjustment`; `amount > 0` salvo en `adjustment` (`<> 0`); destino presente si y solo si `transfer`, y distinto del origen; categoría presente si y solo si `expense` o `income`, con las claves de gasto solo en `expense` y las de ingreso solo en `income` (`uncategorized` en ambos); `description` hasta 80 y `note` hasta 500 caracteres; FK compuestas (`account_id`, `user_id`) y (`destination_account_id`, `user_id`) → `accounts` (`id`, `user_id`) `NO ACTION`; índices (`user_id`, `occurred_on DESC`, `created_at DESC`, `id DESC`), `account_id` y `destination_account_id` |

- `search_text`: descripción y nota en minúsculas y sin tildes, normalizadas en Python; se escribe junto con ellas.
- Las FK `NO ACTION` impiden borrar una cuenta con movimientos (RN-TRX-10). Al borrar un usuario, la cascada elimina cuentas y movimientos en la misma sentencia.
- Las FK compuestas garantizan que las cuentas de un movimiento son del mismo usuario que el movimiento.

**RLS (DP-DB-06, [ADR-0004](../docs/decisions/ADR-0004-row-level-security.md)):** política `transactions_owner` (`USING` y `WITH CHECK`), igual a `accounts_owner`.
