# Arquitectura

| Campo | Valor |
|---|---|
| Estado | `Draft` |
| Última actualización | 29/10/2026 |

## 1. Estado actual

Al inicializar el proyecto (29/10/2026), el repositorio no contenía código. `finora-web/` y `finora-back/` existen solo como carpetas con un README. El stack inicial está definido en [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) (Aceptada, 29/10/2026). La estructura base de ambas aplicaciones está implementada (`project-setup`, completada 29/10/2026). La autenticación sigue [ADR-0002](../docs/decisions/ADR-0002-autenticacion-google-y-correo.md).

## 2. Principios (confirmados)

1. **Aplicaciones independientes**: `finora-web` (frontend) y `finora-back` (backend) tienen dependencias, configuración, pruebas y ciclo de despliegue propios.
2. **Specs compartidas en la raíz**: `specs/` es la fuente de verdad para ambas aplicaciones.
3. **Contrato API explícito**: la única vía de comunicación entre aplicaciones es la API documentada en `specs/api-contracts.md`.
4. **Lógica de negocio en el backend**: cálculos financieros, clasificación de movimientos y validación definitiva residen en `finora-back`.
5. **Aislamiento por usuario** en todas las capas de acceso a datos.
6. **Precisión monetaria segura**: el dinero nunca se representa con punto flotante binario.
7. **Diseño como referencia**: `finora-web` adapta el diseño aprobado de Claude Design a una arquitectura de componentes (componentes reutilizables separados de los específicos de página).
8. **Accesibilidad**: WCAG 2.2 AA como objetivo.
9. **Decisiones registradas**: toda decisión arquitectónica relevante se documenta como ADR en `docs/decisions/`.

## 3. Decisiones resueltas

| ID | Resolución | Fuente |
|---|---|---|
| DP-ARQ-01 | TypeScript + React + Vite (SPA) con React Router, TanStack Query y React Hook Form + Zod. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-02 | Python + FastAPI + Pydantic v2; SQLAlchemy 2. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-03 | REST + JSON, versionado en la ruta (`/api/v1/...`). | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-04 | PostgreSQL; migraciones con Alembic. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-05 | Autenticación propia: correo y contraseña (Argon2id), sesión de servidor en cookie `HttpOnly`/`Secure`/`SameSite=Lax` y token anti-CSRF. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-09 | Decimal de precisión fija: `decimal.Decimal` en el dominio, `NUMERIC(19,4)` en BD, redondeo `ROUND_HALF_EVEN`. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-10 | Web: Vitest, Testing Library, Playwright, axe-core. Back: pytest, Hypothesis, Testcontainers. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-11 | Variables CSS con los tokens de `docs/guides/claude-design.md` (temas oscuro/claro) + CSS Modules. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-ARQ-13 | Inicio con Google por OpenID Connect (Authorization Code + PKCE) gestionado por `finora-back` con Authlib; mismo modelo de sesión que la contraseña. | [ADR-0002](../docs/decisions/ADR-0002-autenticacion-google-y-correo.md) |
| DP-ARQ-14 | Correos transaccionales detrás del puerto `EmailSender`; adaptador SMTP; Mailpit en desarrollo y pruebas; envío en segundo plano. | [ADR-0002](../docs/decisions/ADR-0002-autenticacion-google-y-correo.md) |
| DP-ARQ-15 | Sesiones, tokens de un solo uso y limitación de intentos en PostgreSQL (sin Redis). | [ADR-0002](../docs/decisions/ADR-0002-autenticacion-google-y-correo.md) |

## 4. Decisiones pendientes

| ID | Decisión | Notas |
|---|---|---|
| DP-ARQ-06 | Hosting e infraestructura de despliegue | |
| DP-ARQ-07 | Proveedor de IA para el asistente financiero | |
| DP-ARQ-08 | Almacenamiento de archivos (documentos, capturas) | |
| DP-ARQ-12 | Proveedor de correo transaccional de producción | Ligada a DP-ARQ-06 y PA-AUTH-10. El adaptador SMTP de ADR-0002 sirve mientras tanto. |
