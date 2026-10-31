# Plan técnico: Transacciones

| Campo | Valor |
|---|---|
| Identificador | `FEAT-TRX` |
| Estado | `Aprobado` |
| Especificación | [`spec.md`](spec.md) (`Aprobado`, 30/10/2026) |
| Tareas | [`tasks.md`](tasks.md) |
| Última actualización | 30/10/2026 |

> Plan contra el estado real del repositorio: stack de [ADR-0001](../../../docs/decisions/ADR-0001-stack-tecnologico.md), RLS de [ADR-0004](../../../docs/decisions/ADR-0004-row-level-security.md) y la implementación de `accounts`.

---

## 1. Contexto de la arquitectura actual

- **`finora-back`**:
  - FastAPI + SQLAlchemy 2 + Alembic + PostgreSQL 18.
  - `accounts` implementada: modelo `Account`, `balances.py` (funciones puras con `Money`), servicio, endpoints `/api/v1/accounts` y RLS con el rol `finora_app`.
  - `movement_count` devuelve 0 y el saldo actual es igual al saldo inicial.
  - Patrones reutilizables: `CurrentUser` + `UserDb`, `ApiProblemError` (RFC 9457 con `code`), montos `Amount`/`ApiAmount`, `validation.py` (caracteres de control), `ClockDep`.
- **`finora-web`**:
  - React 19 + TanStack Query + react-hook-form + zod.
  - `AppShell` con navegación ("Inicio", "Cuentas").
  - Componentes: `Dialog`, `ConfirmDialog`, `MoneyField`, `ActionsMenu`, `PageHeader`, `TextField`, `Alert`.
  - Formato: `formatMoney`, `parseMoneyInput`, `toMoneyInput`, `formatDate`, `ACCOUNTING_TIME_ZONE`.
  - Página `/cuentas` con su formulario y su menú de acciones.
- **Pendiente:** DP-DB-03 (modelo de transferencias) en [`database.md`](../../database.md). Este plan lo resuelve con [ADR-0005](../../../docs/decisions/ADR-0005-modelo-de-movimientos.md).

## 2. Solución propuesta

- **Un movimiento = una fila** en `transactions`, también las transferencias, que llevan cuenta de origen y de destino (ADR-0005). Una transferencia nunca queda a medias y su clasificación (ahorro, pago de tarjeta, entre cuentas líquidas) se deduce del tipo de sus dos cuentas.
- **Los saldos se calculan al leer**, sin columnas de saldo guardadas:
  - saldo actual = saldo inicial + efecto de sus movimientos (RN-TRX-06);
  - una consulta agregada por usuario (≤ 100 cuentas, ~10.000 movimientos) devuelve el efecto por cuenta;
  - no hay saldos que se desincronicen al editar o eliminar.
- **Una regla de efecto única** (`transactions/effects.py`), probada unitariamente y reflejada en SQL (`CASE`):

  | Tipo | Cuenta que no es tarjeta | Tarjeta (saldo = deuda) |
  |---|---|---|
  | `expense` en la cuenta | − monto | + monto |
  | `income` en la cuenta | + monto | − monto |
  | `transfer`, cuenta de origen | − monto | + monto |
  | `transfer`, cuenta de destino | + monto | − monto |
  | `adjustment` | + monto firmado | + monto firmado |

- **Ajuste**: la API recibe `target_balance` (saldo real). El servicio calcula `amount = target_balance − saldo actual sin este ajuste`; si da 0, responde VAL-TRX-09.
- **Resumen** (RN-TRX-05): una consulta agregada sobre todos los movimientos que cumplen los filtros, con `CASE` según el tipo y el tipo de las cuentas.
- **Búsqueda sin tildes ni mayúsculas** sin extensiones de PostgreSQL:
  - columna `search_text` con la descripción y la nota normalizadas en Python (minúsculas, sin tildes);
  - las coincidencias con nombres de cuenta y de categoría se resuelven en Python (son pocas) y se agregan como `account_id IN (…)` / `category IN (…)`.

