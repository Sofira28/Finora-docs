# Tareas: Estructura base del proyecto

| Campo | Valor |
|---|---|
| Identificador | `FEAT-SETUP` |
| Estado | `Completado` |
| Especificación | [`spec.md`](spec.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Última actualización | 29/10/2026 |

> Reglas:
> - Solo se listan tareas derivadas de un plan **aprobado**.
> - Una tarea se marca `[x]` **solo** después de ejecutar su verificación con resultado exitoso.
> - Aplicación: `finora-web` | `finora-back` | `shared` | `infraestructura`.
> - Prioridad: `Alta` | `Media` | `Baja`.

Leyenda de estado:

- [ ] Pendiente
- [x] Completada

---

## Frontend (`finora-web`)

- [x] **T-SETUP-FE-01**: Crear el proyecto Vite + React + TypeScript `strict` con pnpm, ESLint, Prettier y Vitest.
  - **Aplicación:** `finora-web`
  - **Dependencias:** —
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-SETUP-01
  - **Verificación:** `pnpm lint && pnpm typecheck && pnpm test && pnpm build`

- [x] **T-SETUP-FE-02**: Tokens de diseño (`tokens.css`, temas oscuro/claro), estilos globales e Inter autoalojada; `index.html` con `lang="es-CO"`.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-SETUP-FE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-SETUP-07, RNF-SETUP-03
  - **Verificación:** `pnpm test` (prueba axe sin violaciones)

- [x] **T-SETUP-FE-03**: `formatMoney` y `formatDate` es-CO con pruebas (montos grandes, negativos, cadenas inválidas, zona `America/Bogota`).
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-SETUP-FE-01, PA-SETUP-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-SETUP-04
  - **Verificación:** `pnpm test src/lib/format`

## Backend (`finora-back`)

- [x] **T-SETUP-BE-01**: Crear el proyecto uv con ruff, mypy `strict` y pytest.
  - **Aplicación:** `finora-back`
  - **Dependencias:** —
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-SETUP-02
  - **Verificación:** `uv run ruff check . && uv run mypy . && uv run pytest`

- [x] **T-SETUP-BE-02**: Tipo `Money` (Decimal, `ROUND_HALF_EVEN`, rechazo de `float`, misma moneda) con pruebas unitarias y Hypothesis.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-SETUP-BE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-SETUP-03, VAL-SETUP-01, VAL-SETUP-02
  - **Verificación:** `uv run pytest tests/unit/test_money.py`

- [x] **T-SETUP-BE-03**: App FastAPI con configuración por entorno, errores RFC 9457 y `GET /api/v1/health`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-SETUP-BE-01, PA-SETUP-03
  - **Prioridad:** Media
  - **Criterios de aceptación:** CA-SETUP-05, VAL-SETUP-03
  - **Verificación:** `uv run pytest tests/integration/test_health.py`

## Base de datos

- [x] **T-SETUP-DB-01**: PostgreSQL local en contenedor, SQLAlchemy 2 + psycopg 3 y Alembic inicializado.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-SETUP-BE-01, PA-SETUP-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-SETUP-03, RF-SETUP-04
  - **Verificación:** `uv run pytest tests/integration/test_db.py && uv run alembic current`

## Integración

- [x] **T-SETUP-INT-01**: Registrar `GET /api/v1/health` en `specs/api-contracts.md` y documentar comandos en ambos README.
  - **Aplicación:** `shared`
  - **Dependencias:** T-SETUP-BE-03
  - **Prioridad:** Media
  - **Criterios de aceptación:** RF-SETUP-05, HU-01
  - **Verificación:** Revisión manual de documentos

## Pruebas

- [x] **T-SETUP-QA-01**: Ejecutar desde cero los criterios CA-SETUP-01 a 05.
  - **Aplicación:** `finora-web` / `finora-back`
  - **Dependencias:** Todas las anteriores
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-SETUP-01 a CA-SETUP-05
  - **Verificación:** Comandos de CA-SETUP-01 y CA-SETUP-02

---

## Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación | Claude Code |
| 29/10/2026 | Completadas FE-01/02/03, BE-01/02/03 e INT-01 (verificaciones en verde). DB-01 escrita pero sin verificar: Docker no accesible para el usuario. TypeScript fijado en 6.0 (typescript-eslint no soporta 7.0). | Claude Code |
| 29/10/2026 | Verificada y completada DB-01 (Docker accesible; `test_db.py` 3/3 y `alembic current` sin errores; suite backend 51/51). | Claude Code |
| 29/10/2026 | Verificada y completada QA-01 en copia limpia (sin dependencias ni `.env`): CA-SETUP-01 (lint, typecheck, 30 pruebas, build), CA-SETUP-02 (ruff, mypy, 51 pruebas), CA-SETUP-03, CA-SETUP-04 y CA-SETUP-05 (`GET /api/v1/health` → 200 `{"status":"ok"}` con uvicorn real). | Claude Code |
