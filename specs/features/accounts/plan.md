# Plan técnico: Cuentas y tarjetas

| Campo | Valor |
|---|---|
| Identificador | `FEAT-ACC` |
| Estado | `En revisión` |
| Especificación | [`spec.md`](spec.md) (Aprobado, 30/10/2026) |
| Tareas | [`tasks.md`](tasks.md) |
| Última actualización | 30/10/2026 |

---

## 1. Contexto de la arquitectura actual

- **`finora-back`** (FastAPI, SQLAlchemy 2, Alembic, PostgreSQL 18):
  - `authentication` implementada: sesión de servidor, CSRF y la dependencia `CurrentUser`.
  - Errores RFC 9457 con `code` (`errors.py`).
  - Tipo `Money` (`domain/money.py`): `Decimal`, `ROUND_HALF_EVEN` y solo COP.
  - Migraciones `0001_auth` y `0002_google`.
  - Todavía no hay tablas financieras ni RLS. La API se conecta con un rol superusuario y dueño de las tablas.
- **`finora-web`** (React 19, react-router 8, TanStack Query 5, react-hook-form + zod, CSS Modules con tokens):
  - Pantallas de autenticación sobre `AuthLayout`.
  - Componentes `Button`, `TextField`, `Dialog` y `Alert`, e íconos SVG propios en `components/icons`.
  - `formatMoney` (cadena decimal → texto es-CO, sin `number`).
  - **No existe la estructura de la aplicación** (barra lateral, encabezado, navegación móvil): la página de inicio es provisional.
- **Diseño**: la pantalla "Cuentas y tarjetas" del prototipo (Claude Design) muestra:
  - tres grupos;
  - tarjetas de crédito con deuda, cupo y barra de uso;
  - el botón "Nueva cuenta";
  - la estructura general de navegación (guía `claude-design.md` §3).
- **Contrato**: montos como cadena decimal (DP-API-03).

## 2. Solución propuesta

- **Backend**: módulo nuevo `finora/accounts/`, con este reparto:
  - `models.py`: tabla `accounts`;
  - `schemas.py`: entrada y salida de la API;
  - `service.py`: casos de uso y reglas;
  - `balances.py`: cálculos puros de saldo, totales y uso del cupo, con `Decimal` y probados aparte;
  - endpoints en `api/v1/accounts.py`.
- **Saldo actual = saldo inicial** mientras no exista `transactions` (RN-ACC-02).
  - El servicio expone una sola función, `movement_count(account)`, que hoy devuelve 0.
  - `transactions` la reemplazará. Es el único punto que cambia para RF-ACC-06 y RF-ACC-09.
- **Aislamiento con dos capas**, según [ADR-0004](../../../docs/decisions/ADR-0004-row-level-security.md) (propuesta):
  - todas las consultas filtran por `user_id`;
  - además hay RLS de PostgreSQL, con un rol de aplicación sin privilegios.
- **Frontend**:
  - estructura de la aplicación mínima según el diseño (§3.1), para que "Cuentas" tenga dónde vivir;
  - página `/cuentas` con resumen de totales, grupos, tarjetas, sección de archivadas y diálogos de alta, edición, archivo y eliminación.
- **Totales**: siempre los calcula el backend y vienen en la misma respuesta del listado. El frontend no suma montos (RNF-ACC-02).

### 2.1 Diferencias con la spec

Ninguna. Los detalles que la spec deja abiertos se resuelven así:

- **Porcentaje de uso**: entero, redondeado con `ROUND_HALF_EVEN`. CA-ACC-02: 820.000 ÷ 6.000.000 = 13,67 % → 14 %.
- **Cupo disponible** = cupo − deuda, con mínimo 0. Una deuda negativa (saldo a favor) lo aumenta (caso borde §11).
- **Montos de salida**: cadena con 2 decimales (`"2800000.00"`). La entrada acepta de 0 a 2 decimales (SUP-ACC-02).

## 3. Cambios en el frontend (`finora-web`)

### 3.1 Estructura de la aplicación (`src/app/shell/`)

Es la base de todas las features siguientes; aquí se implementa solo lo necesario.