## 3. Cambios en el frontend (`finora-web`)

### 3.1 Navegación y rutas

- `NAV_ITEMS`: "Movimientos" entre "Inicio" y "Cuentas", con un ícono nuevo (`TransactionsIcon`, flechas en sentidos opuestos).
- Ruta `/movimientos` (`lazy`). Los filtros viven en la URL (`?q=&desde=&hasta=&tipo=&categoria=&cuenta=&orden=`), así que el botón Atrás y los enlaces conservan la búsqueda.

### 3.2 Página Movimientos (`src/features/transactions/`)

| Archivo | Contenido |
|---|---|
| `api.ts` | Funciones de la API y tipos generados; claves `['transactions', filtros]`. |
| `categories.ts` | Catálogo de RN-TRX-07: clave, nombre es-CO, tipo (gasto/ingreso) y color (`--c-*` del diseño). |
| `periods.ts` | "Este mes", "Mes anterior", "Últimos 3 meses" y "Todo el historial" → `from`/`to` en hora de Colombia. Probado con fechas límite. |
| `useTransactions.ts` | `useInfiniteQuery` de 50 en 50 ("Cargar más") y mutaciones que invalidan `['transactions']` y `['accounts']`. |
| `TransactionsPage.tsx` | Título "Movimientos", botón "Añadir movimiento", filtros, resumen y lista; estados de carga, error con reintento, vacío y sin resultados (RF-TRX-09). |
| `TransactionFilters.tsx` | Búsqueda, periodo, categoría, cuenta, orden y tipo (grupo de botones con `aria-pressed`); "Limpiar filtros". |
| `TransactionSummary.tsx` | Cantidad, ingresos, gastos, ahorro neto y transferencias ("no suman como gasto"), con `role="status"`. |
| `TransactionList.tsx` | Tabla en escritorio (fecha, descripción, categoría, cuenta, monto, acciones) y lista en móvil, como el diseño. Monto con signo y texto, no solo color (RNF-TRX-04). |
| `TransactionFormDialog.tsx` | Modal del diseño, solo la forma manual: tipo (Gasto, Ingreso, Transferencia, Ajuste), monto grande con `MoneyField`, fecha (`type="date"` con máximo hoy), cuenta, destino (transferencias), categoría según el tipo, descripción y nota. En ajustes, "Saldo real de la cuenta" con el saldo actual como ayuda. Aviso del diseño en transferencias ("no se cuentan como gasto ni ingreso…"). Solo ofrece cuentas activas, y la propia si ya estaba archivada. |
| Eliminar | `ConfirmDialog` ("¿Eliminar este movimiento?"), desde el `ActionsMenu` de cada fila (Editar, Eliminar). |

### 3.3 Cambios en Cuentas

- `AccountActions`: nueva acción "Ajustar saldo" en cuentas activas, que abre `TransactionFormDialog` con el tipo ajuste y la cuenta elegidos.
- El formulario de cuenta, con saldo bloqueado (VAL-ACC-07), enlaza a esa acción.
- Las mutaciones de movimientos invalidan `['accounts']`, así que saldos y totales se recargan del backend.

### 3.4 Referencia en Claude Design

Pantalla `movements` y modal de movimiento (pestaña "Manual"). No se trasladan: pestañas de texto, voz, recibo, importación y sincronización (fuera de alcance); "Opciones avanzadas" (hora, método, recurrencia, estado, cuotas, etiquetas, comprobante); "Exportar CSV" y aviso de duplicado.

## 4. Cambios en el backend (`finora-back`)

