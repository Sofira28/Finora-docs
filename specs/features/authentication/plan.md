# Plan técnico: Autenticación y perfiles de usuario

| Campo | Valor |
|---|---|
| Identificador | `FEAT-AUTH` |
| Estado | `Aprobado` |
| Especificación | [`spec.md`](spec.md) (Aprobado, 29/10/2026) |
| Tareas | [`tasks.md`](tasks.md) |
| Última actualización | 30/10/2026 |

> Este plan solo puede aprobarse cuando la especificación referenciada esté en estado `Aprobado`.
> Arquitectura: [ADR-0001](../../../docs/decisions/ADR-0001-stack-tecnologico.md) y [ADR-0002](../../../docs/decisions/ADR-0002-autenticacion-google-y-correo.md), ambos aceptados.

---

## 1. Contexto de la arquitectura actual

- `project-setup` está completada.
- **`finora-back`:**
  - FastAPI con router `/api/v1` (solo `GET /api/v1/health`) y errores RFC 9457 en `errors.py`.
  - `Settings` con prefijo `FINORA_` leído de `.env`.
  - SQLAlchemy 2 **síncrono** con psycopg 3 y `Base` con convención de nombres.
  - Alembic inicializado, sin migraciones.
  - Pruebas con pytest y Testcontainers. Todavía no hay `conftest.py` compartido.
- **`finora-web`:**
  - React 19 + Vite 8, con una página mínima (`App.tsx`), tokens CSS, Inter y formateadores es-CO.
  - Todavía no tiene React Router, TanStack Query, React Hook Form, Zod ni Playwright. ADR-0001 los aprueba y se instalan con esta feature, que es la primera que los usa.
- **Diseño:** Claude Design no incluye pantallas de autenticación. Por CD-04 (resuelto), se derivan de los tokens existentes.

## 2. Solución propuesta

El módulo `auth` de `finora-back` implementa todo el flujo:

- Contraseñas con Argon2id.
- Inicio con Google por OpenID Connect.
- MFA con TOTP.
- Correos detrás de `EmailSender`.

El estado (sesiones, tokens de un solo uso, intentos y flujos OAuth) vive en PostgreSQL, como define ADR-0002.

La sesión tiene una **etapa**:

| Etapa | Qué permite |
|---|---|
| `active` | Acceso a la aplicación |
| `mfa_pending` | Solo enviar el código MFA |
| `link_pending` | Solo vincular Google con la contraseña |

Así se cumplen RF-AUTH-12, 17 y 19 con una sola tabla y una sola cookie.

`finora-web` agrega rutas públicas de autenticación y una ruta protegida de perfil. Nunca maneja tokens: la cookie es `HttpOnly` y el frontend solo reenvía el token anti-CSRF en una cabecera.

### 2.1 Diferencia con la spec (aprobada)

**DIF-AUTH-01** (aprobada el 29/10/2026; la spec ya se actualizó con RF-AUTH-13, 16 y el nuevo RF-AUTH-21). RF-AUTH-13 (desactivar MFA) y RF-AUTH-16 (eliminar la cuenta) piden "confirmar la contraseña". Un usuario que solo entra con Google no tiene contraseña (RF-AUTH-18 le permite crearla, pero no lo obliga).

Propuesta: definir una **reautenticación reciente** que sirva para todas las acciones sensibles.

| Tipo de cuenta | Cómo se reautentica |
|---|---|
| Con contraseña | Contraseña + código MFA si está activa |
| Solo con Google | Volver a pasar por Google + código MFA si está activa |

La reautenticación vale 5 minutos en la sesión actual.

## 3. Cambios en el frontend (`finora-web`)

**Dependencias nuevas:**
- `react-router`, `@tanstack/react-query`, `react-hook-form`, `zod`, `@hookform/resolvers`.
- `qrcode`, para mostrar como código QR el `otpauth_uri` de la configuración de MFA. Se genera en el navegador; el secreto no sale a ningún servicio externo.
- `openapi-typescript` (desarrollo) y `@playwright/test` (desarrollo).

**Páginas y rutas** (textos en es-CO):

| Ruta | Página | Acceso |
|---|---|---|
| `/registro` | Registro (nombre, correo, contraseña, aceptación de la política) | Pública |
| `/registro/revisa-tu-correo` | Confirmación de envío y reenvío | Pública |
| `/verificar-correo?token=` | Verifica el correo y redirige al inicio de sesión | Pública |
| `/iniciar-sesion` | Correo y contraseña, y botón de Google | Pública |
| `/iniciar-sesion/codigo` | Código MFA o código de recuperación | Sesión `mfa_pending` |
| `/iniciar-sesion/vincular-google` | Contraseña para vincular Google | Sesión `link_pending` |
| `/recuperar-contrasena` | Solicitud del enlace | Pública |
| `/politica-de-datos` | Política de tratamiento de datos (provisional, versión `0`, PA-AUTH-12) | Pública |
| `/restablecer-contrasena?token=` | Nueva contraseña (también crea la primera, RF-AUTH-18) | Pública |
| `/perfil` | Datos, contraseña, MFA y eliminación de cuenta | `active` |
| `/` | Página de inicio provisional (la actual `App`) hasta que exista `dashboard` | `active` |

