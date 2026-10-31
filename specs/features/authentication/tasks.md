# Tareas: Autenticación y perfiles de usuario

| Campo | Valor |
|---|---|
| Identificador | `FEAT-AUTH` |
| Estado | `En progreso` |
| Especificación | [`spec.md`](spec.md) |
| Plan técnico | [`plan.md`](plan.md) |
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

Orden sugerido: INT-01 → BE-01 → BE-02 → DB-01 → BE-03 a BE-11 → FE-01 a FE-06 → QA-01 a QA-05 → INT-02.

---

## Integración

- [x] **T-AUTH-INT-01**: Registrar los 21 endpoints de `plan.md` §6 en `specs/api-contracts.md` §3 (estado `Aprobado`, sin implementar) y las tablas de `plan.md` §5 en `specs/database.md`.
  - **Aplicación:** `shared`
  - **Dependencias:** Plan aprobado
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Cada endpoint tiene ruta, sesión, request, response y errores coherentes con `plan.md` §6; las tablas coinciden con `plan.md` §5
  - **Verificación:** Revisión manual de documentos

## Backend (`finora-back`)

- [x] **T-AUTH-BE-01**: Dependencias nuevas (`argon2-cffi`, `pyotp`, `authlib`, `httpx`, `cryptography`, `email-validator`; `respx` en desarrollo), `Clock` inyectable y configuración nueva en `Settings` y `.env.example` (`plan.md` §4).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-INT-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Arranca sin credenciales de Google ni SMTP de producción; `FINORA_MFA_ENCRYPTION_KEY` es obligatoria fuera de `development`
  - **Verificación:** `uv run pytest tests/unit/test_config.py tests/unit/test_clock.py && uv run mypy .`

- [x] **T-AUTH-BE-02**: Servicio `mailpit` en `compose.yaml`; `EmailSender` con `SmtpEmailSender` e `InMemoryEmailSender`; plantillas es-CO (verificación, restablecimiento, aviso de registro duplicado, aviso de cambio de contraseña).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RF-AUTH-02, RF-AUTH-08 (envío), RNF-AUTH-08
  - **Verificación:** `uv run pytest tests/integration/test_email_sender.py`

- [x] **T-AUTH-BE-03**: `passwords.py`: hash Argon2id con nuevo cálculo si cambian los parámetros, política RN-AUTH-02 (longitud en caracteres Unicode, lista de las 100.000 contraseñas más comunes, coincidencia con el correo) y normalización del correo (RN-AUTH-01).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RN-AUTH-01, RN-AUTH-02, VAL-AUTH-02, RNF-AUTH-01
  - **Verificación:** `uv run pytest tests/unit/test_passwords.py`

- [x] **T-AUTH-BE-04**: `tokens.py`: tokens de un solo uso (verificación 24 h, restablecimiento 1 h) guardados como hash; uno nuevo invalida el anterior; uso atómico.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-DB-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RN-AUTH-04, RN-AUTH-05, RNF-AUTH-05, VAL-AUTH-09; dos usos simultáneos del mismo token: solo uno tiene éxito
  - **Verificación:** `uv run pytest tests/integration/test_tokens.py`

- [x] **T-AUTH-BE-05**: `sessions.py`, `csrf.py` y `dependencies.py`:
  - Sesiones con etapas (`active`, `mfa_pending`, `link_pending`), expiración por inactividad (30 min) y máxima (7 días), sesiones parciales de 5 min.
  - Regeneración del identificador al iniciar sesión o cambiar de etapa; revocación de todas las sesiones.
  - Token anti-CSRF y verificación de `Content-Type` y `Origin`; `Cache-Control: no-store`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-DB-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-05, CA-AUTH-06, RNF-AUTH-02, VAL-AUTH-10, VAL-AUTH-11, RN-AUTH-08
  - **Verificación:** `uv run pytest tests/unit/test_sessions.py tests/integration/test_sessions.py`