- **`finora/transactions/models.py`**: `TransactionType` y `Category` (`StrEnum`) y modelo `Transaction` (§5); se registra en `db/models.py`.
- **`finora/transactions/categories.py`**: catálogo con clave, nombre es-CO y tipo; valida que la categoría corresponda al tipo.
- **`finora/transactions/effects.py`**: `effect(type, amount, role, account_type) -> Decimal` (tabla de §2) y `classify_transfer(origin_type, destination_type)` (ahorro, retiro de ahorro u otra transferencia). Funciones puras.
- **`finora/transactions/schemas.py`**:
  - `TransactionRequest`, con discriminador por `type`: campos permitidos por tipo y `extra="forbid"` (VAL-TRX-11);
  - `Amount` positivo de `accounts`; `target_balance` en ajustes;
  - `date` ISO; descripción de 1 a 80 caracteres y nota de hasta 500, sin caracteres de control (la nota admite saltos de línea);
  - respuestas `TransactionResponse`, `TransactionPage` y `Summary`.
- **`finora/transactions/service.py`**:
  - `list_transactions(filtros, orden, cursor)` → página + total + resumen;
  - `get`, `create`, `replace` y `delete`;
  - valida en el servicio:
    - cuentas propias (VAL-TRX-03) y activas (VAL-TRX-04, RN-TRX-08);
    - fecha entre el 01/01/2000 y hoy en Colombia (VAL-TRX-02, con `ClockDep`);
    - destino distinto del origen (VAL-TRX-05);
    - categoría según el tipo (VAL-TRX-06);
    - ajuste distinto de 0 (VAL-TRX-09).
- **`finora/transactions/search.py`**: `normalize(text)` (minúsculas, sin tildes con `unicodedata`), que se usa al guardar `search_text` y al buscar.
- **`finora/accounts`**:
  - `service.movement_count`: cuenta los movimientos donde la cuenta es origen o destino;
  - nueva consulta `effects_by_account(db, user_id)`;
  - `balances.current_balance(opening, effect)`.
  - La respuesta de `/accounts` no cambia de forma: `balance` y `has_movements` pasan a ser reales.
- **`api/v1/transactions.py`**: router de §6 con `CurrentUser`, `UserDb`, Origin y CSRF en escrituras, y `Cache-Control: no-store` (se suma a `NO_STORE_PREFIXES`).

## 5. Cambios en la base de datos

Migración **`0005_transactions`**:

- **`accounts`**: restricción única nueva (`id`, `user_id`), para que las claves foráneas compuestas garanticen que las cuentas de un movimiento son del mismo usuario.
- **Tabla `transactions`**:

  | Columna | Tipo | Notas |
  |---|---|---|
  | `id` | `uuid` | UUIDv7, PK |
  | `user_id` | `uuid` | FK a `users` `ON DELETE CASCADE` |
  | `type` | `text` | `CHECK` en `expense`, `income`, `transfer`, `adjustment` |
  | `amount` | `numeric(19,4)` | `> 0`; en `adjustment`, `<> 0` |
  | `occurred_on` | `date` | Fecha contable (DP-DB-07) |
  | `account_id` | `uuid` | FK compuesta (`account_id`, `user_id`) → `accounts` (`id`, `user_id`), `NO ACTION` |
  | `destination_account_id` | `uuid` nullable | Igual que `account_id`; solo `transfer` |
  | `category` | `text` nullable | `CHECK` con las 13 claves; solo `expense` e `income` |
  | `description` | `text` nullable | Hasta 80 caracteres |
  | `note` | `text` nullable | Hasta 500 caracteres |
  | `search_text` | `text` | Descripción y nota normalizadas |
  | `created_at`, `updated_at` | `timestamptz` | UTC |

- **Restricciones (`CHECK`)**:
  - `transfer` si y solo si hay destino, y destino ≠ origen;
  - categoría presente si y solo si el tipo es `expense` o `income`;
  - categorías de gasto solo en `expense` y de ingreso solo en `income` (`uncategorized` en ambos).
- **Borrado**:
  - Las FK a `accounts` son `NO ACTION`: la base de datos impide borrar una cuenta con movimientos, además del servicio (RN-TRX-10).
  - Al borrar un usuario se eliminan en la misma sentencia sus cuentas y sus movimientos, y la comprobación al final de la sentencia pasa.