- **`AppShell`**, layout de las rutas protegidas, con los rangos del diseño (`claude-design.md` §3):
  - `≥ 1200px`: barra lateral de 252px, contraíble a 72px; la preferencia se guarda en `localStorage`;
  - `768–1199px`: barra lateral contraída;
  - `< 768px`: barra inferior.
- **Navegación**: **solo las secciones implementadas**, "Inicio" y "Cuentas". El resto aparece cuando exista su feature.
- **Encabezado**:
  - menú de perfil con "Mi perfil" y "Cerrar sesión" (reutiliza `useLogout`);
  - sin buscador ni alertas, que pertenecen a otras features.
- **Accesibilidad**:
  - enlace "Saltar al contenido";
  - `nav` con `aria-label`;
  - `aria-current="page"` en la sección activa;
  - `main` con el título de la página.
- **Rutas dentro de `AppShell`**: `/` (inicio provisional, ahora sin `AuthLayout`) y `/cuentas`. `/perfil` también pasa a `AppShell` y conserva su contenido.
- **Íconos**: SVG propios en `components/icons`, como en `authentication`, sin la fuente Material Symbols. CD-08 sigue abierta; si se aprueba la fuente, solo cambia ese módulo.

### 3.2 Página Cuentas (`src/features/accounts/`)

| Archivo | Contenido |
|---|---|
| `api.ts` | Funciones de la API y tipos generados (`pnpm api:types`); claves de TanStack Query `['accounts']`. |
| `useAccounts.ts` | Consulta del listado + mutaciones (crear, editar, archivar, reactivar, eliminar) que invalidan `['accounts']`. |
| `AccountsPage.tsx` | Título y subtítulo del diseño ("Saldos registrados de tus cuentas, billeteras y tarjetas."), botón "Nueva cuenta", estados de carga (esqueleto), error con reintento y vacío (RF-ACC-10). |
| `TotalsSummary.tsx` | Saldo disponible, patrimonio neto y deuda en tarjetas, cada uno con su nombre y una breve explicación (RN-ACC-06). |
| `AccountGroup.tsx` / `AccountItem.tsx` | Grupos "Cuentas y efectivo" y "Ahorro e inversión": ícono por tipo, nombre, entidad, tipo y saldo. Menú de acciones por cuenta. |
| `CreditCardItem.tsx` | Deuda, cupo, cupo disponible, `progressbar` de uso con texto ("14 % usado"), "Corte día 28 · pago día 18" y aviso de cupo superado (VAL-ACC-10). |
| `ArchivedSection.tsx` | Cuentas archivadas, plegada por defecto, con "Reactivar" y "Eliminar". |
| `AccountFormDialog.tsx` | Alta y edición en el `Dialog` existente. Tipo con botones de opción en el alta; en la edición se muestra pero no se edita (RN-ACC-08). Campos de tarjeta solo con `credit_card`. Saldo inicial deshabilitado si la cuenta tiene movimientos, con la explicación de VAL-ACC-07. |
| `ConfirmDialog` (en `components/`) | Confirmación reutilizable para archivar (con aviso si el saldo ≠ 0) y eliminar. |
| `MoneyField` (en `components/`) | Campo de monto es-CO: acepta `1.250.000` o `1250000,5` y entrega una cadena decimal (`"1250000.50"`) sin pasar por `number`. Teclado numérico (`inputmode="decimal"`). |
| `parseMoneyInput` (en `lib/format/money.ts`) | Texto es-CO → cadena decimal o error; unitariamente probado. |

- **Formularios**: zod refleja VAL-ACC-01, 03 y 04 solo como ayuda al usuario. Los errores del backend (`422`, `409`) se muestran en el campo con `applyFieldErrors`, que ya existe.
- **Ruta**: `/cuentas`, cargada con `lazy` para no crecer el bundle inicial.

## 4. Cambios en el backend (`finora-back`)

- **`finora/accounts/models.py`**: modelo `Account` (§5). Se registra en `db/models.py`.
- **`finora/accounts/balances.py`**: funciones puras con `Decimal`:
  - `current_balance`;
  - `totals(accounts) -> Totals(available, net_worth, card_debt)` (RN-ACC-03 a 05; solo cuentas activas, RN-ACC-09);
  - `card_status(debt, limit) -> (available_credit, usage_percent, over_limit)`.
