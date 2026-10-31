# Especificación: Estructura base del proyecto

| Campo | Valor |
|---|---|
| Identificador | `FEAT-SETUP` |
| Estado | `Aprobado` |
| Responsable | Responsable de producto (por confirmar) |
| Última actualización | 29/10/2026 |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> Leyenda de certeza (aplicar a cada requisito o regla):
> - ✅ **Confirmado**: validado por el responsable de producto.
> - 🟡 **Supuesto**: hipótesis de trabajo; requiere validación.
> - ❓ **Pregunta abierta**: sin resolver; bloquea la aprobación si afecta el alcance.

---

## 1. Propósito

Dejar `finora-web` y `finora-back` listas para desarrollar features: dependencias, configuración, calidad de código, pruebas y base de datos local, según el stack aprobado en [ADR-0001](../../../docs/decisions/ADR-0001-stack-tecnologico.md). Es una feature técnica: no entrega funcionalidad visible al usuario final.

## 2. Alcance

- Proyecto `finora-web` (TypeScript + React + Vite) que compila, pasa lint, verificación de tipos y pruebas.
- Proyecto `finora-back` (Python + FastAPI) que arranca, pasa lint, mypy y pruebas.
- Base de datos PostgreSQL local para desarrollo e integración.
- Tokens de diseño de `docs/guides/claude-design.md` como variables CSS (temas oscuro y claro) y fuente Inter.
- Módulo de dinero en el backend (`Decimal`, redondeo `ROUND_HALF_EVEN`) con pruebas de casos borde.
- Formato es-CO de montos y fechas en el frontend, solo para presentación.
- Endpoint de salud para comprobar que el backend responde.

## 3. Fuera de alcance

- Pantallas, navegación y componentes de cualquier módulo (cada uno se especifica en su feature).
- Esquema de base de datos de negocio (cuentas, movimientos, etc.).
- Autenticación (feature `authentication`).
- Hosting, CI/CD, proveedor de IA y almacenamiento de archivos (DP-ARQ-06, 07, 08).

## 4. Actores y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| Desarrollador | Persona que trabaja en el repositorio | Ejecuta las aplicaciones y las pruebas en local |

## 5. Historias de usuario

- **HU-01**: Como desarrollador, quiero instalar y arrancar cada aplicación con pocos comandos documentados, para empezar a implementar features.
- **HU-02**: Como desarrollador, quiero que lint, tipos y pruebas se ejecuten con un solo comando por aplicación, para verificar cada tarea.
- **HU-03**: Como desarrollador, quiero un tipo de dinero seguro y probado, para que ninguna feature use punto flotante.

## 6. Requisitos funcionales

| ID | Requisito | Prioridad | Certeza |
|---|---|---|---|
| RF-SETUP-01 | `finora-web` se instala con pnpm y expone scripts `dev`, `build`, `lint`, `typecheck`, `test`. | Alta | ✅ |
| RF-SETUP-02 | `finora-back` se instala con uv y expone comandos para arrancar el servidor, lint (ruff), tipos (mypy) y pruebas (pytest). | Alta | ✅ |
| RF-SETUP-03 | Existe una base PostgreSQL local reproducible para desarrollo y pruebas de integración. | Alta | ✅ |
| RF-SETUP-04 | `finora-back` tiene Alembic configurado (sin migraciones de negocio). | Alta | ✅ |
| RF-SETUP-05 | `finora-back` expone `GET /api/v1/health` que responde si el servicio está vivo. | Media | ✅ |
| RF-SETUP-06 | `finora-back` ofrece un tipo de dinero basado en `Decimal` que rechaza `float` y redondea con `ROUND_HALF_EVEN`. | Alta | ✅ |
| RF-SETUP-07 | `finora-web` define los tokens de diseño como variables CSS con tema oscuro (predeterminado) y claro. | Alta | ✅ |
| RF-SETUP-08 | `finora-web` ofrece funciones de formato es-CO para montos (cadena decimal → texto) y fechas (ISO 8601 → DD/MM/AAAA), sin aritmética sobre montos. | Alta | ✅ |
| RF-SETUP-09 | Los errores del backend usan RFC 9457 (`application/problem+json`). | Media | ✅ |
| RF-SETUP-10 | Ninguna aplicación contiene secretos; la configuración se lee de variables de entorno con un `.env.example` documentado. | Alta | ✅ |

## 7. Requisitos no funcionales

| ID | Requisito | Certeza |
|---|---|---|
| RNF-SETUP-01 | TypeScript en modo `strict`; mypy en modo estricto. | ✅ |
| RNF-SETUP-02 | Las aplicaciones no comparten dependencias ni configuración. | ✅ |
| RNF-SETUP-03 | La página base declara `lang="es-CO"` y pasa una revisión axe-core sin violaciones. | ✅ |

## 8. Reglas de negocio

| ID | Regla | Certeza |
|---|---|---|
| RN-SETUP-01 | Los montos nunca se representan como `float` (backend) ni se operan como `number` (frontend). | ✅ |
| RN-SETUP-02 | El redondeo monetario usa `ROUND_HALF_EVEN`. | ✅ |
| RN-SETUP-03 | Un monto siempre viaja con su moneda (ISO 4217). | ✅ |