- **Índices**:
  - (`user_id`, `occurred_on DESC`, `created_at DESC`, `id`) para el listado;
  - `account_id` y `destination_account_id` para saldos y `has_movements`.
- **RLS**: `ENABLE ROW LEVEL SECURITY` y política `transactions_owner`, como `accounts_owner`.
- `specs/database.md` §5 registra el esquema y DP-DB-03 pasa a resuelto (ADR-0005).

## 6. Endpoints y contratos API

Todos exigen sesión `active`; `POST`, `PUT` y `DELETE` exigen además CSRF y `Origin`. Un movimiento de otro usuario o inexistente responde `404` (`code: "transaction_not_found"`).

| Método | Ruta | Descripción | Request | Response | Errores |
|---|---|---|---|---|---|
| GET | `/api/v1/transactions` | Listado filtrado y paginado con resumen | Query: `q`, `from`, `to`, `type`, `category`, `account_id`, `sort` (`date_desc` por defecto, `date_asc`, `amount_desc`, `amount_asc`), `cursor` (opaco), `limit` (50, máx. 100) | `200 {items: [Transaction], total, next_cursor, summary: {count, income, expense, savings, transfers}}` | 401, 422 |
| POST | `/api/v1/transactions` | Crear (RF-TRX-01 a 03) | `TransactionRequest` | `201 Transaction` | 401, 403, 422 |
| GET | `/api/v1/transactions/{id}` | Un movimiento | — | `200 Transaction` | 401, 404 |
| PUT | `/api/v1/transactions/{id}` | Reemplazar (RF-TRX-07; el tipo puede cambiar) | `TransactionRequest` | `200 Transaction` | 401, 403, 404, 422 |
| DELETE | `/api/v1/transactions/{id}` | Eliminar (RF-TRX-08) | `{}` | `204` | 401, 403, 404 |

**`TransactionRequest`** (según `type`):

- `expense` / `income`: `{type, amount, date, account_id, category, description, note?}`
- `transfer`: `{type, amount, date, account_id, destination_account_id, description?, note?}`
- `adjustment`: `{type, target_balance, date, account_id, description?, note?}`

**`Transaction`**:

```json
{
  "id": "0193…",
  "type": "transfer",
  "amount": "820000.00",
  "date": "2026-10-30",
  "account_id": "0193…",
  "destination_account_id": "0193…",
  "category": null,
  "description": null,
  "note": null,
  "created_at": "2026-10-30T15:00:00Z",
  "updated_at": "2026-10-30T15:00:00Z"
}
```

- `amount` es positivo, salvo en ajustes (diferencia firmada).
- Los nombres de las cuentas no se repiten: el frontend ya tiene `/accounts` en caché.
- **Se usa `PUT` en lugar de `PATCH`**: al cambiar el tipo cambian los campos obligatorios, y reemplazar el movimiento completo evita estados intermedios inválidos.
- Códigos nuevos: `transaction_not_found` (404). Los errores por campo usan `validation_failed` con los mensajes de la spec §10.

## 7. Flujo de datos

1. **Lectura:** filtros en la URL → `GET /transactions` → `UserDb` fija `finora.user_id` → el servicio consulta con `user_id` + RLS, la página con `ORDER BY … LIMIT` a partir del cursor (keyset), `count` y el resumen en la misma transacción → cadenas decimales → TanStack Query (`useInfiniteQuery`) → `formatMoney` y `formatDate` al mostrar.
2. **Escritura:**
   - formulario (zod como ayuda) → `MoneyField` → `POST`/`PUT` con CSRF;
   - Pydantic (formato y campos por tipo) → servicio (reglas) → `commit`;
   - se invalidan `['transactions']` y `['accounts']`, así que saldos y totales vienen del backend.