- **`finora/accounts/service.py`**:
  - `list_accounts`, `create`, `update`, `archive`, `restore` y `delete`, siempre con el `user_id` de la sesión;
  - nombre único sin distinguir mayúsculas (`409`);
  - máximo 100 cuentas (SUP-ACC-06);
  - `movement_count`, que hoy devuelve 0.
- **`finora/accounts/schemas.py`**:
  - Pydantic con `extra="forbid"`; así, enviar `type` en `PATCH` o campos de tarjeta en otro tipo es un `422` (VAL-ACC-05 y 06);
  - `Amount`: cadena `^-?\d{1,15}(\.\d{1,2})?$` → `Decimal`; los montos en `float` se rechazan;
  - el nombre reutiliza la validación sin caracteres de control de `authentication` (se mueve a un módulo común `finora/validation.py`).
- **`finora/db/rls.py`** (ADR-0004):
  - `SET ROLE finora_app` al conectar;
  - evento `after_begin` que fija `finora.user_id`;
  - dependencia `UserDb` que guarda el `user_id` en `Session.info` y lo fija en la transacción en curso.
- **`api/v1/accounts.py`**: router con los endpoints de §6.

## 5. Cambios en la base de datos

Migración **`0003_accounts`**:

- **Rol `finora_app`** (ADR-0004):
  - `CREATE ROLE ... NOLOGIN NOSUPERUSER NOBYPASSRLS`, si no existe;
  - `GRANT USAGE` en el esquema, `SELECT, INSERT, UPDATE, DELETE` en todas las tablas y `USAGE, SELECT` en las secuencias;
  - `ALTER DEFAULT PRIVILEGES` para las futuras;
  - `GRANT finora_app` al rol que migra (para poder hacer `SET ROLE`).
- **Tabla `accounts`**:

  | Columna | Tipo | Notas |
  |---|---|---|
  | `id` | `uuid` | UUIDv7, PK |
  | `user_id` | `uuid` | FK a `users` `ON DELETE CASCADE` (RN-ACC-11) |
  | `name` | `text` | 1–60 caracteres, sin espacios en los extremos |
  | `type` | `text` | `CHECK` en `bank`, `wallet`, `cash`, `savings`, `investment`, `credit_card` |
  | `institution` | `text` nullable | Hasta 60 caracteres |
  | `opening_balance` | `numeric(19,4)` | En tarjetas, la deuda inicial |
  | `credit_limit` | `numeric(19,4)` nullable | `> 0` |
  | `statement_day` | `smallint` nullable | 1–31 |
  | `payment_due_day` | `smallint` nullable | 1–31 |
  | `archived_at` | `timestamptz` nullable | Vacío = activa |
  | `created_at`, `updated_at` | `timestamptz` | UTC |

- **Restricciones e índices**:
  - `CHECK`: los tres campos de tarjeta van todos llenos si `type = 'credit_card'` y todos vacíos si no;
  - índice único en (`user_id`, `lower(name)`) (VAL-ACC-02);
  - índice en `user_id`.
- **RLS**:
  - `ALTER TABLE accounts ENABLE ROW LEVEL SECURITY`;
  - política `accounts_owner` `USING`/`WITH CHECK` (`user_id = nullif(current_setting('finora.user_id', true), '')::uuid`).
- `specs/database.md` §5 registra el esquema al aprobarse este plan.

## 6. Endpoints y contratos API

Todos exigen sesión `active`. `POST`, `PATCH` y `DELETE` exigen además CSRF y `Origin` (convenciones de `api-contracts.md` §4.1). Una cuenta de otro usuario o inexistente responde `404` (`code: "account_not_found"`).