**Componentes reutilizables** (`src/components/`):
- `TextField` y `PasswordField`: mostrar u ocultar, pegado permitido, `autocomplete` correcto.
- `OtpField`: un solo campo con `inputmode="numeric"` y `autocomplete="one-time-code"`.
- `Button`, `Alert` (errores con `role="alert"`), `AuthLayout` y `GoogleButton`.

**Específicos de página:** van en `src/features/auth/`, separados de los componentes reutilizables.

**Datos y estado:**
- **Cliente HTTP** (`src/lib/api/`):
  - Usa `fetch` con `credentials: 'same-origin'`.
  - Agrega la cabecera `X-CSRF-Token` en los métodos que modifican datos.
  - Convierte las respuestas `application/problem+json` en un error tipado.
  - Ante un `401` en una ruta protegida, redirige a `/iniciar-sesion`.
- **Tipos:** se generan desde `/api/v1/openapi.json` con `pnpm api:types`.
- **Estado de sesión:** TanStack Query con la consulta `GET /auth/session`. Un `ProtectedRoute` decide si muestra la ruta o redirige.
- **Formularios:** React Hook Form + Zod, solo como ayuda al usuario. Replican RN-AUTH-02 y RN-AUTH-03, pero el backend sigue siendo el que decide.
- **Proxy de Vite:** en desarrollo, `/api` apunta a `http://localhost:8000`. Así frontend y backend comparten origen y no hace falta CORS.

**Tokens en la URL** (verificación y restablecimiento):
- Al cargar la página, se quitan de la barra de direcciones con `history.replaceState`.
- La página usa `<meta name="referrer" content="no-referrer">`.

**Referencia visual (CD-04, resuelto):**
- Las pantallas usan los tokens de `src/styles/tokens.css` (fondo cósmico, acentos violeta y lavanda), Inter y los estilos de `Card`, `Button` y `TextField` descritos en `docs/guides/claude-design.md`.
- Se revisan contigo al terminar las páginas.
- Los íconos necesarios (mostrar u ocultar la contraseña, logo de Google) van como SVG en línea, sin decidir CD-08.

## 4. Cambios en el backend (`finora-back`)

```text
src/finora/
├── clock.py                  ← Clock inyectable (UTC) para probar expiraciones
├── auth/
│   ├── models.py             ← tablas de §5
│   ├── passwords.py          ← Argon2id, política RN-AUTH-02, lista de contraseñas comunes
│   ├── common_passwords.txt  ← lista local (sin llamadas externas)
│   ├── sessions.py           ← crear, renovar, expirar y revocar; etapas
│   ├── tokens.py             ← tokens de un solo uso (verificación, restablecimiento)
│   ├── mfa.py                ← TOTP (pyotp), secreto cifrado (AES-GCM), códigos de recuperación
│   ├── google.py             ← cliente OIDC (Authlib) detrás de la interfaz GoogleOidcClient
│   ├── rate_limit.py         ← ventanas por cuenta y por IP
│   ├── csrf.py               ← token sincronizado con la sesión + verificación de Origin
│   ├── events.py             ← registro de eventos de seguridad (RNF-AUTH-06)
│   ├── service.py            ← casos de uso (registrar, iniciar sesión, vincular, eliminar…)
│   ├── schemas.py            ← modelos Pydantic de request/response
│   └── dependencies.py       ← current_session, current_user, require_recent_reauth
├── email/
│   ├── sender.py             ← EmailSender (Protocol), SmtpEmailSender, InMemoryEmailSender (pruebas)
│   └── templates.py          ← verificación, restablecimiento, aviso de registro duplicado, aviso de cambio
└── api/v1/
    ├── auth.py               ← /auth/*
    └── me.py                 ← /me/*
```

**Dependencias nuevas:**
- `argon2-cffi`, `pyotp`, `authlib`, `httpx` (cliente HTTP de Authlib), `cryptography` y `email-validator`.
- Desarrollo: `respx`, para simular las respuestas de Google.

**Lógica clave:**

- **Anti-enumeración de cuentas (RNF-AUTH-03):**
  - Registro, recuperación y reenvío responden siempre `202` con el mismo cuerpo.
  - Los correos se envían con `BackgroundTasks`.
  - Si el correo no existe, se verifica la contraseña contra un hash de relleno, para que el tiempo de respuesta sea el mismo.
- **Registro duplicado (VAL-AUTH-04):**
  - Si el correo ya está verificado, se envía un aviso a su dueño.
  - Si no está verificado, el nuevo registro reemplaza nombre y contraseña, guarda el consentimiento y envía un enlace nuevo, que invalida los anteriores (QA-03: evita la precaptura de la cuenta).
  - El correo de verificación saluda sin el nombre: aún nadie demostró ser dueño de la dirección.
- **Sesión:**
  - El identificador es aleatorio (32 bytes) y en la base solo se guarda su hash SHA-256.
  - `last_activity_at` se actualiza como máximo una vez por minuto, para no escribir en cada petición.
  - Al iniciar sesión o cambiar de etapa se crea una sesión nueva y se borra la anterior (RNF-AUTH-02).
  - Las sesiones parciales expiran a los 5 minutos.
