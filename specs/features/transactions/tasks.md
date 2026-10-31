# Tareas: Transacciones

| Campo | Valor |
|---|---|
| Identificador | `FEAT-TRX` |
| Estado | `En progreso` |
| Especificación | [`spec.md`](spec.md) (Aprobado, 30/10/2026) |
| Plan técnico | [`plan.md`](plan.md) (Aprobado, 30/10/2026) |
| Última actualización | 30/10/2026 |

> Reglas:
> - Solo se listan tareas derivadas de un plan **aprobado**; estas se aprueban junto con el plan.
> - Una tarea se marca `[x]` **solo** después de ejecutar su verificación con resultado exitoso.
> - Aplicación: `finora-web` | `finora-back` | `shared` | `infraestructura`.
> - Prioridad: `Alta` | `Media` | `Baja`.
> - Los comandos de `finora-back` se ejecutan en esa carpeta y requieren Docker. Los de `finora-web`, en la suya.

Leyenda de estado:

- [ ] Pendiente
- [x] Completada

Orden sugerido: INT-01 → DB-01 → BE-01 a BE-04 → FE-01 a FE-04 → QA-01 a QA-05 → INT-02.

---

## Integración

- [x] **T-TRX-INT-01**: Registrar los 5 endpoints de `plan.md` §6 en `specs/api-contracts.md` (§3 y una sección nueva "Transacciones", estado `Aprobado`, sin implementar), la tabla de `plan.md` §5 en `specs/database.md` §5 y DP-DB-03 como resuelto (ADR-0005).
  - **Aplicación:** `shared`
  - **Dependencias:** Plan aprobado
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Endpoints, modelo `Transaction`, `TransactionRequest` por tipo, códigos de error y tabla coinciden con `plan.md` §5 y §6
  - **Verificación:** Revisión manual de documentos

## Base de datos

- [x] **T-TRX-DB-01**: Migración `0005_transactions`: única (`id`, `user_id`) en `accounts`; tabla `transactions` con `CHECK` por tipo, categoría y destino, FK compuestas `NO ACTION`, índices, RLS y política `transactions_owner`. Modelo `Transaction` y enums registrados en `db/models.py`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-TRX-INT-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** `alembic upgrade`/`downgrade` limpios; `alembic check` sin diferencias; RLS sin usuario fijado devuelve cero filas; una FK a una cuenta de otro usuario falla; no se puede borrar una cuenta con movimientos; borrar el usuario borra cuentas y movimientos
  - **Verificación:** `uv run pytest tests/integration/test_transactions_schema.py tests/integration/test_transactions_rls.py && uv run alembic check`

## Backend (`finora-back`)

- [x] **T-TRX-BE-01**: `transactions/effects.py` (efecto por tipo, papel y tipo de cuenta; clasificación de transferencias), `categories.py` (catálogo de RN-TRX-07 con nombres es-CO) y `search.py` (`normalize`).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-TRX-DB-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Tabla de efectos de `plan.md` §2 completa; ahorro, retiro de ahorro y otras transferencias; `normalize("Café ÁRBOL")` = `"cafe arbol"`
  - **Verificación:** `uv run pytest tests/unit/test_transaction_effects.py tests/unit/test_transaction_categories.py`

- [x] **T-TRX-BE-02**: `schemas.py` (petición por tipo con `extra="forbid"`, respuestas) y `service.py` (`list`, `get`, `create`, `replace`, `delete`, con las validaciones VAL-TRX-01 a 11, el ajuste por `target_balance` y el resumen de RN-TRX-05 con filtros y orden).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-TRX-BE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** VAL-TRX-01 a 11 con sus mensajes; fecha "hoy" en hora de Colombia con `ClockDep`; resumen sobre todo el filtro, no solo la página; búsqueda sin tildes en descripción, nota, categoría y cuenta; orden estable
  - **Verificación:** `uv run pytest tests/unit/test_transaction_schemas.py tests/integration/test_transactions_service.py`

