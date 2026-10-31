# Plan técnico: Estructura base del proyecto

| Campo | Valor |
|---|---|
| Identificador | `FEAT-SETUP` |
| Estado | `Aprobado` |
| Especificación | [`spec.md`](spec.md) (29/10/2026) |
| Tareas | [`tasks.md`](tasks.md) |
| Última actualización | 29/10/2026 |

> Este plan solo puede aprobarse cuando la especificación referenciada esté en estado `Aprobado`.

---

## 1. Contexto de la arquitectura actual

`finora-web/` y `finora-back/` solo contienen un README. Stack aprobado en [ADR-0001](../../../docs/decisions/ADR-0001-stack-tecnologico.md). Entorno local: Node 22, Python 3.14, uv y Docker (sin `compose`) o Podman; pnpm no instalado.

## 2. Solución propuesta

Crear cada aplicación con sus herramientas oficiales, añadir solo la configuración mínima de calidad y pruebas, y dos piezas transversales que todas las features usarán: el tipo de dinero (backend) y los formateadores es-CO + tokens de diseño (frontend).

## 3. Cambios en el frontend (`finora-web`)

```text
finora-web/
├── package.json, pnpm-lock.yaml, tsconfig*.json, vite.config.ts
├── eslint.config.js, .prettierrc, .env.example
├── index.html                 ← lang="es-CO", Inter
└── src/
    ├── main.tsx, App.tsx      ← página mínima de comprobación
    ├── styles/tokens.css      ← variables CSS, tema oscuro y claro ([data-theme])
    ├── styles/global.css
    └── lib/format/            ← formatMoney(amount: string, currency), formatDate(iso)
        └── *.test.ts
```

- Inter autoalojada mediante `@fontsource-variable/inter`.
- `formatMoney` usa `Intl.NumberFormat('es-CO')` sin convertir la cadena con `Number()` para operar; la conversión a número, si `Intl` la necesita, solo ocurre para mostrar y se cubre con pruebas de precisión en montos grandes. Alternativa si falla la precisión: formatear la cadena manualmente.
- React Router, TanStack Query y React Hook Form + Zod **no** se instalan aún: entran con la primera feature que los use.
- Pruebas: Vitest + Testing Library; `axe-core` directo sobre `App` (`vitest-axe` está sin mantenimiento). En jsdom no se mide contraste; queda para Playwright.
- TypeScript fijado en `~6.0`: typescript-eslint aún no soporta TypeScript 7. Playwright se configura con la primera feature con flujo E2E.

## 4. Cambios en el backend (`finora-back`)

```text
finora-back/
├── pyproject.toml, uv.lock, .env.example
├── alembic.ini, migrations/
├── src/finora/
│   ├── main.py                ← crea la app FastAPI, router /api/v1
│   ├── config.py              ← pydantic-settings, lee variables de entorno
│   ├── errors.py              ← manejadores RFC 9457
│   ├── api/v1/health.py
│   └── domain/money.py        ← Money(amount: Decimal, currency)
└── tests/
    ├── unit/test_money.py     ← casos borde + Hypothesis
    └── integration/test_db.py ← conexión a PostgreSQL (Testcontainers)
```

- `Money`: inmutable, rechaza `float`, impide operar monedas distintas, `quantize` con `ROUND_HALF_EVEN`; precisión de almacenamiento 4 decimales.
- Herramientas: ruff (lint + formato), mypy `strict`, pytest, Hypothesis, Testcontainers, SQLAlchemy 2 + psycopg 3, Alembic.

## 5. Cambios en la base de datos

- PostgreSQL 18 en contenedor para desarrollo (`finora-back/compose.yaml` con `docker compose`, PA-SETUP-01).
- Alembic inicializado, sin migraciones de negocio. RLS se configura con la primera tabla con datos de usuario.

## 6. Endpoints y contratos API

| Método | Ruta | Descripción | Request | Response | Errores |
|---|---|---|---|---|---|
| GET | `/api/v1/health` | Comprobación de vida | — | `200 {"status": "ok"}` | — |

> Aprobado (PA-SETUP-03, 29/10/2026); se registra en `specs/api-contracts.md` §3 en T-SETUP-INT-01.

## 7. Flujo de datos

No aplica (sin datos de negocio).

## 8. Consideraciones de seguridad

- Sin secretos en el repositorio: `.env` en `.gitignore`, solo `.env.example` versionado.
- `/api/v1/health` no expone versión, dependencias ni datos internos.

## 9. Estrategia de validación

- Frontend: no aplica.
- Backend: Pydantic en la configuración; `Money` valida tipos en construcción.

## 10. Manejo de errores

Manejadores globales que devuelven `application/problem+json` (404, 422, 500) con `title` en es-CO y sin trazas internas.

## 11. Estrategia de pruebas

| Nivel | Alcance | Herramienta | Criterios de aceptación cubiertos |
|---|---|---|---|
| Unitarias | `Money`, formateadores es-CO | pytest + Hypothesis, Vitest | CA-SETUP-03, CA-SETUP-04 |
| Integración | Endpoint de salud, errores RFC 9457, conexión a BD | pytest + httpx + Testcontainers | CA-SETUP-05 |
| E2E | — | — | — |
| Accesibilidad | Página mínima | axe-core | RNF-SETUP-03 |

## 12. Dependencias y riesgos

| Riesgo / dependencia | Impacto | Mitigación |
|---|---|---|
| Docker sin `compose` | No se levanta la BD local | Instalar `docker-compose-plugin` (PA-SETUP-01) |
| Testcontainers con Podman | Requiere socket de Docker/Podman activo | Documentar `DOCKER_HOST` para Podman |
| Librerías aún sin soporte para Python 3.14 | Fallo al instalar | Fijar Python 3.13 con uv si ocurre |

## 13. Secuencia de implementación

1. Backend: proyecto uv, herramientas de calidad, `Money` con pruebas.
2. Backend: FastAPI, configuración, errores RFC 9457, endpoint de salud.
3. Base de datos: contenedor PostgreSQL, SQLAlchemy, Alembic, prueba de integración.
4. Frontend: proyecto Vite, herramientas de calidad, tokens e Inter.
5. Frontend: formateadores es-CO con pruebas; revisión axe.
6. Documentación: actualizar los README con comandos.

## 14. Decisiones arquitectónicas

| Decisión | Alternativas consideradas | ADR | Estado |
|---|---|---|---|
| Stack completo | Ver ADR | `docs/decisions/ADR-0001-stack-tecnologico.md` | Aceptada |

## 15. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación | Claude Code |
| 29/10/2026 | axe-core directo en vez de vitest-axe; TypeScript 6.0 por compatibilidad con typescript-eslint | Claude Code |
