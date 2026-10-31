# Tareas: Cuentas y tarjetas

| Campo | Valor |
|---|---|
| Identificador | `FEAT-ACC` |
| Estado | `En progreso` |
| Especificación | [`spec.md`](spec.md) (Aprobado, 30/10/2026) |
| Plan técnico | [`plan.md`](plan.md) (Aprobado, 30/10/2026) |
| Última actualización | 30/10/2026 |

> Reglas:
> - Solo se listan tareas derivadas de un plan **aprobado**.
> - Una tarea se marca `[x]` **solo** después de ejecutar su verificación con resultado exitoso.
> - Aplicación: `finora-web` | `finora-back` | `shared` | `infraestructura`.
> - Prioridad: `Alta` | `Media` | `Baja`.
> - Los comandos de `finora-back` se ejecutan en esa carpeta y requieren Docker. Los de `finora-web`, en la suya.

Leyenda de estado:

- [ ] Pendiente
- [x] Completada

Orden sugerido: INT-01 → DB-01 → DB-02 → BE-01 a BE-03 → FE-01 a FE-05 → QA-01 a QA-04 → INT-02.

---

## Integración

- [x] **T-ACC-INT-01**: Registrar los 7 endpoints de `plan.md` §6 en `specs/api-contracts.md` (§3 y §5, estado `Aprobado`, sin implementar) y la tabla de `plan.md` §5 en `specs/database.md` §5.2.
  - **Aplicación:** `shared`
  - **Dependencias:** Plan aprobado
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Endpoints, modelo `Account`, códigos de error y tabla coinciden con `plan.md` §5 y §6
  - **Verificación:** Revisión manual de documentos

## Base de datos

- [x] **T-ACC-DB-01**: RLS según ADR-0004. Migración `0003_app_role` que crea el rol `finora_app` si no existe, con permisos sobre todas las tablas y secuencias y privilegios por defecto para las futuras. `db/rls.py`: `SET ROLE finora_app` al conectar el motor de la API (no el de Alembic), evento `after_begin` que fija `finora.user_id` desde `Session.info` y dependencia `UserDb`. Las pruebas usan el mismo motor con `SET ROLE`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-ACC-INT-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** La API trabaja como `finora_app` (no superusuario); `finora.user_id` se fija en cada transacción, también después de un `commit`, y no pasa a otra petición; toda la suite de `authentication` sigue en verde
  - **Verificación:** `uv run pytest tests/integration/test_rls.py && uv run pytest && uv run mypy .`

- [x] **T-ACC-DB-02**: Migración `0004_accounts`: tabla `accounts` con sus `CHECK`, índice único en (`user_id`, `lower(name)`) (sirve también para buscar por usuario), `ENABLE ROW LEVEL SECURITY` y política `accounts_owner`. Modelo `Account` registrado en `db/models.py`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-ACC-DB-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** `upgrade` y `downgrade` limpios; sin `finora.user_id` no se ve ni se inserta ninguna fila; un usuario no ve filas de otro aunque la consulta no filtre; borrar un usuario borra sus cuentas (RN-ACC-11); el modelo coincide con la migración (`alembic check`)
  - **Verificación:** `uv run pytest tests/integration/test_accounts_schema.py tests/integration/test_accounts_rls.py && uv run alembic check`

## Backend (`finora-back`)

- [x] **T-ACC-BE-01**: `accounts/balances.py`: saldo actual, totales (saldo disponible, patrimonio neto, deuda en tarjetas; solo cuentas activas) y estado de la tarjeta (cupo disponible, % de uso con `ROUND_HALF_EVEN`, cupo superado, saldo a favor), todo con `Decimal`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** —
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RN-ACC-02 a 05 y 09; CA-ACC-01, 02, 03 y 07 a nivel de cálculo; casos borde de §11 (deuda negativa, deuda sobre el cupo, saldos negativos, sin cuentas)
  - **Verificación:** `uv run pytest tests/unit/test_account_balances.py`

- [x] **T-ACC-BE-02**: `accounts/schemas.py` y `accounts/service.py`: crear, listar, consultar, editar, archivar, reactivar y eliminar, siempre con el `user_id` de la sesión; nombre único sin distinguir mayúsculas; tope de 100 cuentas; campos de tarjeta solo en `credit_card`; `movement_count` (hoy 0) para RF-ACC-06 y 09. La validación de texto sin caracteres de control se mueve a `finora/validation.py` y la reutiliza `authentication`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-ACC-DB-02, T-ACC-BE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-ACC-01 y 05 a 09; RN-ACC-08 a 10; VAL-ACC-01 a 08; montos solo como cadena con hasta 2 decimales (SUP-ACC-02)
  - **Verificación:** `uv run pytest tests/unit/test_account_schemas.py tests/integration/test_accounts_service.py`

