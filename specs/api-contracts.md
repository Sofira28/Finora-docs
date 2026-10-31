# Contratos API

| Campo | Valor |
|---|---|
| Estado | `Draft` |
| Última actualización | 30/10/2026 |

> Cada endpoint se propone en el `plan.md` de su feature, se aprueba y solo entonces se registra aquí. No se deben implementar endpoints que no figuren en este documento.

## 1. Principios (confirmados)

- Este documento es la fuente de verdad del contrato entre `finora-web` y `finora-back`.
- Ningún contrato cambia en silencio: los cambios se versionan, se aprueban y se comunican a ambas aplicaciones.
- El backend valida todas las entradas, aunque el frontend ya las haya validado.
- Toda operación sobre datos financieros se ejecuta en el contexto del usuario autenticado; nunca se aceptan identificadores de otro usuario.
- Los montos se transmiten con precisión segura (cadena decimal + moneda, DP-API-03).
- Los mensajes de error orientados al usuario se presentan en español (`es-CO`).

## 2. Decisiones resueltas

| ID | Resolución | Fuente |
|---|---|---|
| DP-API-01 | REST + JSON. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-02 | Versión en la ruta: `/api/v1/...`. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-03 | Monto como cadena decimal + código ISO 4217: `{"amount": "125000.00", "currency": "COP"}`. El cliente no opera con montos. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-04 | ISO 8601 en el payload (`2026-10-29`); DD/MM/AAAA solo en presentación. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-05 | RFC 9457 (`application/problem+json`) con mensajes en es-CO. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-06 | Paginación por cursor en listados de movimientos; filtros y ordenamiento se definen por endpoint. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-07 | Cookie de sesión `HttpOnly` + token anti-CSRF en peticiones que modifican datos. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-API-08 | OpenAPI 3.1 generado por FastAPI; tipos del frontend con `openapi-typescript`. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |

## 3. Registro de endpoints