- [x] **T-TRX-BE-03**: Saldos reales en `accounts`: `effects_by_account` (una consulta agregada), `current_balance(opening, effect)` y `movement_count` real. La consulta SQL y `effects.py` deben coincidir.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-TRX-BE-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-TRX-01 a 06 con los saldos y totales de `/accounts`; CA-TRX-08 (VAL-ACC-07 y 08 con movimientos reales); las pruebas existentes de `accounts` siguen en verde
  - **Verificación:** `uv run pytest tests/integration/test_accounts_with_transactions.py tests/integration/test_accounts_api.py`

- [ ] **T-TRX-BE-04**: `api/v1/transactions.py` con los 5 endpoints de `plan.md` §6 (`CurrentUser` + `UserDb`, Origin y CSRF en escrituras, `Cache-Control: no-store`).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-TRX-BE-03
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-TRX-01 a 12 por HTTP; respuestas idénticas al contrato; `401` sin sesión, `403` sin CSRF, `404` para movimientos ajenos, `422` para cuentas ajenas; paginación con `next_cursor` (cursor inválido → `422`)
  - **Verificación:** `uv run pytest tests/integration/test_transactions_api.py && uv run ruff check . && uv run mypy .`

## Frontend (`finora-web`)

- [ ] **T-TRX-FE-01**: `pnpm api:types`; `categories.ts` (nombres y colores), `periods.ts` (periodos → `from`/`to` en hora de Colombia), `TransactionsIcon`, "Movimientos" en `NAV_ITEMS` y ruta `/movimientos` (`lazy`).
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-BE-04
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Periodos correctos en cambios de mes y año y a medianoche de Colombia; navegación con "Inicio", "Movimientos" y "Cuentas"; pruebas de `AppShell` en verde
  - **Verificación:** `pnpm test src/features/transactions src/app && pnpm lint && pnpm typecheck`

- [ ] **T-TRX-FE-02**: Página Movimientos: filtros en la URL, resumen, lista (tabla en escritorio, lista en móvil), "Cargar más" y estados de carga, error con reintento, vacío y sin resultados.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-FE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-TRX-04 a 06, 09 y 10; CA-TRX-10 en pantalla; montos con signo y texto (RNF-TRX-04); el frontend no suma montos
  - **Verificación:** `pnpm test src/features/transactions`

- [ ] **T-TRX-FE-03**: `TransactionFormDialog` para crear y editar los cuatro tipos: campos por tipo, monto con `MoneyField`, fecha hasta hoy, solo cuentas activas, categorías del tipo, ajuste con saldo real y errores `422` del backend en su campo.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-FE-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-TRX-01 a 03 y 07; CA-TRX-09; VAL-TRX-01 a 09 con sus mensajes; editar un movimiento de una cuenta archivada sin cambiar la cuenta
  - **Verificación:** `pnpm test src/features/transactions`

- [ ] **T-TRX-FE-04**: Eliminar con `ConfirmDialog` desde el menú de cada movimiento (`404` cierra y recarga) y "Ajustar saldo" en el menú de acciones de Cuentas (también enlazado desde el saldo bloqueado del formulario de cuenta).
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-FE-03
  - **Prioridad:** Media
  - **Criterios de aceptación:** RF-TRX-08; CA-TRX-06 y 07 en pantalla; saldos y totales de Cuentas se recargan tras cada cambio
  - **Verificación:** `pnpm test src/features/transactions src/features/accounts`

## Pruebas

- [ ] **T-TRX-QA-01**: E2E (Playwright) contra el backend real: CA-TRX-01, 03, 04 y 05 con las cuentas de CA-ACC-01 y los totales de Cuentas; editar y eliminar.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-FE-04
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Flujo completo sin errores
  - **Verificación:** `pnpm test:e2e`

- [ ] **T-TRX-QA-02**: Accesibilidad: axe WCAG 2.2 AA (con contraste) en Movimientos y el modal en los tres rangos, reflow a 320 px, foco visible y teclado en filtros, menú y diálogos.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-QA-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RNF-TRX-04 sin violaciones
  - **Verificación:** `pnpm test:e2e e2e/accessibility.spec.ts`

- [ ] **T-TRX-QA-03**: Rendimiento: listado, resumen y `/accounts` con 10.000 movimientos por usuario.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-TRX-BE-04
  - **Prioridad:** Media
  - **Criterios de aceptación:** RNF-TRX-06 (< 1 s); si no se cumple, propuesta según ADR-0005 §Consecuencias
  - **Verificación:** `uv run pytest tests/integration/test_transactions_performance.py`