- [x] **T-ACC-BE-03**: `api/v1/accounts.py`: los 7 endpoints del contrato §5 con `CurrentUser` + `UserDb`, CSRF en escrituras, `404 account_not_found` idéntico para cuentas ajenas e inexistentes y errores RFC 9457 con los `code` del contrato.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-ACC-BE-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-ACC-01 a 08 vía HTTP; respuestas idénticas a `api-contracts.md` §5; sin sesión `401`, sin CSRF `403`
  - **Verificación:** `uv run pytest tests/integration/test_accounts_api.py && uv run ruff check . && uv run mypy .`

## Frontend (`finora-web`)

- [ ] **T-ACC-FE-01**: Base compartida: `pnpm api:types`; `parseMoneyInput` en `lib/format/money.ts` (texto es-CO → cadena decimal sin `number`); componentes `MoneyField` y `ConfirmDialog`; íconos SVG de los 6 tipos de cuenta y de la navegación.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-BE-03
  - **Prioridad:** Alta
  - **Criterios de aceptación:** `1.250.000`, `1250000,5` y `$ 1.250.000` se convierten exactamente; texto inválido o con más de 2 decimales da error; los componentes pasan axe
  - **Verificación:** `pnpm test src/lib/format src/components && pnpm typecheck`

- [ ] **T-ACC-FE-02**: `AppShell` (`src/app/shell/`): barra lateral de escritorio (252px, contraíble a 72px, preferencia en `localStorage`), barra lateral contraída en tablet, barra inferior en móvil, encabezado con menú de perfil ("Mi perfil", "Cerrar sesión"), "Saltar al contenido", `aria-current`. Solo "Inicio" y "Cuentas" en la navegación. `/`, `/cuentas` y `/perfil` dentro de `AppShell`.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-FE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Rangos de `claude-design.md` §3; navegación por teclado; axe sin violaciones; las pruebas de `authentication` siguen en verde
  - **Verificación:** `pnpm test src/app && pnpm test && pnpm lint && pnpm typecheck`

- [ ] **T-ACC-FE-03**: Página `/cuentas` (`lazy`): resumen de totales con su nombre y explicación, grupos "Cuentas y efectivo" y "Ahorro e inversión", tarjetas con deuda, cupo, cupo disponible, `progressbar` de uso, corte y pago y aviso de cupo superado; sección de archivadas plegada; estados de carga, error con reintento y vacío.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-FE-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-ACC-02 a 04, 08 y 10; RN-ACC-06; CA-ACC-02 en pantalla; el frontend no suma montos
  - **Verificación:** `pnpm test src/features/accounts`

- [ ] **T-ACC-FE-04**: `AccountFormDialog` para alta y edición: tipo con botones de opción (fijo en la edición), campos de tarjeta solo con `credit_card`, saldo inicial deshabilitado si hay movimientos, zod como ayuda y errores `422`/`409` del backend en su campo.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-FE-03
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-ACC-01, 05 y 06; CA-ACC-08; VAL-ACC-01 a 04 y 07 con sus mensajes
  - **Verificación:** `pnpm test src/features/accounts`

- [ ] **T-ACC-FE-05**: Archivar (con aviso si el saldo ≠ 0), reactivar y eliminar con `ConfirmDialog`; ante `409 account_has_movements` se ofrece archivar; `404` cierra el diálogo y recarga.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-FE-04
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-ACC-07 a 09; CA-ACC-04 y 05; VAL-ACC-08 y 09
  - **Verificación:** `pnpm test src/features/accounts && pnpm lint && pnpm typecheck && pnpm build`

## Pruebas

- [ ] **T-ACC-QA-01**: E2E (Playwright): crear las 4 cuentas de CA-ACC-01 y comprobar totales y datos de la tarjeta; archivar y reactivar; eliminar; un segundo usuario no ve las cuentas del primero.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-FE-05
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-ACC-01, 02, 04, 05 y 06 en el navegador real
  - **Verificación:** `pnpm test:e2e`

- [ ] **T-ACC-QA-02**: Accesibilidad: axe WCAG 2.2 AA (con contraste) en `AppShell` y Cuentas en los tres rangos, reflow a 320px, foco visible y navegación por teclado del menú y los diálogos.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-QA-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RNF-ACC-04
  - **Verificación:** `pnpm test:e2e e2e/accessibility.spec.ts`

- [ ] **T-ACC-QA-03**: Revisión de seguridad de la feature (aislamiento, RLS, validación de montos) y corrección de los hallazgos confirmados.
  - **Aplicación:** `finora-web` / `finora-back`
  - **Dependencias:** T-ACC-QA-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Sin hallazgos confirmados abiertos de severidad alta o media
  - **Verificación:** Informe de la revisión y suites de ambas aplicaciones en verde