- [x] **T-AUTH-BE-06**: `rate_limit.py` (5 fallos por cuenta en 15 min; límites por IP configurables; `429` con `Retry-After`) y `events.py` (eventos de seguridad sin datos sensibles).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-DB-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-09, RNF-AUTH-04, RNF-AUTH-06, VAL-AUTH-07
  - **Verificación:** `uv run pytest tests/integration/test_rate_limit.py tests/integration/test_security_events.py`

- [x] **T-AUTH-BE-07**: Endpoints `POST /auth/register`, `/auth/verify-email`, `/auth/verify-email/resend`, `/auth/login`, `/auth/logout` y `GET /auth/session`, con anti-enumeración (respuestas idénticas, hash de relleno, correos en segundo plano) y aceptación de la política de datos.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-02 a T-AUTH-BE-06
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-01, CA-AUTH-02, CA-AUTH-03, RF-AUTH-01 a 04, RF-AUTH-06, VAL-AUTH-01 a 06
  - **Verificación:** `uv run pytest tests/integration/test_auth_register.py tests/integration/test_auth_login.py`

- [x] **T-AUTH-BE-08**: Recuperación y perfil:
  - `POST /auth/password/forgot` y `/auth/password/reset`.
  - `GET` y `PATCH /me`.
  - `POST /me/password` y `/me/password/setup`.
  - Cambiar o restablecer la contraseña cierra las demás sesiones y avisa por correo.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-07
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-07, CA-AUTH-11, RF-AUTH-08, 09, 14, 15, 18, RN-AUTH-03
  - **Verificación:** `uv run pytest tests/integration/test_password_recovery.py tests/integration/test_profile.py`

- [x] **T-AUTH-BE-09**: MFA y reautenticación:
  - `mfa.py`: TOTP con ventana de ±1 intervalo, sin aceptar dos veces el mismo código; secreto cifrado con AES-GCM; 10 códigos de recuperación guardados con hash.
  - Endpoints `POST /me/mfa/setup`, `/me/mfa/confirm`, `/me/mfa/disable`, `/auth/login/mfa` y `/auth/reauthenticate`.
  - Reautenticación reciente de 5 minutos.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-07
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-08, RF-AUTH-10 a 13, RF-AUTH-21, RN-AUTH-06, VAL-AUTH-08
  - **Verificación:** `uv run pytest tests/unit/test_mfa.py tests/integration/test_mfa.py tests/integration/test_reauthenticate.py`

- [x] **T-AUTH-BE-10**: Google:
  - `GoogleOidcClient` (Authlib) con un cliente falso para las pruebas.
  - `GET /auth/google/start` (`intent=login|reauth`; `404` sin credenciales), `GET /auth/google/callback` y `POST /auth/google/link`.
  - Validación completa del ID token; MFA también con Google; reautenticación por Google.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-09
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-04, CA-AUTH-13, CA-AUTH-14, RF-AUTH-05, 17, 19, VAL-AUTH-12, VAL-AUTH-13
  - **Verificación:** `uv run pytest tests/integration/test_google_oidc.py tests/integration/test_google_flows.py`

- [x] **T-AUTH-BE-11**: `DELETE /me` (confirmación `"ELIMINAR"` y reautenticación reciente; cascada; eventos desvinculados del usuario) y `purge_expired()`.
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-09, T-AUTH-BE-10
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-10, RF-AUTH-16, RN-AUTH-07, SUP-AUTH-04
  - **Verificación:** `uv run pytest tests/integration/test_delete_account.py tests/integration/test_purge.py`

## Base de datos

- [x] **T-AUTH-DB-01**: Modelos SQLAlchemy de las 10 tablas de `plan.md` §5, migración `0001_auth` (restricciones, índices, `ON DELETE CASCADE` y `SET NULL` en `security_events`) y `tests/conftest.py` compartido (contenedor PostgreSQL, sesión de BD, cliente HTTP, `Clock` falso, `InMemoryEmailSender`).
  - **Aplicación:** `finora-back`
  - **Dependencias:** T-AUTH-BE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** La migración sube y baja sin errores; los modelos coinciden con la migración; correo único sin distinguir mayúsculas
  - **Verificación:** `uv run pytest tests/integration/test_auth_schema.py && uv run alembic upgrade head && uv run alembic check`