| Feature | Método | Ruta | Estado | Versión |
|---|---|---|---|---|
| `project-setup` | GET | `/api/v1/health` | Implementado (29/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/register` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/verify-email` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/verify-email/resend` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/login` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/login/mfa` | Implementado (30/10/2026) | v1 |
| `authentication` | GET | `/api/v1/auth/google/start` | Implementado (30/10/2026) | v1 |
| `authentication` | GET | `/api/v1/auth/google/callback` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/google/link` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/password/forgot` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/password/reset` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/reauthenticate` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/auth/logout` | Implementado (30/10/2026) | v1 |
| `authentication` | GET | `/api/v1/auth/session` | Implementado (30/10/2026) | v1 |
| `authentication` | GET | `/api/v1/me` | Implementado (30/10/2026) | v1 |
| `authentication` | PATCH | `/api/v1/me` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/me/password` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/me/password/setup` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/me/mfa/setup` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/me/mfa/confirm` | Implementado (30/10/2026) | v1 |
| `authentication` | POST | `/api/v1/me/mfa/disable` | Implementado (30/10/2026) | v1 |
| `authentication` | DELETE | `/api/v1/me` | Implementado (30/10/2026) | v1 |
| `accounts` | GET | `/api/v1/accounts` | Aprobado (30/10/2026), sin implementar | v1 |
| `accounts` | POST | `/api/v1/accounts` | Aprobado (30/10/2026), sin implementar | v1 |
| `accounts` | GET | `/api/v1/accounts/{id}` | Aprobado (30/10/2026), sin implementar | v1 |
| `accounts` | PATCH | `/api/v1/accounts/{id}` | Aprobado (30/10/2026), sin implementar | v1 |
| `accounts` | POST | `/api/v1/accounts/{id}/archive` | Aprobado (30/10/2026), sin implementar | v1 |
| `accounts` | POST | `/api/v1/accounts/{id}/restore` | Aprobado (30/10/2026), sin implementar | v1 |
| `accounts` | DELETE | `/api/v1/accounts/{id}` | Aprobado (30/10/2026), sin implementar | v1 |

### GET `/api/v1/health`

Comprobación de vida. Sin autenticación. No expone versión ni dependencias.

- **200** `{"status": "ok"}`
- Errores: formato RFC 9457 (como toda la API).

---

## 4. Autenticación (`authentication`)

Definidos en [`features/authentication/plan.md`](features/authentication/plan.md) §6. Reglas de negocio en [`features/authentication/spec.md`](features/authentication/spec.md).

### 4.1 Convenciones comunes

- **Cookie de sesión:** `finora_session`, con `HttpOnly`, `Secure`, `SameSite=Lax` y `Path=/`, sin `Max-Age`. La expiración real la controla el servidor: 30 minutos de inactividad y 7 días como máximo (RF-AUTH-07). JavaScript nunca la lee.
- **Etapas de sesión:**

  | Etapa | Qué permite |
  |---|---|
  | `active` | Acceso a la aplicación |
  | `mfa_pending` | Solo `POST /auth/login/mfa` |
  | `link_pending` | Solo `POST /auth/google/link` |

  Las etapas parciales duran 5 minutos. Si una petición protegida llega con una sesión que no está en la etapa que necesita, recibe `401`.
- **CSRF:** toda petición `POST`, `PATCH` o `DELETE` hecha con una sesión exige la cabecera `X-CSRF-Token` con el valor de `GET /auth/session`. Si falta o no coincide, responde `403` con `code: "csrf_failed"`.
- **Origen:** toda petición `POST`, `PATCH` o `DELETE` exige `Content-Type: application/json` y un `Origin` permitido (`FINORA_WEB_URL`). Si no se cumplen, responde `403`.
- **Reautenticación reciente (RF-AUTH-21):** los endpoints marcados con "Reauth" exigen haber hecho `POST /auth/reauthenticate`, o pasado por Google con `intent=reauth`, en los últimos 5 minutos. Si no, responden `403` con `code: "reauthentication_required"`.
- **Anti-enumeración (RNF-AUTH-03):** los endpoints que reciben un correo responden lo mismo, en código y en cuerpo, exista o no la cuenta.
- **Errores:** siguen RFC 9457, con `title` en es-CO y un campo `code` estable para que el frontend decida qué hacer.

  | `code` | Estado | Cuándo |
  |---|---|---|
  | `validation_failed` | 422 | Regla de negocio incumplida en un campo (p. ej. RN-AUTH-02). `errors` incluye `loc`, `type` y `message` en es-CO. Los 422 de formato (Pydantic) solo traen `loc` y `type`. |
  | `invalid_credentials` | 401 | Correo, contraseña o código incorrectos |
  | `email_not_verified` | 403 | Correo sin verificar |
  | `too_many_attempts` | 429 | Límite de intentos superado; incluye `Retry-After` |
  | `invalid_token` | 400 | Enlace vencido, usado o inválido |
  | `session_expired` | 401 | Sesión expirada o inexistente |
  | `csrf_failed` | 403 | Token anti-CSRF ausente o incorrecto |
  | `reauthentication_required` | 403 | Falta la reautenticación reciente |
  | `password_already_set` | 409 | Ya tiene contraseña |
  | `mfa_already_enabled` | 409 | La MFA ya está activa |

- **Respuestas:** las de `/auth/*` y `/me` llevan `Cache-Control: no-store`.

### 4.2 Registro y verificación

#### POST `/api/v1/auth/register`

- **Request:** `{"email": "ana@correo.com", "name": "Ana", "password": "…", "privacy_policy_version": "0"}`
- **202** `{"message": "Te enviamos un correo para confirmar tu cuenta."}`. Responde lo mismo si el correo ya existe; en ese caso se envía un aviso o se reenvía la verificación (VAL-AUTH-04).
- **Errores:** `422` (VAL-AUTH-01 a 03; también si `privacy_policy_version` no es la vigente) y `429`.

#### POST `/api/v1/auth/verify-email`

- **Request:** `{"token": "…"}`
- **204**
- **Errores:** `400 invalid_token`.

#### POST `/api/v1/auth/verify-email/resend`

- **Request:** `{"email": "…"}`
- **202** `{"message": "…"}`. Responde igual exista o no la cuenta.
- **Errores:** `422` y `429`.

### 4.3 Inicio y cierre de sesión

#### POST `/api/v1/auth/login`

- **Request:** `{"email": "…", "password": "…"}`
- **200** `{"next": "app"}`, con una sesión `active`, o `{"next": "mfa"}`, con una sesión `mfa_pending`. En ambos casos se emite una cookie nueva.
- **Errores:** `401 invalid_credentials`, `403 email_not_verified` y `429 too_many_attempts`.

#### POST `/api/v1/auth/login/mfa`

- **Sesión:** `mfa_pending`
- **Request:** `{"code": "123456"}`. Acepta un código TOTP o un código de recuperación.
- **200** `{"next": "app"}`, con una cookie nueva y la sesión `active`.
- **Errores:** `401 invalid_credentials`, `401 session_expired` y `429`.

#### POST `/api/v1/auth/logout`

- **Sesión:** cualquier etapa
- **204**. Borra la sesión y la cookie. Es idempotente: también responde `204` si no hay sesión.

#### GET `/api/v1/auth/session`

- **Sesión:** cualquier etapa
- **200** `{"stage": "active", "csrf_token": "…", "google_enabled": true, "user": {"name": "Ana", "email": "ana@correo.com"}}`. En las etapas parciales, `user` es `null`.
- **200 sin sesión** (o con la sesión vencida): `{"stage": null, "csrf_token": null, "google_enabled": true, "user": null}`. Así la página de inicio de sesión sabe si mostrar el botón de Google sin tener sesión.
- **Errores:** ninguno propio (cambio aprobado el 30/10/2026; antes respondía `401 session_expired` sin sesión).

### 4.4 Google

#### GET `/api/v1/auth/google/start`

- **Query:** `intent=login` (por defecto) o `intent=reauth`. `reauth` exige una sesión `active`. Con `intent=login`, `privacy_policy_version` (la vigente) indica que el usuario vio el aviso de la política junto al botón; solo se usa si se crea una cuenta nueva, y sin ella no se crea ninguna (`error=google`).
- **302** a Google, con PKCE, `state` y `nonce`.
- **Errores:** `404` si no hay credenciales de Google configuradas.

#### GET `/api/v1/auth/google/callback`

- **Query:** `code` y `state`. Este endpoint lo llama Google, no `finora-web`.
- **302** a `FINORA_WEB_URL`, a una de estas rutas:
  - `/`: sesión `active`.
  - `/iniciar-sesion/codigo`: sesión `mfa_pending`.
  - `/iniciar-sesion/vincular-google`: sesión `link_pending`.
  - `/perfil?reauth=ok`: reautenticación completada.
  - `/perfil?reauth=codigo`: Google superado; falta el código MFA con `POST /auth/reauthenticate`.
  - `/perfil?reauth=error`: la reautenticación falló (otra cuenta de Google, cancelación o error).
  - `/iniciar-sesion?error=google_unverified`: la cuenta de Google no tiene el correo verificado (VAL-AUTH-12).
  - `/iniciar-sesion?error=google`: cualquier otro error o cancelación (VAL-AUTH-13).
- **Errores:** ninguno en formato RFC 9457; todos los resultados se comunican con la redirección.

#### POST `/api/v1/auth/google/link`

- **Sesión:** `link_pending`
- **Request:** `{"password": "…"}`
- **200** `{"next": "app"}` o `{"next": "mfa"}`. Google queda vinculado a la cuenta (RF-AUTH-17).
- **Errores:** `401 invalid_credentials`, `401 session_expired` y `429`.

### 4.5 Contraseña y reautenticación

#### POST `/api/v1/auth/password/forgot`

- **Request:** `{"email": "…"}`
- **202** `{"message": "…"}`. Responde igual exista o no la cuenta. Si la cuenta solo usa Google, el enlace sirve para crear su primera contraseña (RF-AUTH-18).
- **Errores:** `422` y `429`.

#### POST `/api/v1/auth/password/reset`

- **Request:** `{"token": "…", "new_password": "…"}`
- **204**. Cierra todas las sesiones del usuario (RF-AUTH-09) y le envía un aviso por correo.
- **Errores:** `400 invalid_token` y `422` (RN-AUTH-02).

#### POST `/api/v1/auth/reauthenticate`

- **Sesión:** `active`
- **Request:** `{"password": "…", "code": "123456"}`. `code` solo se exige si la MFA está activa. Las cuentas sin contraseña se reautentican con `GET /auth/google/start?intent=reauth`; si tienen MFA, completan con `{"code": "123456"}` (sin `password`) en los 5 minutos siguientes a volver de Google.
- **204**
- **Errores:** `401 invalid_credentials` y `429`.

### 4.6 Perfil (`/me`)

Todos exigen una sesión `active`. El usuario siempre sale de la sesión: ningún endpoint recibe un identificador de usuario (RN-AUTH-08).

#### GET `/api/v1/me`

- **200** `{"name": "Ana", "email": "ana@correo.com", "has_password": true, "google_linked": false, "mfa_enabled": false}`

#### PATCH `/api/v1/me`

- **Request:** `{"name": "Ana María"}`
- **200**, con el perfil completo, igual que en `GET /me`.
- **Errores:** `422` (RN-AUTH-03).

#### POST `/api/v1/me/password`

- **Request:** `{"current_password": "…", "new_password": "…"}`
- **204**. Cierra las demás sesiones (RF-AUTH-09) y envía un aviso por correo.
- **Errores:** `401 invalid_credentials`, `422` y `429`.

#### POST `/api/v1/me/password/setup`

- **202**. Envía un enlace para crear la primera contraseña (RF-AUTH-18).
- **Errores:** `409 password_already_set`.

#### POST `/api/v1/me/mfa/setup` · Reauth

- **200** `{"secret": "BASE32…", "otpauth_uri": "otpauth://totp/FINORA:ana%40correo.com?secret=…&issuer=FINORA"}`. La MFA queda pendiente hasta confirmarla. Si se vuelve a llamar antes de confirmar, el secreto anterior se reemplaza.
- **Errores:** `403 reauthentication_required` y `409 mfa_already_enabled`.

#### POST `/api/v1/me/mfa/confirm`

- **Request:** `{"code": "123456"}`
- **200** `{"recovery_codes": ["ABCDE-FGHIJ", …]}`, con exactamente 10 códigos. Solo se muestran esta vez.
- **Errores:** `422`, si el código es incorrecto o no hay una configuración pendiente.

#### POST `/api/v1/me/mfa/disable` · Reauth

- **Request:** `{"code": "123456"}`
- **204**. Borra el secreto y los códigos de recuperación.
- **Errores:** `403 reauthentication_required` y `422`.

#### DELETE `/api/v1/me` · Reauth

- **Request:** `{"confirmation": "ELIMINAR"}`
- **204**. Borra al usuario y, en cascada, todos sus datos; cierra todas sus sesiones y borra la cookie. No es reversible (RN-AUTH-07).
- **Errores:** `403 reauthentication_required` y `422` (si la confirmación no coincide).

---

## 5. Cuentas y tarjetas (`accounts`)

Definidos en [`features/accounts/plan.md`](features/accounts/plan.md) §6. Reglas de negocio en [`features/accounts/spec.md`](features/accounts/spec.md).

### 5.1 Convenciones

- Todos exigen sesión `active`; `POST`, `PATCH` y `DELETE` exigen además CSRF y `Origin` (§4.1).
- Solo se ven y modifican las cuentas del usuario de la sesión. Una cuenta de otro usuario o inexistente responde `404` con `code: "account_not_found"` y el título "No encontramos esa cuenta.".
- Montos: cadena decimal en COP. La entrada acepta de 0 a 2 decimales (`"1250000"`, `"1250000.5"`); la salida siempre trae 2 (`"1250000.50"`). Un monto como número JSON se rechaza con `422`.
- Los totales los calcula solo el backend.

  | `code` | Estado | Cuándo |
  |---|---|---|
  | `account_not_found` | 404 | Cuenta inexistente o de otro usuario |
  | `account_name_taken` | 409 | Ya existe una cuenta del usuario con ese nombre (sin distinguir mayúsculas) |
  | `account_limit_reached` | 409 | El usuario ya tiene 100 cuentas |
  | `account_has_movements` | 409 | Editar el saldo inicial o eliminar una cuenta con movimientos |
  | `validation_failed` | 422 | Regla incumplida en un campo, con `errors[].message` en es-CO |

### 5.2 Modelo `Account`

```json
{
  "id": "0193…",
  "name": "Tarjeta Órbita",
  "type": "credit_card",
  "institution": "Banco Andino",
  "opening_balance": "820000.00",
  "balance": "820000.00",
  "has_movements": false,
  "archived": false,
  "credit_card": {
    "credit_limit": "6000000.00",
    "available_credit": "5180000.00",
    "usage_percent": 14,
    "over_limit": false,
    "statement_day": 28,
    "payment_due_day": 18
  },
  "created_at": "2026-10-30T15:00:00Z"
}
```

- `type`: `bank`, `wallet`, `cash`, `savings`, `investment` o `credit_card`. No cambia después de crear la cuenta.
- `balance`: saldo actual (hoy igual a `opening_balance`; con `transactions`, más los movimientos). En tarjetas es la deuda.
- `credit_card`: `null` si no es tarjeta. `available_credit` = cupo − deuda, mínimo 0 (un saldo a favor lo aumenta); `usage_percent` entero redondeado al par más cercano; `over_limit` si la deuda supera el cupo.

### 5.3 Endpoints

#### GET `/api/v1/accounts`

- **200** `{"accounts": [Account], "totals": {"available_balance": "2890000.00", "net_worth": "14070000.00", "credit_card_debt": "820000.00"}}`. Incluye activas y archivadas (`archived: true`); los totales solo cuentan las activas (RN-ACC-03 a 05 y 09).

#### POST `/api/v1/accounts`

- **Request:** `{"name": "Nómina", "type": "bank", "institution": "Banco Andino", "opening_balance": "2800000"}`. Con `type: "credit_card"` también `credit_limit`, `statement_day` y `payment_due_day` (1 a 31), obligatorios; en otros tipos están prohibidos.
- **201** `Account`.
- **Errores:** `409 account_name_taken`, `409 account_limit_reached` y `422`.

#### GET `/api/v1/accounts/{id}`

- **200** `Account`. **Errores:** `404`.

#### PATCH `/api/v1/accounts/{id}`

- **Request:** cualquier subconjunto de `name`, `institution` (`null` la borra), `opening_balance` y, en tarjetas, `credit_limit`, `statement_day` y `payment_due_day`. `type` u otros campos → `422`.
- **200** `Account`.
- **Errores:** `404`, `409 account_name_taken`, `409 account_has_movements` (si cambia `opening_balance`) y `422`.

#### POST `/api/v1/accounts/{id}/archive` y `/restore`

- **Request:** `{}`. **200** `Account`. Idempotentes. **Errores:** `404`.

#### DELETE `/api/v1/accounts/{id}`

- **204**. Definitivo.
- **Errores:** `404` y `409 account_has_movements`.