- [ ] **T-ACC-QA-04**: Revisión visual de `AppShell` y Cuentas contra Claude Design con la responsable de producto, en escritorio, tablet y móvil.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-ACC-QA-02
  - **Prioridad:** Media
  - **Criterios de aceptación:** Aprobación de producto; diferencias registradas y corregidas o aceptadas
  - **Verificación:** Revisión manual con capturas

## Cierre

- [ ] **T-ACC-INT-02**: Endpoints a `Implementado` en `api-contracts.md`, esquema implementado en `database.md`, README de ambas aplicaciones (rol `finora_app`, RLS) y estado de la feature.
  - **Aplicación:** `shared`
  - **Dependencias:** T-ACC-QA-03, T-ACC-QA-04
  - **Prioridad:** Media
  - **Criterios de aceptación:** Los comandos documentados funcionan tal como están escritos
  - **Verificación:** Revisión manual de documentos y suites completas de ambas aplicaciones

---

## Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 30/10/2026 | Tareas a partir del plan aprobado. Completada INT-01: 7 endpoints y modelo `Account` en `api-contracts.md` (§3 y §5) y tabla `accounts` con RLS en `database.md` §5.2. | Claude Code |
| 30/10/2026 | Completada DB-01: migración `0003_app_role`, `db/rls.py` (`SET ROLE` al conectar, `after_begin` con `set_config(..., true)`, `bind_user`/`unbind_user`) y dependencia `UserDb`; la API y las pruebas trabajan como `finora_app`. `tests/integration/test_rls.py` (7): rol sin privilegios y no dueño, sin acceso a `alembic_version`, usuario conservado tras `commit` y no heredado por otra sesión del pool, `UserDb`, política que falla cerrado y migración que baja y sube. Verificado: backend 402/402 (ruff, mypy), E2E 17/17 contra el backend con el rol nuevo y migración aplicada a la base de desarrollo. | Claude Code |
| 30/10/2026 | Completada DB-02: modelo `Account` (`finora/accounts/models.py`) y migración `0004_accounts` con `CHECK`, índice único (`user_id`, `lower(name)`), RLS y política `accounts_owner`. Pruebas: `test_accounts_schema.py` (permisos de `finora_app`, RLS activa, dueño distinto, migración que baja y sube, restricciones de tipo, nombre, entidad, unicidad, datos de tarjeta, 4 decimales, saldos cero y negativos) y `test_accounts_rls.py` (cada usuario ve solo lo suyo sin `WHERE`, sin usuario no ve nada, no crea ni traspasa cuentas a otro, `UPDATE`/`DELETE` sin filtro no tocan cuentas ajenas, la cascada al borrar el usuario funciona sin usuario fijado). Verificado: backend 433/433, `alembic check` sin diferencias, migración aplicada a la base de desarrollo. | Claude Code |
| 30/10/2026 | Completada BE-01: `accounts/balances.py` con `current_balance`, `totals` y `card_status` sobre `Money`. `tests/unit/test_account_balances.py` (24): CA-ACC-01, 02, 03 y 07, solo bancarias/billeteras/efectivo en el saldo disponible, archivadas excluidas, sin cuentas, sobregiro, saldo a favor, montos de 19 dígitos, uso redondeado al par (12,5 % → 12; 13,5 % → 14), deuda igual y superior al cupo, cupo no positivo. Verificado: `uv run pytest tests/unit/test_account_balances.py` (24/24), ruff y mypy. | Claude Code |
| 30/10/2026 | Completada BE-02: `finora/validation.py` (texto sin caracteres de control, ahora compartido con `authentication`), `accounts/schemas.py` (montos solo como cadena con hasta 2 decimales, nombre y entidad de 1 a 60, entidad vacía = ninguna, días 1–31 estrictos, `extra="forbid"`) y `accounts/service.py` (crear, listar, consultar, editar, archivar, reactivar, eliminar; filtro por usuario; nombre único sin mayúsculas con respaldo del índice único ante carreras; tope de 100; `movement_count` = 0). Pruebas: `test_account_schemas.py` (48) y `test_accounts_service.py` (31). Verificado: backend 536/536, ruff y mypy. | Claude Code |
| 30/10/2026 | Completada BE-03: `api/v1/accounts.py` con los 7 endpoints (`CurrentUser` + `UserDb`, Origin y CSRF en escrituras), modelos de respuesta en `accounts/schemas.py` (montos como cadena con 2 decimales, también en el OpenAPI) y `Cache-Control: no-store` en `/accounts`. `tests/integration/test_accounts_api.py` (21): CA-ACC-01 a 08 por HTTP, respuesta idéntica al contrato, cupo superado, archivar/reactivar con totales, eliminar, aislamiento entre dos sesiones, `401` sin sesión, `403` sin CSRF o desde otro origen. Verificado: backend 557/557, ruff y mypy. Contrato: endpoints `Implementado en finora-back`; `DELETE` documenta el cuerpo `{}`. | Claude Code |
