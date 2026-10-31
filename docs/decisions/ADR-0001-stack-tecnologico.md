# ADR-0001: Stack tecnológico inicial de finora-web y finora-back

- **Estado:** Aceptada (29/10/2026)
- **Fecha:** 29/10/2026
- **Specs relacionadas:** `specs/architecture.md` (DP-ARQ-01 a 05, 09, 10, 11), `specs/database.md` (DP-DB-01, 02, 06, 07), `specs/api-contracts.md` (DP-API-01 a 08), `specs/security.md` (DP-SEC-01), `docs/guides/claude-design.md`

## Contexto

El repositorio no tiene código y el stack de ambas aplicaciones está sin decidir. Restricciones confirmadas:

- `finora-web` y `finora-back` son aplicaciones independientes que se comunican solo mediante el contrato de API.
- El dinero nunca se representa con punto flotante binario; los cálculos financieros viven en el backend.
- Aislamiento estricto de datos por usuario.
- El diseño importado de Claude Design es un prototipo React con variables CSS (dos temas), SVG en línea para gráficas y sin assets externos.
- Hay módulos futuros con IA, OCR de recibos y lectura de extractos PDF/Excel.
- Locale `es-CO`, COP, WCAG 2.2 AA.

## Decisión

### finora-web

| Aspecto | Propuesta |
|---|---|
| Lenguaje | TypeScript (modo `strict`) |
| Framework | React + Vite como SPA, con React Router |
| Estado del servidor | TanStack Query (caché, reintentos y estados de carga/error) |
| Formularios | React Hook Form + Zod (validación solo como ayuda al usuario) |
| Estilos y tokens (DP-ARQ-11) | Variables CSS con los tokens de `claude-design.md` (temas oscuro/claro) + CSS Modules |
| Gráficas | Componentes SVG propios (`LineAreaChart`, `BarChart`, `DonutChart`), adaptados del prototipo |
| Íconos y fuentes | Inter y Material Symbols Rounded autoalojadas (sujeto a CD-08) |
| Formato es-CO | `Intl.NumberFormat` / `Intl.DateTimeFormat` solo para mostrar; los montos llegan como cadena decimal y no se operan en el cliente |
| Tipos de la API | Generados desde OpenAPI (`openapi-typescript`) |
| Pruebas | Vitest + Testing Library (unitarias/componentes), Playwright (E2E), axe-core (accesibilidad) |
| Calidad | ESLint, Prettier, `tsc --noEmit`; gestor de paquetes pnpm |

### finora-back

| Aspecto | Propuesta |
|---|---|
| Lenguaje | Python 3 (versión estable vigente al iniciar), tipado con mypy |
| Framework | FastAPI + Pydantic v2 |
| Acceso a datos | SQLAlchemy 2 (ORM) + Alembic (migraciones) |
| Dinero (DP-ARQ-09, DP-DB-02) | `decimal.Decimal` en el dominio, `NUMERIC(19,4)` en la base de datos, redondeo explícito con `ROUND_HALF_EVEN` (redondeo bancario, confirmado 29/10/2026) al presentar o liquidar |
| Pruebas | pytest + Hypothesis (propiedades de cálculos monetarios) + Testcontainers (PostgreSQL real en integración) |
| Calidad | ruff (lint y formato), mypy; gestor de entornos uv |

### Base de datos (DP-ARQ-04, DP-DB-01, 06, 07)

- **PostgreSQL** (versión estable vigente).
- Aislamiento en dos capas: filtro obligatorio por `user_id` en el repositorio de la aplicación + **Row-Level Security** como defensa en profundidad.
- Instantes en `timestamptz` (UTC); fechas contables como `DATE` interpretadas en `America/Bogota`.
- Desarrollo local con Docker Compose (solo la base de datos).

### API (DP-ARQ-03, DP-API-01 a 08)