## Frontend (`finora-web`)

- [x] **T-AUTH-FE-01**: Base del frontend:
  - Dependencias: `react-router`, `@tanstack/react-query`, `react-hook-form`, `zod`, `@hookform/resolvers`, `qrcode`; `openapi-typescript` en desarrollo.
  - Proxy de Vite `/api` y script `pnpm api:types`.
  - Cliente API (`credentials`, `X-CSRF-Token`, `problem+json` tipado, `401` en rutas protegidas).
  - Router con `ProtectedRoute` y consulta de sesión.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-INT-01, T-AUTH-BE-07
  - **Prioridad:** Alta
  - **Criterios de aceptación:** VAL-AUTH-10, VAL-AUTH-11 (cliente); el estado de sesión decide qué rutas se muestran
  - **Verificación:** `pnpm test src/lib/api src/features/auth/session && pnpm typecheck`

- [x] **T-AUTH-FE-02**: Componentes reutilizables derivados de los tokens (CD-04):
  - `TextField`, `PasswordField`, `OtpField`, `Button`, `Alert`, `AuthLayout` y `GoogleButton`.
  - Íconos como SVG en línea.
  - Pegado permitido y `autocomplete` correcto en los campos.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-FE-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** RNF-AUTH-07, RNF-AUTH-09; sin violaciones axe en cada componente
  - **Verificación:** `pnpm test src/components`

- [x] **T-AUTH-FE-03**: Páginas `/registro`, `/registro/revisa-tu-correo` y `/verificar-correo`, con validación Zod de ayuda, enlace a la política de datos (página provisional, versión `0`), token retirado de la URL y `no-referrer`.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-FE-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** HU-01, HU-03, VAL-AUTH-01 a 04, VAL-AUTH-09 (mensajes)
  - **Verificación:** `pnpm test src/features/auth/register src/features/auth/verify-email`

- [x] **T-AUTH-FE-04**: Páginas `/iniciar-sesion` (botón de Google solo si está configurado), `/iniciar-sesion/codigo` y `/iniciar-sesion/vincular-google`; manejo de `error=google`; cierre de sesión; página de inicio provisional protegida.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-FE-02, T-AUTH-BE-09, T-AUTH-BE-10
  - **Prioridad:** Alta
  - **Criterios de aceptación:** HU-02, HU-04, VAL-AUTH-05 a 08, VAL-AUTH-13 (mensajes)
  - **Verificación:** `pnpm test src/features/auth/login`

- [x] **T-AUTH-FE-05**: Páginas `/recuperar-contrasena` y `/restablecer-contrasena`; esta última también sirve para crear la primera contraseña. Token retirado de la URL y `no-referrer`.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-FE-02, T-AUTH-BE-08
  - **Prioridad:** Alta
  - **Criterios de aceptación:** HU-05, RF-AUTH-18, VAL-AUTH-09
  - **Verificación:** `pnpm test src/features/auth/password-recovery`

- [x] **T-AUTH-FE-06**: Página `/perfil`:
  - Nombre editable y cambio o creación de contraseña.
  - MFA: QR con `qrcode`, confirmación, códigos de recuperación mostrados una vez, advertencia RF-AUTH-20 y desactivación.
  - Diálogo de reautenticación (contraseña o Google, más código).
  - Eliminación de cuenta con la confirmación `ELIMINAR`.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-FE-02, T-AUTH-BE-08, T-AUTH-BE-09, T-AUTH-BE-11
  - **Prioridad:** Alta
  - **Criterios de aceptación:** HU-06, HU-07, HU-08, RF-AUTH-13, 14, 16, 20, 21
  - **Verificación:** `pnpm test src/features/auth/profile`

## Pruebas

- [x] **T-AUTH-QA-01**: Configurar Playwright y escribir el E2E completo contra el backend y Mailpit reales: registro → verificación con el enlace de Mailpit → inicio de sesión → perfil → MFA → cierre de sesión → recuperación → eliminación.
  - **Aplicación:** `finora-web` / `finora-back`
  - **Dependencias:** T-AUTH-FE-03 a T-AUTH-FE-06
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-01, CA-AUTH-07, CA-AUTH-08, CA-AUTH-10
  - **Verificación:** `pnpm test:e2e`