- **Limitación de intentos (RNF-AUTH-04):**
  - Cada intento fallido se registra con su clave: `account:<hash del correo>` o `ip:<IP>`.
  - El límite por cuenta es de 5 fallos en 15 minutos.
  - Los límites por IP son configurables: por defecto, 20 intentos por 15 minutos en inicio de sesión, y 10 por hora en registro y recuperación.
  - Detrás de un proxy, la IP solo se toma de `X-Forwarded-For` si se configura (depende del hosting, DP-ARQ-06).
- **MFA:**
  - El secreto TOTP se cifra con AES-GCM usando `FINORA_MFA_ENCRYPTION_KEY`.
  - Se acepta una ventana de ±1 intervalo y no se acepta dos veces el mismo código.
  - Los 10 códigos de recuperación tienen 10 caracteres en base32 y se guardan con hash.
- **Eliminación (RN-AUTH-07):**
  - Se hace con un `DELETE` del usuario; todas las claves foráneas tienen `ON DELETE CASCADE`.
  - Los eventos de seguridad conservan el evento, pero se desvinculan del usuario (`SET NULL`), como define SUP-AUTH-04.

**Configuración nueva** (`.env.example`):
- `FINORA_WEB_URL`
- `FINORA_SMTP_HOST`, `FINORA_SMTP_PORT`, `FINORA_SMTP_USERNAME`, `FINORA_SMTP_PASSWORD` y `FINORA_EMAIL_FROM`
- `FINORA_GOOGLE_CLIENT_ID` y `FINORA_GOOGLE_CLIENT_SECRET`
- `FINORA_MFA_ENCRYPTION_KEY`
- `FINORA_PRIVACY_POLICY_VERSION`
- `FINORA_TRUSTED_PROXY`

Si faltan las credenciales de Google, `GET /auth/session` devuelve `google_enabled: false`, el botón de Google se oculta y `GET /auth/google/start` responde `404`. Así se puede desarrollar sin PA-AUTH-11.

**`compose.yaml`:** se agrega el servicio `mailpit`, con SMTP en `127.0.0.1:1025` e interfaz web en `127.0.0.1:8025`.

## 5. Cambios en la base de datos

Una migración de Alembic (`0001_auth`). Todos los instantes son `timestamptz` en UTC y todas las claves foráneas a `users` tienen `ON DELETE CASCADE`, salvo donde se indica.

| Tabla | Columnas principales | Restricciones / índices |
|---|---|---|
| `users` | `id` (UUIDv7), `email`, `email_verified_at`, `name`, `password_hash` (nullable), `created_at` | Único en `lower(email)`; el correo se guarda normalizado (`email = lower(btrim(email))`); `name` sin espacios en los extremos y de 1 a 100 caracteres |
| `external_identities` | `id`, `user_id`, `provider` (`google`), `subject`, `created_at` | Único en (`provider`, `subject`) |
| `auth_sessions` | `id`, `token_hash` (`bytea`), `user_id`, `stage`, `csrf_token`, `created_at`, `last_activity_at`, `expires_at`, `reauthenticated_at`, `google_reauth_at` (nullable: paso por Google pendiente del código MFA), `pending_google_subject` (nullable) | Único en `token_hash`; índice en `user_id`; `stage` limitado a `active`, `mfa_pending` y `link_pending`; `pending_google_subject` solo y siempre en `link_pending` |
| `auth_tokens` | `id`, `user_id`, `purpose` (`verify_email`, `reset_password`), `token_hash` (`bytea`), `created_at`, `expires_at`, `used_at` | Único en `token_hash`; índice en (`user_id`, `purpose`) |
| `mfa_credentials` | `user_id` (PK), `secret_encrypted` (`bytea`), `confirmed_at` (nullable mientras no se confirma), `last_used_step`, `created_at` | — |
| `mfa_recovery_codes` | `id`, `user_id`, `code_hash`, `used_at` | Índice en `user_id` |
| `oauth_flows` | `id`, `state_hash` (`bytea`), `nonce`, `code_verifier`, `intent` (`login`, `reauth`), `session_id` (nullable, `ON DELETE CASCADE` a `auth_sessions`), `privacy_policy_version` (nullable), `created_at`, `expires_at` | Único en `state_hash`; expira a los 10 minutos |
| `login_attempts` | `id` (`bigint` identidad), `key`, `kind`, `created_at` | Índice en (`key`, `kind`, `created_at`) |
| `data_consents` | `id`, `user_id`, `policy_version`, `accepted_at` | Índice en `user_id` |
| `security_events` | `id` (`bigint` identidad), `user_id` (nullable, **`ON DELETE SET NULL`**), `type`, `ip` (`inet`), `created_at` | Índice en `user_id`, `created_at` |

**RLS:** estas tablas no usan Row-Level Security. Hay dos razones:

1. El inicio de sesión necesita buscar al usuario por su correo antes de saber quién es.
2. Solo el módulo `auth` accede a ellas.

DP-DB-06 (RLS) empieza con la primera tabla de datos financieros (`accounts`). La tabla `users` sí será la referencia de RLS para esas features.

**Limpieza:** una función `purge_expired()` borra los tokens, flujos OAuth, intentos y sesiones vencidos. Mientras no haya hosting definido, se ejecuta en la primera petición a `/auth` o `/me` tras arrancar y luego como máximo una vez por hora (`PeriodicPurge`, una instancia por aplicación).

## 6. Endpoints y contratos API

Todos van bajo `/api/v1`. Los errores siguen RFC 9457.