- **REST + JSON**, versionado en la ruta (`/api/v1/...`).
- Especificación **OpenAPI 3.1** generada por FastAPI y publicada como referencia de `specs/api-contracts.md`; el frontend genera tipos desde ella.
- Montos como **cadena decimal** más código de moneda: `{"amount": "125000.00", "currency": "COP"}`.
- Fechas en ISO 8601 (`2026-10-29`); el formato DD/MM/AAAA es solo de presentación.
- Errores en formato RFC 9457 (`application/problem+json`) con mensajes en es-CO.
- Paginación por cursor en listados de movimientos.

### Autenticación (DP-ARQ-05, DP-SEC-01)

- Correo y contraseña con hash **Argon2id**, gestionada por `finora-back`.
- Sesión en el servidor con cookie `HttpOnly`, `Secure`, `SameSite=Lax`; token anti-CSRF en peticiones que modifican datos.
- MFA (TOTP) y recuperación de cuenta se definen en `authentication/spec.md`.

### Fuera de esta decisión

Hosting (DP-ARQ-06), proveedor de IA (DP-ARQ-07) y almacenamiento de archivos (DP-ARQ-08) no bloquean las primeras features y se decidirán en ADRs posteriores.

## Alternativas consideradas

| Alternativa | Pros | Contras | Motivo del descarte |
|---|---|---|---|
| **Backend en TypeScript** (NestJS o Fastify + Prisma) | Un solo lenguaje en todo el proyecto; tipos compartibles | El número de JS es punto flotante: el decimal depende de librerías (`decimal.js`) y es fácil equivocarse; ecosistema de IA/OCR/PDF menos maduro | Mayor riesgo en la regla de precisión monetaria |
| **Next.js full-stack** | SSR, despliegue integrado | Mezcla frontend y backend, contra la independencia de las aplicaciones; SSR aporta poco en una app autenticada | Contradice `architecture.md` §2.1 |
| **Java/Kotlin + Spring Boot** | `BigDecimal` sólido, muy maduro en banca | Más ceremonia y ciclos de desarrollo más lentos para un equipo pequeño | Costo de desarrollo alto para la etapa actual |
| **.NET (ASP.NET Core)** | `decimal` nativo, buen rendimiento | Ecosistema de IA/documentos menos directo que Python | Sin ventaja decisiva sobre Python |
| **Montos como enteros en unidad mínima** | Aritmética entera simple | ISO 4217 asigna 2 decimales al COP pero en la práctica no se usan; tasas e intereses requieren más precisión intermedia | `Decimal` + `NUMERIC` cubre ambos casos sin conversiones |
| **Proveedor de autenticación gestionado** (Auth0, Clerk, Keycloak) | Menos código de seguridad propio, MFA incluido | Costo, dependencia externa, datos personales fuera de la plataforma (implicaciones de la normativa colombiana por precisar) | Se puede adoptar después; se reevalúa en `authentication` |
| **Tailwind CSS** | Rapidez, utilidades consistentes | Duplica el sistema de tokens del diseño; el prototipo ya está en variables CSS | Fidelidad al diseño con menos traducción |
| **Librería de gráficas** (Recharts, ECharts) | Menos código propio | Difícil igualar el estilo del prototipo; peso del bundle | El prototipo ya define gráficas SVG simples |

## Consecuencias

**Positivas**

- La precisión monetaria la garantizan el tipo nativo y la base de datos, no la disciplina del equipo.
- El contrato OpenAPI y los tipos generados evitan que frontend y backend se desalineen.
- RLS reduce el impacto de un error de filtrado por usuario.
- React facilita trasladar el prototipo (que ya es React) a componentes reutilizables.
- Python deja abierto el camino a IA, OCR y lectura de extractos.

**Negativas y riesgos**

- Dos lenguajes (TypeScript y Python): dos toolchains y dos conjuntos de convenciones.
- La autenticación propia exige revisión de seguridad cuidadosa (sesiones, CSRF, limitación de intentos, recuperación).
- RLS añade complejidad en migraciones y pruebas (hay que fijar el usuario en cada conexión).
- Los montos como cadena obligan al frontend a no hacer aritmética con ellos; si alguna vez la necesita, debe usar una librería decimal.