| Método | Ruta | Descripción | Request | Response | Errores |
|---|---|---|---|---|---|
| GET | `/api/v1/accounts` | Cuentas del usuario (activas y archivadas) y totales | — | `200 {accounts: [Account], totals: {available_balance, net_worth, credit_card_debt}}` | 401 |
| POST | `/api/v1/accounts` | Crear cuenta (RF-ACC-01) | `{name, type, institution?, opening_balance, credit_limit?, statement_day?, payment_due_day?}` | `201 Account` | 401, 403, 409 `account_name_taken`, 409 `account_limit_reached`, 422 |
| GET | `/api/v1/accounts/{id}` | Una cuenta | — | `200 Account` | 401, 404 |
| PATCH | `/api/v1/accounts/{id}` | Editar (RF-ACC-05/06) | Cualquier subconjunto de `{name, institution, opening_balance, credit_limit, statement_day, payment_due_day}` | `200 Account` | 401, 403, 404, 409 `account_name_taken`, 409 `account_has_movements` (saldo inicial), 422 |
| POST | `/api/v1/accounts/{id}/archive` | Archivar (RF-ACC-07); idempotente | `{}` | `200 Account` | 401, 403, 404 |
| POST | `/api/v1/accounts/{id}/restore` | Reactivar (RF-ACC-08); idempotente | `{}` | `200 Account` | 401, 403, 404 |
| DELETE | `/api/v1/accounts/{id}` | Eliminar (RF-ACC-09) | — | `204` | 401, 403, 404, 409 `account_has_movements` |

**`Account`**:

```json
{
  "id": "0193…",
  "name": "Tarjeta Órbita",
  "type": "credit_card",
  "institution": "Banco Andino",
  "opening_balance": "820000.00",
  "balance": "820000.00",
  "has_movements": false,
  "archived": false,
  "credit_card": {
    "credit_limit": "6000000.00",
    "available_credit": "5180000.00",
    "usage_percent": 14,
    "over_limit": false,
    "statement_day": 28,
    "payment_due_day": 18
  },
  "created_at": "2026-10-30T15:00:00Z"
}
```

- `credit_card` es `null` en los demás tipos.
- En tarjetas, `balance` es la deuda (RN-ACC-02).
- Los mensajes en es-CO son los de la spec §10. Los códigos nuevos se agregan a la tabla de `code` del contrato.

## 7. Flujo de datos

1. `GET /accounts`: `CurrentUser` → `UserDb` fija `finora.user_id` → el servicio consulta `WHERE user_id = :id`, y RLS repite el filtro → `balances.totals` calcula los totales → respuesta con cadenas decimales → TanStack Query (`['accounts']`) → `formatMoney` al mostrar.
2. Escrituras: formulario (zod) → `MoneyField` da la cadena decimal → `apiRequest` con CSRF → validación Pydantic + reglas del servicio → `commit` → la mutación invalida `['accounts']` y el listado y los totales se recargan.

## 8. Consideraciones de seguridad

- **Autenticación**: sesión `active`, CSRF y `Origin` en escrituras, como en `authentication`.
- **Aislamiento**:
  - filtro por `user_id` en todas las consultas del servicio;
  - RLS con `finora_app` (ADR-0004);
  - `404` idéntico para una cuenta ajena y una inexistente;
  - el `id` es un UUIDv7, no secuencial;
  - la prueba de RLS consulta sin usuario fijado y espera cero filas.
- **Validación**: montos solo como cadena (nunca `float`); nombre y entidad sin caracteres de control; `extra="forbid"`.
- **Datos sensibles**: no se guardan números de cuenta ni de tarjeta, solo un nombre libre. El formulario aconseja no escribirlos.
- **Abuso**: tope de 100 cuentas por usuario.

## 9. Estrategia de validación

- **Frontend** (ayuda): zod con las mismas reglas de nombre, monto, cupo y días; campos de tarjeta condicionales.
- **Backend** (obligatoria):
  - Pydantic (formato, rangos, campos permitidos por tipo);
  - servicio (unicidad, tope, movimientos);
  - base de datos (`CHECK`, índice único, RLS).

## 10. Manejo de errores

- `422 validation_failed`, con `errors[].message` en es-CO por campo; `409` con `code` específico. El formulario los muestra en el campo correspondiente.
- `404 account_not_found`: el diálogo se cierra, se recarga la lista y se muestra "No encontramos esa cuenta.".
- Error de red o `5xx` en el listado: estado de error con "Reintentar". En un diálogo: alerta dentro del diálogo, conservando lo escrito.
- `401 session_expired`: el flujo existente de `authentication`.

## 11. Estrategia de pruebas