Convenciones:
- **CSRF:** las peticiones con sesión que modifican datos exigen `X-CSRF-Token`. Todas las peticiones `POST`, `PATCH` y `DELETE` exigen `Content-Type: application/json` y un `Origin` permitido.
- **Columna "Sesión":** "—" significa que no requiere sesión; las etapas se describen en §2.
- **Columna "Reauth":** "Sí" significa que exige reautenticación reciente (DIF-AUTH-01).

| Método | Ruta | Sesión | Reauth | Request | Response | Errores |
|---|---|---|---|---|---|---|
| POST | `/auth/register` | — | — | `{email, name, password, privacy_policy_version}` | `202 {message}` | 422, 429 |
| POST | `/auth/verify-email` | — | — | `{token}` | `204` | 400 |
| POST | `/auth/verify-email/resend` | — | — | `{email}` | `202 {message}` | 422, 429 |
| POST | `/auth/login` | — | — | `{email, password}` | `200 {next: "app" \| "mfa"}` + cookie | 401, 403 (correo sin verificar), 429 |
| POST | `/auth/login/mfa` | `mfa_pending` | — | `{code}` (TOTP o código de recuperación) | `200 {next: "app"}` + cookie nueva | 401, 429 |
| GET | `/auth/google/start` | — | — | query `intent=login\|reauth`, `privacy_policy_version?` | `302` a Google | 404 (sin configurar) |
| GET | `/auth/google/callback` | — | — | query `code`, `state` | `302` a `FINORA_WEB_URL` (`/`, `/iniciar-sesion/codigo`, `/iniciar-sesion/vincular-google`, `/perfil?reauth=ok\|codigo\|error` o `/iniciar-sesion?error=google\|google_unverified`) | — (los errores se comunican en la redirección) |
| POST | `/auth/google/link` | `link_pending` | — | `{password}` | `200 {next: "app" \| "mfa"}` | 401, 429 |
| POST | `/auth/password/forgot` | — | — | `{email}` | `202 {message}` | 422, 429 |
| POST | `/auth/password/reset` | — | — | `{token, new_password}` | `204` | 400, 422 |
| POST | `/auth/reauthenticate` | `active` | — | `{password?, code?}` (sin `password` solo tras volver de Google) | `204` | 401, 429 |
| POST | `/auth/logout` | cualquiera | — | — | `204` y borra la cookie | — |
| GET | `/auth/session` | opcional | — | — | `200 {stage, csrf_token, google_enabled, user?: {name, email}}`; sin sesión, `stage`, `csrf_token` y `user` en `null` | — |
| GET | `/me` | `active` | — | — | `200 {name, email, has_password, google_linked, mfa_enabled}` | 401 |
| PATCH | `/me` | `active` | — | `{name}` | `200` (perfil) | 401, 403, 422 |
| POST | `/me/password` | `active` | — | `{current_password, new_password}` | `204` (cierra las demás sesiones) | 401, 403, 422, 429 |
| POST | `/me/password/setup` | `active` | — | — | `202` (envía el enlace, RF-AUTH-18) | 401, 403, 409 (ya tiene contraseña) |
| POST | `/me/mfa/setup` | `active` | Sí | — | `200 {secret, otpauth_uri}` | 401, 403, 409 (ya activa) |
| POST | `/me/mfa/confirm` | `active` | — | `{code}` | `200 {recovery_codes: string[10]}` | 401, 403, 422 |
| POST | `/me/mfa/disable` | `active` | Sí | `{code}` | `204` | 401, 403, 422 |
| DELETE | `/me` | `active` | Sí | `{confirmation: "ELIMINAR"}` | `204` y borra la cookie | 401, 403, 422 |

**Cookie:** se llama `finora_session` y tiene los atributos `HttpOnly`, `Secure`, `SameSite=Lax` y `Path=/`. No tiene `Max-Age`, así que es una cookie de sesión del navegador; la expiración real la controla el servidor.

Cuando este plan se apruebe, los endpoints se registran en `specs/api-contracts.md` §3 (T-AUTH-INT-01), antes de implementarlos.

## 7. Flujo de datos

```text
Registro:     web ─POST /auth/register─▶ back ─▶ users + data_consents + auth_tokens
                                              └─(segundo plano)─▶ EmailSender ─▶ correo con /verificar-correo?token=…
Verificación: web /verificar-correo ─POST /auth/verify-email─▶ back ─▶ marca email_verified_at, usa el token
Login:        web ─POST /auth/login─▶ back ─┬─ sin MFA ─▶ auth_sessions(active)      ─▶ cookie
                                            └─ con MFA ─▶ auth_sessions(mfa_pending) ─▶ cookie ─▶ POST /auth/login/mfa ─▶ active
Google:       web ─▶ GET /auth/google/start ─▶ oauth_flows ─▶ 302 Google ─▶ GET /auth/google/callback
                 ─▶ valida el ID token ─┬─ identidad conocida ─────────────────▶ active / mfa_pending
                                        ├─ correo nuevo ─▶ crea usuario verificado ─▶ active
                                        └─ correo con cuenta y contraseña ─▶ link_pending ─▶ POST /auth/google/link
Petición:     cookie ─▶ current_session (hash, etapa, expiraciones, CSRF) ─▶ current_user ─▶ endpoint
```

## 8. Consideraciones de seguridad