- [x] **T-AUTH-QA-02**: Accesibilidad en Playwright: axe con contraste en todas las páginas de autenticación, recorrido solo con teclado y pegado en los campos de contraseña y código.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-QA-01
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-12, RNF-AUTH-07
  - **Verificación:** `pnpm test:e2e e2e/accessibility.spec.ts`

- [x] **T-AUTH-QA-03**: Revisión de seguridad de la feature con `/security-review` y corrección de los hallazgos confirmados.
  - **Aplicación:** `finora-web` / `finora-back`
  - **Dependencias:** T-AUTH-BE-11, T-AUTH-FE-06
  - **Prioridad:** Alta
  - **Criterios de aceptación:** Sin hallazgos confirmados abiertos de severidad alta o media
  - **Verificación:** Informe de `/security-review` y suites de ambas aplicaciones en verde

- [ ] **T-AUTH-QA-04**: Revisión visual de las pantallas derivadas de los tokens (CD-04) con el responsable de producto, en escritorio, tablet y móvil.
  - **Aplicación:** `finora-web`
  - **Dependencias:** T-AUTH-FE-06
  - **Prioridad:** Media
  - **Criterios de aceptación:** RNF-AUTH-09; pantallas aprobadas o ajustes registrados
  - **Verificación:** Aprobación manual registrada en el historial

- [ ] **T-AUTH-QA-05**: Verificación final desde una copia limpia de CA-AUTH-01 a CA-AUTH-14 y de las suites completas de ambas aplicaciones.
  - **Aplicación:** `finora-web` / `finora-back`
  - **Dependencias:** Todas las anteriores salvo T-AUTH-INT-02
  - **Prioridad:** Alta
  - **Criterios de aceptación:** CA-AUTH-01 a CA-AUTH-14
  - **Verificación:** `pnpm lint && pnpm typecheck && pnpm test && pnpm build && pnpm test:e2e` (web) y `uv run ruff check . && uv run mypy . && uv run pytest` (back)

## Cierre

- [x] **T-AUTH-INT-02**: Actualizar `specs/api-contracts.md` (endpoints a `Implementado`), los README de ambas aplicaciones (Mailpit, variables nuevas, `api:types`, `test:e2e`) y el estado de la feature.
  - **Aplicación:** `shared`
  - **Dependencias:** T-AUTH-QA-05
  - **Prioridad:** Media
  - **Criterios de aceptación:** Los comandos documentados funcionan tal como están escritos
  - **Verificación:** Revisión manual de documentos

### Bloqueos para producción (fuera de estas tareas)

PA-AUTH-10 (proveedor de correo, DP-ARQ-12), PA-AUTH-11 (credenciales de Google), PA-AUTH-12 (texto de la política de datos) y DP-ARQ-06 (hosting). No bloquean ninguna tarea de esta lista.

---

## Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 29/10/2026 | 25 tareas derivadas del plan aprobado. Estado `Aprobado`. | Claude Code |
| 29/10/2026 | Completada INT-01: 21 endpoints en `api-contracts.md` §3 y §4 (convenciones, códigos de error y detalle por endpoint) y 10 tablas en `database.md` §5.1. Comprobación automática: plan, registro, detalle y esquema coinciden. Se precisaron en el plan `google_enabled` y `/perfil?reauth=ok`. | Claude Code |
| 29/10/2026 | Completada BE-01: dependencias instaladas (`httpx` convive con `httpx2`, que usa el `TestClient`), `finora.clock` (`SystemClock`, `FrozenClock`, `get_clock`), `Settings` con correo, Google, clave MFA (AES-256 en base64, obligatoria fuera de `development`; temporal en `development`), política y proxy; `environment` limitado a `development`, `test` y `production`. `test_config.py` y `test_clock.py`: 22/22; suite backend 73/73; ruff y mypy sin errores. | Claude Code |
| 29/10/2026 | Completada BE-02: Mailpit `v1.32.0` en `compose.yaml`; `finora.email` (`OutgoingEmail`, `EmailSender`, `SmtpEmailSender` con STARTTLS obligatorio si hay credenciales, `InMemoryEmailSender`, `get_email_sender`) y 4 plantillas es-CO. `test_email_sender.py`: 21/21, incluido el envío real con STARTTLS y credenciales contra Mailpit; suite backend 94/94. Nuevo marcador `mailpit`. En Fedora (SELinux), el certificado de prueba se monta con la opción `z`. | Claude Code |
| 29/10/2026 | Completada DB-01: modelos en `finora.auth.models`, registro en `finora.db.models`, dependencia `get_db_session`, migración `0001_auth` (sube, baja y vuelve a subir; `alembic check` sin diferencias) y `tests/conftest.py` compartido (PostgreSQL por sesión, transacción revertida por prueba, `client` con BD, reloj, correo y configuración de prueba; marcador `db` automático). `test_db.py` usa ahora las fixtures compartidas. `test_auth_schema.py`: 29/29; suite backend 123/123; sin Docker, 84 pruebas con `-m "not db and not mailpit"`. | Claude Code |
| 29/10/2026 | Completada BE-03: `finora.auth.passwords` (Argon2id, NFKC, política RN-AUTH-02, hash de relleno, `needs_rehash`, `normalize_email`) y lista local de 100.000 contraseñas comunes de 12 o más caracteres. `test_passwords.py`: 19/19; ruff y mypy sin errores. Cambio de la lista pendiente de aprobación (ver historial del plan). | Claude Code |
| 30/10/2026 | Completada BE-04: `finora.auth.tokens` (`issue_token`, `consume_token` con un solo `UPDATE … RETURNING`, `TOKEN_TTL` de 24 h y 1 h, hash SHA-256 de 256 bits aleatorios). `test_tokens.py`: 12/12, incluida la prueba con 8 hilos y transacciones reales (solo un consumo tiene éxito); suite backend 154/154. | Claude Code |
| 30/10/2026 | Completada BE-05: `finora.auth.sessions` (etapas, 30 min de inactividad, 7 días máximo, parciales de 5 min, renovación como máximo por minuto, rotación, revocación, cookie), `finora.auth.csrf` (Content-Type JSON, Origin y `X-CSRF-Token`), `finora.auth.dependencies` (`optional_session`, `require_session`, `CurrentUser`, `require_recent_reauth`, `verify_request_origin`), middleware `Cache-Control: no-store` en `/auth` y `/me`, y `ApiProblemError` en `errors.py` (RFC 9457 con `code`). Las carpetas de pruebas pasan a ser paquetes para admitir nombres repetidos entre `unit/` e `integration/`. `test_sessions.py` (unit + integración): 37/37; suite backend 191/191. | Claude Code |
| 30/10/2026 | Completada BE-06: `finora.auth.rate_limit` (5 fallos por cuenta en 15 min con clave = hash del correo; límites por IP configurables `FINORA_RATE_LIMIT_LOGIN_PER_IP` y `FINORA_RATE_LIMIT_EMAIL_PER_IP`; `Retry-After` según el intento más antiguo; IP real solo con proxy de confianza; purga) y `finora.auth.events` (14 tipos de evento, sin datos sensibles). `test_rate_limit.py` y `test_security_events.py`: 21/21; suite backend 212/212. | Claude Code |
| 30/10/2026 | Completada BE-07: `finora.auth.schemas`, `finora.auth.service` (registro, verificación, reenvío, inicio y cierre de sesión) y router `/api/v1/auth` (6 endpoints). Anti-enumeración: mismas respuestas, hash de contraseña calculado siempre en el registro y verificación contra hash de relleno en el inicio de sesión. `/auth/login` revoca la sesión anterior sin exigir CSRF (`previous_session`). Contrato: se agrega el código `validation_failed` (422). `test_auth_register.py` y `test_auth_login.py`: 44/44; suite backend 256/256; prueba de humo con uvicorn, PostgreSQL y Mailpit reales. | Claude Code |
| 30/10/2026 | Aplicado el cambio aprobado de `GET /auth/session` (200 con campos en `null` sin sesión). Completada BE-08: `POST /auth/password/forgot` y `/auth/password/reset` (la nueva contraseña se valida antes de gastar el enlace, con `peek_token`; cierra todas las sesiones y avisa por correo), router `/api/v1/me` (`GET`, `PATCH`, `POST /me/password` con límite de 5 fallos y cierre de las demás sesiones, `POST /me/password/setup`). `test_password_recovery.py` y `test_profile.py`: 24/24; suite backend 280/280. | Claude Code |
| 30/10/2026 | Completada BE-09: `finora.auth.mfa` y endpoints `POST /auth/login/mfa`, `/auth/reauthenticate`, `/me/mfa/setup`, `/me/mfa/confirm` y `/me/mfa/disable`. `test_mfa.py` (unitaria e integración) y `test_reauthenticate.py`: 42/42; suite backend 322/322. | Claude Code |
| 30/10/2026 | Completada BE-10 tras aprobar 4 cambios de spec y contrato (consentimiento junto al botón de Google, cuenta sin verificar tomada por Google, código MFA tras reautenticar con Google, `error=google_unverified`), con la migración `0002_google`. `finora.auth.google` y endpoints `GET /auth/google/start`, `GET /auth/google/callback` y `POST /auth/google/link`. `test_google_oidc.py` (cliente real contra respx, sin red) y `test_google_flows.py`: 52/52; suite backend 374/374. El inicio real con Google sigue pendiente de PA-AUTH-11. | Claude Code |
| 30/10/2026 | Completada BE-11: `DELETE /api/v1/me` (confirmación `ELIMINAR`, reautenticación reciente, cascada, eventos sin usuario ni IP, intentos borrados) y `finora.auth.maintenance` (`purge_expired()` y `PeriodicPurge`). `test_delete_account.py` y `test_purge.py`: 14/14; suite backend 388/388. Backend de autenticación completo: 21/21 endpoints. | Claude Code |
| 30/10/2026 | Completada FE-01: dependencias, proxy de Vite, `pnpm api:types`, cliente API (`src/lib/api`), sesión y `ProtectedRoute` (`src/features/auth/session`), router y `QueryClient`. `pnpm test src/lib/api src/features/auth/session`: 20/20; suite web 50/50; `typecheck`, `lint` y `build` en verde. Prueba de humo a través del proxy con el backend real: sesión anónima, registro, verificación, inicio de sesión, CSRF (403 sin token, 204 con token). | Claude Code |
| 30/10/2026 | Completada FE-02: `src/components` con `Button`, `TextField`, `PasswordField`, `OtpField`, `Alert`, `AuthLayout`, `GoogleButton` e íconos SVG en línea. `pnpm test src/components`: 28/28, sin violaciones de axe en ningún componente; pegado verificado en contraseña y código. Suite web 78/78; `typecheck`, `lint` y `build` en verde. La revisión visual queda para QA-04. | Claude Code |
| 30/10/2026 | Completada FE-03: `/registro`, `/registro/revisa-tu-correo`, `/verificar-correo` y `/politica-de-datos`, con validación Zod de ayuda, errores del backend por campo, token retirado de la URL (una sola petición aun con StrictMode) y `no-referrer`. `pnpm test src/features/auth/register src/features/auth/verify-email`: 19/19, sin violaciones de axe; suite web 97/97; `typecheck`, `lint` y `build` en verde. | Claude Code |
| 30/10/2026 | Completada FE-04: `/iniciar-sesion` (Google solo si está configurado, `error=google`/`google_unverified`, avisos de correo confirmado y de sesión vencida, reenvío si el correo no está verificado), `/iniciar-sesion/codigo`, `/iniciar-sesion/vincular-google`, cierre de sesión e inicio provisional protegido. `pnpm test src/features/auth/login`: 19/19, sin violaciones de axe; suite web 114/114 (tres corridas); `typecheck`, `lint` y `build` en verde. | Claude Code |
| 30/10/2026 | Completada FE-05: `/recuperar-contrasena` (respuesta genérica, sin enumeración) y `/restablecer-contrasena` (token retirado de la URL, `no-referrer`, errores RN-AUTH-02 en el campo, VAL-AUTH-09 con enlace para pedir otro, cierre de la sesión local). `pnpm test src/features/auth/password-recovery`: 9/9, sin violaciones de axe; suite web 123/123 (tres corridas); `typecheck`, `lint` y `build` en verde. | Claude Code |
| 30/10/2026 | Completada FE-06: `/perfil` con datos y nombre editable, cambio o creación de contraseña, MFA (QR, confirmación, 10 códigos mostrados una vez con confirmación RF-AUTH-20, desactivación), diálogo de reautenticación (contraseña, Google o solo código) y eliminación con `ELIMINAR`. `pnpm test src/features/auth/profile`: 15/15, sin violaciones de axe; suite web 141/141 (tres corridas); `typecheck`, `lint` y `build` en verde, sin aviso de tamaño. | Claude Code |
| 30/10/2026 | Completada QA-01: Playwright configurado (`playwright.config.ts`, `e2e/backend.sh`, helpers de Mailpit y TOTP) y `e2e/auth.spec.ts` con el flujo completo contra backend, PostgreSQL y Mailpit reales: registro y verificación (CA-AUTH-01), perfil, MFA con reautenticación, inicio de sesión con TOTP y código de recuperación de un solo uso (CA-AUTH-08), cierre de sesión, recuperación con dos dispositivos (CA-AUTH-07: sesiones cerradas, contraseña anterior rechazada, enlace no reutilizable) y eliminación con dos dispositivos (CA-AUTH-10: sesiones inválidas, cuenta inexistente y correo registrable de nuevo). `pnpm test:e2e`: 1/1 en dos corridas (≈2,7 min); suite web 141/141. Que los datos dejen de existir en la base de datos lo verifica `test_delete_account.py` en el backend; la E2E lo comprueba por comportamiento. | Claude Code |
| 30/10/2026 | Completada QA-02: `e2e/accessibility.spec.ts` con axe WCAG 2.2 AA y contraste en todas las páginas de autenticación alcanzables (públicas, con errores, con sesión, perfil, diálogo, QR, códigos de recuperación y código MFA), reflow a 320 px, orden de Tab con foco visible, registro → inicio de sesión → perfil → diálogo (Esc devuelve el foco) → cierre de sesión solo con teclado, y pegado en contraseña y código. Dos hallazgos corregidos (desborde a 320 px en `TextField`; pérdida de foco en `Button` con `loading`). `pnpm test:e2e e2e/accessibility.spec.ts`: 16/16; `pnpm test:e2e` completo: 17/17 en dos corridas (≈4 min); suite web 142/142; `lint` y `build` en verde. | Claude Code |
| 30/10/2026 | Completada QA-03: revisión de seguridad manual de `finora-back` y `finora-web` (a pedido de la usuaria, en lugar de `/security-review`) y auditoría de dependencias sin vulnerabilidades conocidas. Corregidos con aprobación: precaptura de cuentas sin verificar (alto), nombre no verificado en el correo y caracteres de control en el nombre (medio) y `safeRedirect` con `\` (bajo); detalle en el historial del plan. Verificado: backend 395/395 (ruff, mypy), web 155/155 (lint, typecheck, build) y E2E 17/17. | Claude Code |
| 30/10/2026 | Completada INT-02 (a pedido de la usuaria, antes de QA-05): los 21 endpoints de `authentication` pasan a `Implementado` en `specs/api-contracts.md`; README de `finora-back` (Mailpit, tabla de variables, clave de MFA estable, pruebas sin Docker) y de `finora-web` (proxy `FINORA_API_PROXY_TARGET`, `api:types`, requisitos y funcionamiento de `test:e2e`). Comandos comprobados: arranque con `docker compose up -d db mailpit`, migraciones, `uvicorn`, `/docs` y `/api/v1/openapi.json`, generación de la clave de MFA, `pytest -m "not db and not mailpit"` (164) y `pnpm dev`/`lint`. La feature sigue `En progreso` hasta cerrar QA-04 y QA-05; si QA-05 encuentra diferencias en los comandos, se corrigen los README. | Claude Code |