3. **Saldos de cuentas:** `GET /accounts` → `effects_by_account` (una consulta agregada) → `current_balance` → `totals` (sin cambios en las reglas de `accounts`).

## 8. Consideraciones de seguridad

- **Autenticación**: sesión `active`, CSRF y `Origin` en escrituras.
- **Aislamiento**:
  - filtro por `user_id` en todas las consultas;
  - RLS `transactions_owner`;
  - las FK compuestas impiden en la base de datos que un movimiento use una cuenta de otro usuario;
  - una cuenta ajena en la petición responde como inexistente (VAL-TRX-03), sin revelar que existe.
- **Validación**: montos solo como cadena; `extra="forbid"`; textos sin caracteres de control; `limit` máximo 100; `q` de hasta 100 caracteres. La búsqueda va como parámetro, nunca concatenada en SQL.
- **Datos sensibles**: descripciones y notas son texto libre del usuario; no se registran en logs.

## 9. Estrategia de validación

- **Frontend** (ayuda):
  - zod con monto positivo (`parseMoneyInput`), fecha hasta hoy, campos según el tipo, destino distinto, categoría del tipo y longitudes;
  - las listas solo ofrecen cuentas activas y categorías del tipo.
- **Backend** (obligatoria):
  - Pydantic (formato y campos por tipo);
  - servicio (propiedad y estado de las cuentas, fechas en hora de Colombia, ajuste distinto de 0);
  - base de datos (`CHECK`, FK compuestas, RLS).

## 10. Manejo de errores

- `422 validation_failed` con `errors[].message` por campo; el formulario lo muestra en su campo (`applyFieldErrors`, con nombres de campo de la API → formulario, como en `AccountFormDialog`).
- `404 transaction_not_found`: el diálogo se cierra, se recarga la lista y se avisa "No encontramos ese movimiento.".
- Error de red o `5xx` en el listado: "No pudimos cargar tus movimientos" con "Reintentar" (texto del diseño). En un diálogo: alerta dentro, conservando lo escrito.
- Filtros inválidos en la URL (p. ej. una fecha mal escrita): se ignoran y se usa el valor por defecto, sin error.

## 11. Estrategia de pruebas

| Nivel | Alcance | Herramienta | Criterios de aceptación cubiertos |
|---|---|---|---|
| Unitarias (back) | `effects.py` (los 5 casos × tarjeta / no tarjeta), clasificación de transferencias, `normalize`, catálogo de categorías, esquemas por tipo | pytest | CA-TRX-01 a 06, 12 |
| Integración (back) | Endpoints contra PostgreSQL real: los criterios con las cuentas de CA-ACC-01 y sus totales, edición y borrado, cuentas archivadas, fechas, aislamiento, RLS sin usuario, FK compuestas, borrado de cuenta con movimientos (servicio y base de datos), cascada de usuario, filtros, orden, paginación y resumen | pytest + Testcontainers | CA-TRX-01 a 12 |
| Unitarias (web) | `periods.ts`, catálogo, formulario por tipo, filtros en la URL, lista, resumen, estados, "Cargar más", eliminar y "Ajustar saldo" desde Cuentas | Vitest + Testing Library | CA-TRX-07, 09, 10 |
| E2E | CA-TRX-01, 03, 04 y 05 de punta a punta con los totales de Cuentas | Playwright | CA-TRX-01, 03, 04, 05 |
| Accesibilidad | axe WCAG 2.2 AA en Movimientos y el modal, reflow a 320 px, teclado | axe + Playwright | RNF-TRX-04 |
| Rendimiento | Listado y `/accounts` con 10.000 movimientos | pytest (integración) | RNF-TRX-06 |

## 12. Dependencias y riesgos