- [ ] **T-TRX-QA-04**: Revisión de seguridad de la feature (aislamiento, RLS, FK compuestas, validación de montos y fechas, búsqueda) y corrección de los hallazgos confirmados.
  - **Aplicación:** `shared`
  - **Dependencias:** T-TRX-QA-02, T-TRX-QA-03
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Sin hallazgos abiertos de severidad alta o media
  - **Verificación:** Informe de revisión y suites completas en verde

- [ ] **T-TRX-QA-05**: Revisión visual de Movimientos y el modal contra Claude Design con la responsable de producto, en escritorio, tablet y móvil.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-TRX-QA-02
  - **Prioridad:** Media
  - **Criterios de aceptación:** Aprobación de la responsable de producto
  - **Verificación:** Revisión manual

## Cierre

- [ ] **T-TRX-INT-02**: Endpoints a `Implementado` en `api-contracts.md`, esquema implementado en `database.md`, ADR-0005 en el índice, READMEs y estado de la feature.
  - **Aplicación:** `shared`
  - **Dependencias:** T-TRX-QA-04, T-TRX-QA-05
  - **Prioridad:** Media
  - **Criterios de aceptación:** Documentación coherente con lo implementado
  - **Verificación:** Revisión manual de documentos

## Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 30/10/2026 | Tareas derivadas del plan (`En revisión`): INT-01, DB-01, BE-01 a 04, FE-01 a 04, QA-01 a 05 e INT-02. | Claude Code |
| 30/10/2026 | Aprobadas junto con el plan. | Claude Code |
| 30/10/2026 | T-TRX-INT-01 completada: §3 y §6 de `api-contracts.md`, §3 y §5.3 de `database.md`. Paginación por cursor (DP-API-06) confirmada por la usuaria; orden por monto en valor absoluto; claves de categoría en inglés. | Claude Code |
| 30/10/2026 | T-TRX-DB-01 completada: migración `0005_transactions` (única (`id`, `user_id`) en `accounts`, tabla con `CHECK`, FK compuestas `NO ACTION`, índices y RLS), modelo `Transaction` con `TransactionType` y `Category`. 33 pruebas nuevas de esquema y RLS; suite completa 590/590, ruff, mypy y `alembic check` limpios. El `CHECK` de categoría exige `IS NOT NULL` explícito, porque `NULL IN (...)` es NULL y el `CHECK` lo dejaría pasar. | Claude Code |
| 30/10/2026 | T-TRX-BE-01 completada: `effects.py` (tabla de signos `SIGNS` + inversión en tarjetas, única fuente de la regla para el `CASE` de SQL en BE-03; `classify_transfer`), `categories.py` (nombres es-CO, `is_allowed`, `matching` para la búsqueda) y `search.py` (`normalize`, `search_text` y `like_pattern`, que escapa `%`, `_` y `\\`). 92 pruebas unitarias, incluidos CA-TRX-01 a 06 y 12 sobre los totales de `accounts`; unitarias 277/277, ruff y mypy limpios. | Claude Code |
| 30/10/2026 | T-TRX-BE-02 completada: `schemas.py` (petición única con validación de formato; fecha solo `AAAA-MM-DD`; nota con saltos de línea), `ledger.py` (saldos en SQL desde la tabla de `effects`) y `service.py` (crear, reemplazar, eliminar, consultar y listar con filtros, búsqueda, resumen y cursor; VAL-TRX-01 a 11). 31 pruebas unitarias y 68 de integración, incluida la paridad SQL/Python de los efectos en todas las combinaciones de tipo y cuenta; suite completa 781/781, ruff y mypy limpios. Desviaciones del plan registradas en su historial. | Claude Code |
| 30/10/2026 | T-TRX-BE-03 completada: `/accounts` calcula saldos y `has_movements` con una sola consulta (`effects_by_account`) para todo el listado; `movement_count` cuenta movimientos reales como origen o destino; `delete_account` responde `409 account_has_movements` también si la FK lo detecta por una carrera. 13 pruebas HTTP nuevas (CA-TRX-01 a 08, cuenta archivada y aislamiento); suite completa 794/794, ruff y mypy limpios. | Claude Code |