- **Autenticación:**
  - Argon2id con los parámetros por defecto de `argon2-cffi`; se vuelve a calcular el hash si los parámetros cambian.
  - El ID token de Google se valida completo, y se exige `email_verified=true`.
- **Autorización:**
  - `current_user` sale siempre de la sesión. Ningún endpoint recibe un `user_id` (RN-AUTH-08).
  - Las sesiones parciales solo pasan la dependencia de su propio endpoint.
- **Aislamiento:** el usuario de la sesión es la base del filtro por `user_id` de las features siguientes.
- **Datos sensibles:**
  - Las contraseñas, tokens, códigos y secretos nunca se registran ni se devuelven, salvo el `secret` de configuración de MFA, una sola vez, y los códigos de recuperación, una sola vez.
  - Los tokens se guardan con hash y el secreto TOTP se cifra.
- **CSRF:**
  - Las peticiones con sesión usan un token sincronizado.
  - En las peticiones sin sesión (registro e inicio de sesión), la protección viene de exigir `Content-Type: application/json`, que obliga al navegador a hacer *preflight* entre orígenes, y de verificar `Origin`.
- **Cabeceras:** `Cache-Control: no-store` en `/auth/*` y `/me`.
- **Secretos:** solo en variables de entorno (DP-SEC-03 sigue pendiente para producción).

## 9. Estrategia de validación

- **Frontend (ayuda):**
  - Zod replica el formato del correo, RN-AUTH-02 (longitud) y RN-AUTH-03.
  - La lista de contraseñas comunes **solo** se comprueba en el backend, para no enviarla al navegador.
- **Backend (obligatoria):**
  - Pydantic valida forma y longitudes, y normaliza el correo (RN-AUTH-01).
  - `passwords.validate()` aplica RN-AUTH-02 completa: longitud en caracteres Unicode, lista común y comparación con el correo.
  - Los tokens se validan por hash, propósito, expiración y uso, con un `UPDATE … WHERE used_at IS NULL` atómico para que no se puedan usar dos veces en peticiones simultáneas.

## 10. Manejo de errores

- Se usan los códigos y mensajes es-CO de VAL-AUTH-01 a 13, en formato RFC 9457.
- Se agrega un campo `code` estable (por ejemplo, `email_not_verified` o `too_many_attempts`) para que el frontend elija la acción, como ofrecer el reenvío. Hoy `errors.problem()` acepta campos extra; no hace falta cambiar el formato.
- `429` incluye la cabecera `Retry-After`.
- **Frontend:**
  - Los errores de campo se muestran junto al campo, y los generales en un `Alert` con `role="alert"`.
  - Los botones muestran estado de carga y se desactivan mientras esperan la respuesta.
  - Si falla la red, se muestra "No pudimos conectarnos. Revisa tu conexión e intenta de nuevo."

## 11. Estrategia de pruebas

| Nivel | Alcance | Herramienta | Criterios de aceptación cubiertos |
|---|---|---|---|
| Unitarias (back) | Política de contraseñas, normalización del correo, TOTP y ventana, códigos de recuperación, cifrado del secreto, expiración de sesiones con `Clock` falso | pytest + Hypothesis | CA-AUTH-05, 06 (lógica), RN-AUTH-01 a 06 |
| Integración (back) | Todos los endpoints contra PostgreSQL real, con `InMemoryEmailSender` y un cliente Google falso; idénticos entre correo existente y no existente; concurrencia de tokens; cascada al eliminar; aislamiento entre dos usuarios | pytest + httpx + Testcontainers | CA-AUTH-01 a 11, 13, 14 |
| Integración (back) | Validación del ID token con claves JWK generadas en la prueba (firma, `aud`, `nonce`, `email_verified`) | pytest + respx | CA-AUTH-04 y VAL-AUTH-12 |
| Integración (back) | `SmtpEmailSender` contra Mailpit en un contenedor | Testcontainers | RF-AUTH-02 y 08 |
| Unitarias (web) | Cliente API (CSRF, problem+json, 401), esquemas Zod, formularios y páginas | Vitest + Testing Library | Apoyo a CA-AUTH-01, 07, 08 |
| E2E | Registro → correo en Mailpit → verificación → inicio de sesión → perfil → MFA → cierre de sesión → recuperación → eliminación | Playwright (back + Mailpit reales) | CA-AUTH-01, 07, 08, 10 |
| Accesibilidad | Todas las páginas de autenticación: axe en Vitest y en Playwright (incluye contraste), recorrido con teclado y pegado en los campos | axe-core + Playwright | CA-AUTH-12 |

Google se prueba de punta a punta solo con el cliente falso. El inicio real con Google se verifica de forma manual cuando exista PA-AUTH-11.

## 12. Dependencias y riesgos