## 9. Requisitos de datos

| Entidad / campo | Tipo lógico | Obligatorio | Notas |
|---|---|---|---|
| `Money.amount` | Decimal | Sí | `NUMERIC(19,4)` en BD |
| `Money.currency` | Código ISO 4217 | Sí | Inicialmente solo `COP` |

## 10. Validaciones y escenarios de error

| ID | Condición | Comportamiento esperado | Mensaje al usuario (es-CO) |
|---|---|---|---|
| VAL-SETUP-01 | Se crea `Money` con un `float` | Error de tipo | — (error de programación) |
| VAL-SETUP-02 | Se suman montos de monedas distintas | Error | — (error de programación) |
| VAL-SETUP-03 | Ruta inexistente en la API | 404 en formato RFC 9457 | "No encontramos el recurso solicitado." |

## 11. Casos borde

- Redondeo en el punto medio: `0.125 → 0.12`, `0.135 → 0.14` (a 2 decimales).
- Montos negativos (saldos negativos de tarjetas) se formatean con signo.
- Montos con más de 4 decimales al persistir.
- Cadena de monto inválida en el formateador del frontend.
- Fechas: el día contable se interpreta en `America/Bogota`, no en la zona del navegador.

## 12. Criterios de aceptación

**CA-SETUP-01**: Verificación del frontend
- **Dado** un clon limpio del repositorio
- **Cuando** se ejecutan `pnpm install` y `pnpm lint && pnpm typecheck && pnpm test && pnpm build` en `finora-web`
- **Entonces** todos terminan sin errores.

**CA-SETUP-02**: Verificación del backend
- **Dado** un clon limpio del repositorio
- **Cuando** se ejecutan `uv sync` y `uv run ruff check . && uv run mypy . && uv run pytest` en `finora-back`
- **Entonces** todos terminan sin errores.

**CA-SETUP-03**: Redondeo bancario
- **Dado** el tipo de dinero del backend
- **Cuando** se redondea `0.125 COP` y `0.135 COP` a 2 decimales
- **Entonces** el resultado es `0.12` y `0.14`.

**CA-SETUP-04**: Formato es-CO
- **Dado** el monto `"1234567.5"` en `COP` y la fecha `2026-10-29`
- **Cuando** se formatean en el frontend
- **Entonces** se muestran como `$ 1.234.568` (COP sin decimales, redondeo `ROUND_HALF_EVEN` solo para mostrar) y `29/10/2026`.
- **Nota:** el espacio entre `$` y el número es un espacio no separable (U+00A0), tal como lo genera `Intl.NumberFormat('es-CO')`, para que el símbolo y el monto nunca queden en líneas distintas. Las pruebas comparan contra U+00A0, no contra un espacio normal.

**CA-SETUP-05**: Salud del backend
- **Dado** el backend en ejecución
- **Cuando** se llama `GET /api/v1/health`
- **Entonces** responde `200` con `{"status": "ok"}`.

## 13. Dependencias

- [ADR-0001](../../../docs/decisions/ADR-0001-stack-tecnologico.md) (Aceptada).
- Tokens de `docs/guides/claude-design.md`; conflicto CD-08 (íconos) sin resolver, no bloquea esta feature.

## 14. Preguntas abiertas

| ID | Pregunta | Responsable | Estado |
|---|---|---|---|
| PA-SETUP-01 | Docker está instalado pero sin el plugin `compose`. ¿Se instala `docker compose`, se usa `podman compose`, o un script con `docker run`? | Desarrollador | Resuelta (29/10/2026): se instala el plugin `docker compose` y se usa `compose.yaml`. |
| PA-SETUP-02 | ¿Los montos en COP se muestran sin decimales (`$ 1.234.568`) o con dos (`$ 1.234.567,50`)? | Producto | Resuelta (29/10/2026): sin decimales. |
| PA-SETUP-03 | ¿Se aprueba `GET /api/v1/health` como primer endpoint del contrato? | Producto / Desarrollador | Resuelta (29/10/2026): aprobado. |

## 15. Supuestos

| ID | Supuesto | Riesgo si es falso |
|---|---|---|
| SUP-SETUP-01 | Se usan las versiones estables vigentes: Node 22 (instalado), Python 3.14 (instalado), PostgreSQL 18. | Ajustar versiones en el plan |
| SUP-SETUP-02 | ✅ Confirmado: pnpm se instala con `npm install -g pnpm` (no hay corepack en el sistema). | — |

## 16. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación a partir de ADR-0001 | Claude Code |
| 29/10/2026 | Aprobada; PA-SETUP-01 a 03 siguen abiertas y bloquean T-SETUP-DB-01, BE-03 y FE-03 | Claude Code |
| 29/10/2026 | Resueltas PA-SETUP-01 (docker compose), PA-SETUP-02 (COP sin decimales) y PA-SETUP-03 (health aprobado) | Claude Code |
| 29/10/2026 | Precisado CA-SETUP-04: el separador entre `$` y el monto es un espacio no separable (U+00A0). Aclaración detectada en QA-01; no cambia la regla de negocio ni el código. | Claude Code |