| Nivel | Alcance | Herramienta | Criterios de aceptación cubiertos |
|---|---|---|---|
| Unitarias (back) | `balances.py`: totales, uso del cupo con redondeo, saldo a favor, deuda sobre el cupo, cuentas archivadas excluidas, suma exacta 0,10 + 0,20 | pytest | CA-ACC-01, 02, 03, 07 |
| Integración (back) | Todos los endpoints contra PostgreSQL real: validaciones, unicidad sin mayúsculas, tope, archivar/reactivar/eliminar, aislamiento entre dos usuarios, RLS sin usuario fijado, cascada al eliminar el usuario, CSRF | pytest + Testcontainers | CA-ACC-01 a 08 |
| Unitarias (web) | `parseMoneyInput`, `MoneyField`, formulario por tipo, tarjeta (texto y `progressbar`), totales, estados vacío/error/carga, `AppShell` y su navegación | Vitest + Testing Library | CA-ACC-02, 04, 05, 08 |
| E2E | Crear las cuentas de CA-ACC-01 y comprobar los totales; archivar y reactivar; eliminar | Playwright | CA-ACC-01, 04, 05 |
| Accesibilidad | axe WCAG 2.2 AA en Cuentas y `AppShell`, reflow a 320px, foco visible, navegación por teclado del menú y los diálogos | axe + Playwright | RNF-ACC-04 |

## 12. Dependencias y riesgos

| Riesgo / dependencia | Impacto | Mitigación |
|---|---|---|
| ADR-0004 sin aprobar | Sin RLS real | Se implementa al aprobarse; mientras tanto, nada de esta feature se escribe |
| `CREATEROLE` no disponible en el hosting futuro (DP-ARQ-06) | La migración falla | La migración crea el rol solo si no existe; el rol se puede crear a mano una vez |
| Las pruebas existentes de `authentication` pasan a correr con `finora_app` | Falla si falta un permiso | Los permisos se dan sobre todas las tablas y las suites completas se corren en la tarea de RLS |
| `AppShell` es la base de todas las pantallas futuras | Retrabajo si el diseño cambia | Se sigue el prototipo; revisión visual en QA, como en `authentication` (QA-04) |
| `transactions` no existe | `has_movements` siempre es falso | Una sola función a reemplazar (§2) y pruebas que la simulan |
| CD-08 (íconos) abierta | Cambio de íconos más adelante | Íconos concentrados en `components/icons` |

## 13. Secuencia de implementación

1. RLS y rol `finora_app` (ADR-0004), con las suites de `authentication` en verde.
2. Migración `0003_accounts`, modelo y `balances.py` con sus pruebas unitarias.
3. Servicio, esquemas y endpoints con pruebas de integración; registro en `api-contracts.md`.
4. `pnpm api:types`; `parseMoneyInput`, `MoneyField` y `ConfirmDialog`.
5. `AppShell` y las rutas protegidas dentro de él.
6. Página Cuentas: listado, totales, tarjetas y estados.
7. Diálogos de alta, edición, archivo y eliminación.
8. E2E y accesibilidad; revisión de seguridad; verificación final y documentación.

## 14. Decisiones arquitectónicas

| Decisión | Alternativas consideradas | ADR | Estado |
|---|---|---|---|
| RLS con rol de aplicación `finora_app` vía `SET ROLE` y `finora.user_id` por transacción | RLS sin cambiar de rol (sin efecto), dos URL de conexión, solo filtro en código | [`ADR-0004`](../../../docs/decisions/ADR-0004-row-level-security.md) | Propuesta |
| Estructura de la aplicación (`AppShell`) solo con las secciones implementadas | Mostrar las 15 secciones con "Próximamente" | — (decisión de producto en este plan) | Propuesta |
| Íconos SVG propios en lugar de Material Symbols (CD-08 sigue abierta) | Fuente Material Symbols desde Google Fonts | — | Propuesta |
| Totales en la misma respuesta del listado | Endpoint aparte `/accounts/summary` | — | Propuesta |

## 15. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 30/10/2026 | Plan completo a partir de la spec aprobada; propone ADR-0004 (RLS). Pasa a `En revisión`. | Claude Code |