| Riesgo / dependencia | Impacto | Mitigación |
|---|---|---|
| CD-04: no hay diseño de las pantallas de autenticación | Las pantallas pueden no encajar con el resto del diseño | Resuelto: se derivan de los tokens existentes y se revisan después de implementarlas |
| PA-AUTH-11: credenciales de Google | El inicio real con Google no se puede probar | Cliente falso en las pruebas; la funcionalidad se apaga sin credenciales |
| PA-AUTH-10 / DP-ARQ-12: proveedor de correo | No hay envío real en producción | SMTP + Mailpit en desarrollo; bloquea solo la salida a producción |
| PA-AUTH-12: texto de la política de datos | No se puede publicar | Página provisional "Política en preparación" con versión `0`; bloquea solo la salida a producción |
| Frontend y backend en orígenes distintos en producción (DP-ARQ-06) | La cookie `SameSite=Lax` y la verificación de `Origin` pueden necesitar ajustes | El proxy de Vite iguala el origen en desarrollo; se revisa al decidir el hosting |
| Código de seguridad propio (ADR-0002) | Vulnerabilidades sutiles | Pruebas de abuso en integración y ejecución de `/security-review` antes de cerrar la feature |
| Lista de contraseñas comunes | Tamaño del paquete | Se usan las 100.000 más comunes (unos 1 MB, licencia MIT, SecLists) |

## 13. Secuencia de implementación

1. **Contrato:** registrar los endpoints de §6 en `specs/api-contracts.md` y las tablas en `specs/database.md`.
2. **Back, base:**
   - `Clock`, configuración nueva y Mailpit en `compose.yaml`.
   - `EmailSender`, con sus pruebas.
3. **Back, datos:** modelos, migración `0001_auth` y `conftest.py` compartido (contenedor, sesión de BD y cliente HTTP).
4. **Back, núcleo:**
   - Contraseñas y tokens.
   - Sesiones, CSRF y dependencias.
   - Limitación de intentos y eventos.
5. **Back, flujos:**
   - Registro, verificación e inicio y cierre de sesión.
   - Recuperación.
   - Perfil y cambio de contraseña.
6. **Back, MFA:** configuración, confirmación, inicio de sesión con código, recuperación, desactivación y reautenticación.
7. **Back, Google:** cliente OIDC, *callback*, vinculación y MFA con Google.
8. **Back, eliminación:** eliminación de la cuenta y purga de registros vencidos.
9. **Web, base:** router, Query, cliente API, tipos generados, proxy y componentes reutilizables.
10. **Web, páginas:** registro y verificación, inicio de sesión y MFA, recuperación y perfil.
11. **E2E y accesibilidad:** Playwright y axe.
12. **Cierre:** revisión de seguridad, README de ambas aplicaciones y verificación de todos los CA.

## 14. Decisiones arquitectónicas

| Decisión | Alternativas consideradas | ADR | Estado |
|---|---|---|---|
| Google OIDC en el backend, `EmailSender`/SMTP/Mailpit y estado en PostgreSQL | Ver ADR | `docs/decisions/ADR-0002-autenticacion-google-y-correo.md` | Aceptada |
| Sesión con etapas (`active`, `mfa_pending`, `link_pending`) en una sola tabla | Cookie temporal firmada para el paso MFA | — (detalle de implementación dentro de ADR-0002) | Aprobada (29/10/2026) |
| Tablas de autenticación sin RLS | RLS también en `users` | — (aplicación de DP-DB-06) | Aprobada (29/10/2026) |
| Reautenticación reciente para acciones sensibles (DIF-AUTH-01, RF-AUTH-21) | Exigir crear una contraseña | — | Aprobada (29/10/2026) |