| Riesgo / dependencia | Impacto | Mitigación |
|---|---|---|
| Saldos calculados al leer | Lentitud con muchos movimientos | Consulta agregada con índices por cuenta; prueba de rendimiento con 10.000 (RNF-TRX-06); si no alcanza, se evalúa guardar saldos (ADR nuevo) |
| `accounts` cambia de "sin movimientos" a saldos reales | Fallan pruebas de `accounts` que asumían saldo = inicial | Las pruebas existentes no crean movimientos y siguen válidas; se agregan casos con movimientos |
| `accounts` aún sin QA | Hallazgos de QA pueden tocar código compartido | La QA de `accounts` se puede hacer antes o junto con la de esta feature |
| Catálogo fijo de categorías | `budgets` lo reemplazará | Categoría como `text` con `CHECK`; migrar a una tabla será una migración de `budgets` |
| `<input type="date">` muestra el formato del navegador | Fuera de DD/MM/AAAA en navegadores en otro idioma | `lang="es-CO"` en el documento; la lista y el resumen usan `formatDate`. Se revisa en QA visual |
| Cambio de tipo en la edición | Reglas distintas por tipo | `PUT` con validación completa del nuevo estado |

## 13. Secuencia de implementación

1. Migración `0005_transactions`, modelo, catálogo y `effects.py` con sus pruebas.
2. Registro de endpoints en `api-contracts.md`; servicio y esquemas con pruebas de integración.
3. Saldos reales en `accounts` (`effects_by_account`, `movement_count`).
4. Endpoints `/transactions` con pruebas por HTTP.
5. `pnpm api:types`; catálogo, periodos y navegación.
6. Página Movimientos: filtros, resumen, lista y estados.
7. Formulario de movimiento, eliminar y "Ajustar saldo" desde Cuentas.
8. E2E, accesibilidad, rendimiento, revisión de seguridad y documentación.

## 14. Decisiones arquitectónicas

| Decisión | Alternativas consideradas | ADR | Estado |
|---|---|---|---|
| Un movimiento por fila, también las transferencias (origen y destino) | Dos movimientos enlazados por transferencia | [`ADR-0005`](../../../docs/decisions/ADR-0005-modelo-de-movimientos.md) | Aprobada (30/10/2026) |
| Saldos calculados al leer, sin columnas de saldo | Saldo guardado en `accounts` y actualizado en cada escritura | [`ADR-0005`](../../../docs/decisions/ADR-0005-modelo-de-movimientos.md) | Aprobada (30/10/2026) |
| FK compuestas (`account_id`, `user_id`) además de RLS | Solo filtro en el servicio y RLS | — | Aprobada (30/10/2026) |
| Búsqueda con `search_text` normalizado en Python | Extensión `unaccent` de PostgreSQL | — | Aprobada (30/10/2026) |
| `PUT` para editar movimientos | `PATCH` parcial | — | Aprobada (30/10/2026) |
| Paginación por cursor opaco (keyset por la clave de orden + `created_at` + `id`), según DP-API-06 | `offset`/`limit` (versión inicial de este plan, contraria a ADR-0001) | [`ADR-0001`](../../../docs/decisions/ADR-0001-stack-tecnologico.md) | Aprobada (30/10/2026) |
| Filtros por `from`/`to` en la API (los periodos se calculan en el frontend) | Periodos con nombre en la API | — | Aprobada (30/10/2026) |

## 15. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 30/10/2026 | Plan completo a partir de la spec aprobada; propone ADR-0005 (modelo de movimientos). Pasa a `En revisión`. | Claude Code |
| 30/10/2026 | Aprobado por la usuaria, con las seis decisiones de §14 (ADR-0005 pasa a `Aceptada`). | Claude Code |
| 30/10/2026 | T-TRX-INT-01: la paginación pasa de `offset` a cursor opaco para cumplir DP-API-06 (ADR-0001), que el plan aprobado contradecía. El orden por monto usa el valor absoluto (los ajustes pueden ser negativos). Pendiente de confirmación de la usuaria. | Claude Code |
| 30/10/2026 | La usuaria confirma la paginación por cursor. | Claude Code |