## 15. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 29/10/2026 | Plan completo según la spec aprobada y ADR-0002. Pasa a `En revisión`. | Claude Code |
| 29/10/2026 | Resueltas DIF-AUTH-01 (reautenticación reciente) y CD-04 (pantallas derivadas de los tokens). | Claude Code |
| 29/10/2026 | Aprobado. Se agrega `qrcode` en el frontend para mostrar el QR de MFA (omitido en la primera versión del plan). | Claude Code |
| 29/10/2026 | T-AUTH-INT-01: se precisan en §6 el campo `google_enabled` de `GET /auth/session` y la redirección `/perfil?reauth=ok` del *callback* de Google, al detallar el contrato. | Claude Code |
| 29/10/2026 | T-AUTH-BE-02: las plantillas de correo quedan en el módulo `email/templates.py` (funciones con texto y HTML) en vez de una carpeta; no cambia el comportamiento. | Claude Code |
| 29/10/2026 | T-AUTH-DB-01: §5 se precisa con lo implementado (`created_at` en `auth_tokens`, `mfa_credentials` y `oauth_flows`; hashes en `bytea`; UUIDv7; restricciones de normalización del correo y del nombre y de coherencia de `link_pending`). Mismo cambio en `specs/database.md` §5.1. | Claude Code |
| 29/10/2026 | T-AUTH-BE-03 (aprobado el 30/10/2026): la lista de contraseñas comunes son las 100.000 más frecuentes de **12 o más caracteres** de SecLists `Pwdb_top-10000000.txt` (1,4 MB, MIT). La lista "top 100.000" original solo tenía 1.421 contraseñas de 12 o más caracteres. | Claude Code |
| 30/10/2026 | Aprobados: lista de contraseñas de BE-03, código `validation_failed` (422) y `GET /auth/session` con `200` y campos en `null` cuando no hay sesión (para que la página de inicio de sesión conozca `google_enabled`). | Claude Code |
| 30/10/2026 | Precisiones de BE-09 (sin cambio de contrato): el secreto TOTP se cifra con AES-256-GCM usando el `id` del usuario como dato asociado; los códigos de recuperación (formato `ABCDE-FGHIJ`, 50 bits) se guardan como HMAC-SHA256 con `FINORA_MFA_ENCRYPTION_KEY` en vez de Argon2 (verificar 10 hashes Argon2 por intento sería muy costoso); el consumo del código TOTP (`last_used_step`) y del de recuperación es atómico; los fallos de `/auth/login/mfa` y `/auth/reauthenticate` suman al límite de 5 fallos por cuenta; `/me/mfa/disable` solo acepta TOTP, no códigos de recuperación. | Claude Code |
| 30/10/2026 | Cambios aprobados en BE-10 (ver historial de la spec): columnas `oauth_flows.privacy_policy_version` y `auth_sessions.google_reauth_at` (migración `0002`); `privacy_policy_version` en `/auth/google/start`; nuevas redirecciones `/perfil?reauth=codigo\|error` y `?error=google_unverified`; `password` opcional en `/auth/reauthenticate` tras volver de Google. Mismo cambio en `api-contracts.md` y `database.md`. | Claude Code |
| 30/10/2026 | Precisiones de BE-10 (internas, sin cambio de contrato): la `redirect_uri` es `{FINORA_WEB_URL}/api/v1/auth/google/callback` (mismo origen gracias al proxy); una cookie `finora_oauth_state` (HttpOnly, `Path=/api/v1/auth/google`, 10 min) ata el flujo al navegador que lo empezó, contra el CSRF de inicio de sesión; la reautenticación pide a Google `max_age=0` y revisa `auth_time` si viene; todas las peticiones a Google pasan por un `httpx.Client` propio: de Authlib solo se usan las funciones que arman la URL y el PKCE (sus clientes HTTP usan `httpx2`), y el ID token se valida con `joserfc`; `mypy` ignora la falta de tipos de `authlib.*`. | Claude Code |
| 30/10/2026 | Precisiones de BE-11: además de desvincular los eventos (`SET NULL`), al eliminar la cuenta se borra la IP de sus eventos y los intentos ligados a su correo (la clave es un hash reversible probando correos), para cumplir SUP-AUTH-04; el evento `account_deleted` se guarda sin usuario ni IP. La limpieza corre en la primera petición de autenticación y no en el arranque, para no depender de la BD al iniciar. Registro del contrato: los 21 endpoints quedan como implementados en `finora-back`. | Claude Code |
| 30/10/2026 | Precisiones de FE-01: versiones instaladas `react-router` 8.5, `@tanstack/react-query` 5, `react-hook-form` 7, `zod` 4, `@hookform/resolvers` 6, `qrcode` 1.5 y `openapi-typescript` 7.13 (declara TypeScript ^5 como dependencia par; funciona con la 6 del proyecto). `.env.example` de `finora-web`: se reemplaza `VITE_API_BASE_URL` (otro origen, contrario al proxy de §3) por `FINORA_API_PROXY_TARGET`, que solo lee Vite. Los tipos generados (`src/lib/api/schema.d.ts`) se versionan para que las pruebas no necesiten el backend. Ante un `401 session_expired`, el cliente borra de la caché todo dato del usuario y la sesión pasa a "sin sesión"; la página de inicio de sesión muestra VAL-AUTH-10 con `consumeSessionExpired()`. Las rutas de FE-03 a FE-06 existen con una página provisional. | Claude Code |
| 30/10/2026 | Precisiones de FE-02: `GoogleButton` es un enlace (`<a href>`) a `/auth/google/start`, no un botón, porque el flujo es una navegación completa; lleva el logo "G" oficial sobre fondo blanco. `PasswordField` usa un botón de alternancia con nombre fijo "Mostrar contraseña" y `aria-pressed`. `OtpField` no impone longitud (el backend tolera espacios y guiones) y con `allowRecoveryCode` acepta letras. `Alert` usa `role="alert"` solo para errores. Logotipo de FINORA en SVG en línea según la guía §4. El helper `expectNoAxeViolations` (`src/test/axe.ts`) desactiva solo `color-contrast`, que se verifica en navegador real (QA-02). | Claude Code |
| 30/10/2026 | Precisiones de FE-03: nueva ruta pública `/politica-de-datos` para la página provisional (el plan la pedía sin ruta). La versión de la política la fija `finora-web` (`PRIVACY_POLICY_VERSION = "0"`, la del texto que muestra); si el backend tiene otra, el `422 policy_not_accepted` se muestra como aviso para recargar. El registro también ofrece Google, con el aviso de la política, si está configurado. `GuestRoute` lleva al inicio si ya hay sesión activa en `/registro`; `/verificar-correo` es pública aunque haya sesión. Tras verificar, se redirige a `/iniciar-sesion` con `state.emailVerified`, que muestra la página de FE-04. | Claude Code |
| 30/10/2026 | Precisiones de FE-04: el destino tras iniciar sesión (`state.from`) se conserva entre etapas (contraseña → código MFA → página original) y solo admite rutas internas (`safeRedirect`, evita redirecciones abiertas). El inicio de sesión solo exige campos no vacíos: un correo mal formado recibe el mismo `401` genérico (VAL-AUTH-05). Tras `POST /auth/login`, `/auth/login/mfa` y `/auth/google/link` se vuelve a pedir `GET /auth/session` para obtener el nuevo token anti-CSRF. La página de inicio provisional protegida (`HomePage`) reemplaza a la `App` de FEAT-SETUP y tiene el botón de cierre de sesión; el cierre borra de memoria los datos del usuario. | Claude Code |
| 30/10/2026 | Precisiones de FE-05: `/recuperar-contrasena` y `/restablecer-contrasena` son públicas aunque haya sesión (el enlace de crear contraseña también se pide desde el perfil). La página de restablecer usa un solo campo de contraseña con botón "Mostrar" (sin campo de confirmación, igual que el registro; WCAG 3.3.7 evita la entrada redundante). Si el backend rechaza la contraseña (`422`), se corrige y reenvía con el mismo enlace, porque el backend valida antes de gastarlo. Tras restablecer, se vuelve a pedir la sesión (el backend cerró todas, RF-AUTH-09) y se lleva a `/iniciar-sesion` con `state.passwordReset`, que muestra un aviso. | Claude Code |
| 30/10/2026 | Precisiones de FE-06: la reautenticación (RF-AUTH-21) se pide cuando una acción responde `403 reauthentication_required`; tras confirmarla, la activación de MFA y la eliminación de la cuenta se repiten solas. La desactivación de MFA no se repite: el backend no acepta reutilizar un código TOTP y la reautenticación ya gastó el vigente, así que se pide el siguiente. Con Google (cuenta sin contraseña) la acción no se puede reanudar tras la redirección: al volver (`reauth=ok`) se avisa que hay 5 minutos para repetirla; con `reauth=codigo` se abre el diálogo solo con el código. Los avisos al perder la sesión (sesión vencida, cuenta eliminada) pasan a una marca de un solo uso (`loginNotice`), porque `ProtectedRoute` también redirige y reemplazaría el estado de navegación. El QR se genera como SVG (sin canvas) y `qrcode` y la página de perfil se cargan bajo demanda (`import()` y `lazy` de react-router) para mantener el bundle inicial por debajo de 500 kB. Nuevo componente `Dialog` sobre `<dialog>` nativo. Se eliminan los marcadores `PendingPage`. | Claude Code |
| 30/10/2026 | Precisiones de QA-01: Playwright (`@playwright/test` 1.64.1, Chromium) con `pnpm test:e2e`. Requisito: `docker compose up -d db mailpit` en finora-back. `e2e/backend.sh` recrea en cada corrida una base de datos propia `finora_e2e` (la de desarrollo no se toca), aplica las migraciones y levanta el backend en el puerto 8001; Vite corre en el 5174 con el proxy hacia él, para no chocar con los servidores de desarrollo. Cada corrida usa una clave de MFA nueva, Google desactivado y límites por IP altos. El correo se lee con la API de Mailpit y los códigos TOTP se generan en la prueba (RFC 6238 con `node:crypto`, sin dependencia nueva); como el backend no reutiliza intervalos, la prueba espera al siguiente cuando hace falta (unos 2,5 minutos en total). Los correos de prueba usan `example.com`: el backend rechaza dominios de uso especial como `.test`. Vitest se limita a `src/`. | Claude Code |
| 30/10/2026 | Precisiones de QA-02: `e2e/accessibility.spec.ts` inyecta el `axe-core` ya instalado (sin dependencia nueva) con las etiquetas WCAG 2.0, 2.1 y 2.2 A/AA, incluido el contraste, y emula `prefers-reduced-motion` para medir el estado final. Además de lo pedido, comprueba reflow a 320 px (WCAG 1.4.10), foco visible en cada parada de Tab (2.4.7) y los atributos `autocomplete` para gestores de contraseñas (3.3.8). `/iniciar-sesion/vincular-google` no se puede alcanzar sin Google (PA-AUTH-11); queda cubierta por axe en Vitest (sin contraste) y comparte componentes y tokens con las demás. Los flujos comunes de la E2E pasan a `e2e/support/flows.ts` y las pruebas de Playwright tienen su propio `tsconfig.e2e.json` (Node y DOM). | Claude Code |
| 30/10/2026 | Hallazgos de QA-02 corregidos: (1) a 320 px las páginas con campo de contraseña se desbordaban 36 px porque el `<input>` imponía su ancho intrínseco a la columna; `TextField` lo anula (`width: 0` con `flex: 1`). (2) `Button` con `loading` usaba `disabled`, que saca el foco del botón y lo deja en `<body>` en todos los formularios (WCAG 2.4.3; al cerrar el diálogo el foco no volvía al botón que lo abrió); ahora usa `aria-disabled` y cancela el clic, lo que también bloquea el envío implícito con Enter. | Claude Code |
| 30/10/2026 | Hallazgos de QA-03 corregidos (revisión manual de ambas aplicaciones y auditoría de dependencias con `pnpm audit` y `pip-audit`, sin vulnerabilidades conocidas): (1) ALTO, precaptura: un registro sobre un correo sin verificar conservaba la contraseña del primero, así que un tercero podía entrar cuando la dueña confirmara el correo; ahora manda el último registro. (2) MEDIO, el nombre sin verificar llegaba al correo de verificación y en texto plano admitía saltos de línea (texto engañoso enviado por FINORA); el correo saluda sin nombre y el nombre (registro, perfil y Google) rechaza o quita caracteres de control. (3) BAJO, `safeRedirect` aceptaba `/\sitio`, que el navegador lee como `//sitio`; ahora rechaza `\` y caracteres de control. | Claude Code |
