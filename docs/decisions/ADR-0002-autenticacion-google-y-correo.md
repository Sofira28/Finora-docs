# ADR-0002: Inicio de sesión con Google, envío de correos y estado de autenticación

- **Estado:** Aceptada (29/10/2026)
- **Fecha:** 29/10/2026
- **Specs relacionadas:** `specs/features/authentication/spec.md` (RF-AUTH-02, 05, 07, 08, 17 a 19; RNF-AUTH-03 a 05; PA-AUTH-10, 11), `specs/architecture.md` (DP-ARQ-05), `specs/security.md` (DP-SEC-01, 05, 06), [ADR-0001](ADR-0001-stack-tecnologico.md)

## Contexto

ADR-0001 definió una autenticación propia en `finora-back`: correo y contraseña con Argon2id, sesión de servidor en cookie `HttpOnly`/`Secure`/`SameSite=Lax` y token anti-CSRF. La spec aprobada de `authentication` amplía ese alcance con tres necesidades que ADR-0001 no cubre:

1. **Inicio de sesión con Google** (RF-AUTH-05), con vinculación segura a cuentas existentes (RF-AUTH-17) y MFA de FINORA también al entrar con Google (RF-AUTH-19).
2. **Envío de correos transaccionales** para verificar el correo y recuperar la contraseña (RF-AUTH-02, RF-AUTH-08). El proveedor de producción no está decidido (PA-AUTH-10).
3. **Estado de autenticación en el servidor**: sesiones con expiración por inactividad y máxima (RF-AUTH-07), revocación de todas las sesiones (RF-AUTH-09, RN-AUTH-07) y limitación de intentos (RNF-AUTH-04).

Restricciones:

- Los tokens de sesión nunca son accesibles desde JavaScript (ADR-0001).
- Las respuestas no deben revelar si un correo está registrado, tampoco por su tiempo de respuesta (RNF-AUTH-03).
- El hosting no está decidido (DP-ARQ-06): la solución no puede depender de un servicio de una nube concreta.
- Los datos personales se minimizan y se quedan en la plataforma siempre que sea posible (Ley 1581 de 2012).

## Decisión

### 1. Google con OpenID Connect gestionado por `finora-back`

- `finora-back` implementa el flujo **Authorization Code con PKCE** de OpenID Connect contra Google, con `state` y `nonce`. Se usa la librería **Authlib**.
- El navegador no maneja tokens de Google: `finora-web` solo redirige a un endpoint del backend que inicia el flujo. Google devuelve al usuario a un *callback* del backend, que valida el ID token (firma, `iss`, `aud`, `exp`, `nonce` y `email_verified`).
- Al terminar, el backend crea **la misma sesión de servidor** que el inicio con contraseña. Desde ese punto no hay diferencia entre ambos métodos.
- La identidad de Google se guarda por su `sub` (identificador estable de Google), no por el correo.
- Cuando el usuario debe completar un paso más, el backend crea una **sesión parcial** de corta duración que solo permite ese paso, no el acceso a la aplicación. Los dos casos son la contraseña de FINORA para vincular una cuenta existente (RF-AUTH-17) y el código MFA (RF-AUTH-19).
- Solo se piden los *scopes* `openid email profile`. No se guardan tokens de acceso ni de actualización de Google.
- Las credenciales del cliente OAuth se leen de variables de entorno y nunca se guardan en el repositorio (PA-AUTH-11).

### 2. Envío de correos detrás de un puerto propio

- `finora-back` define una interfaz `EmailSender` y las plantillas de correo (en es-CO). La lógica de autenticación solo conoce esa interfaz.
- El primer adaptador es **SMTP**, porque sirve para desarrollo y para casi cualquier proveedor transaccional.
- En desarrollo y en las pruebas de integración se usa **Mailpit**, un servidor SMTP de pruebas con interfaz web. Se agrega como servicio en `finora-back/compose.yaml`. Ningún correo sale a internet.
- El proveedor de producción queda como decisión pendiente (**DP-ARQ-12**, ligada a PA-AUTH-10 y al hosting, DP-ARQ-06). Si ese proveedor exige su propia API HTTP, se agrega otro adaptador sin cambiar la lógica de autenticación.
- El envío se hace **fuera de la respuesta HTTP** (tareas en segundo plano de FastAPI). Así el tiempo de respuesta es el mismo si el correo existe o no (RNF-AUTH-03), y la petición no depende de un proveedor lento.

### 3. Sesiones, tokens y límites en PostgreSQL

- Las sesiones, los tokens de verificación y recuperación (guardados como hash) y los contadores de intentos fallidos se guardan en **PostgreSQL**, sin agregar Redis ni otro almacén.
- La cookie solo lleva un identificador de sesión aleatorio. En la base de datos se guarda su hash, junto con `última_actividad` y `expira_en`, para aplicar las dos expiraciones (30 minutos y 7 días).
- Para revocar sesiones basta con borrar filas: cerrar sesión, restablecer la contraseña o eliminar la cuenta.
- La limitación de intentos usa ventanas de tiempo guardadas en una tabla. Los registros vencidos se limpian periódicamente.

## Alternativas consideradas

| Alternativa | Pros | Contras | Motivo del descarte |
|---|---|---|---|
| **Google Identity Services en el frontend** (el botón de Google entrega un ID token a `finora-web`, que lo envía al backend) | Integración rápida con el botón oficial | El token pasa por JavaScript; depende de un script de terceros en la página (CSP más amplia); el flujo se reparte entre las dos aplicaciones | Contradice la regla de no exponer tokens al navegador y complica la CSP pendiente |
| **Proveedor de autenticación gestionado** (Auth0, Clerk, Keycloak, Supabase Auth) | Google, MFA y correos ya resueltos | Costo y dependencia externa; datos personales fuera de la plataforma; dos modelos de sesión conviviendo con la ya implementada | Ya reevaluado en ADR-0001; la spec aprobada se cubre con poco código adicional |
| **Elegir ya un proveedor de correo de producción** (SES, Postmark, Resend, SendGrid) | Sin decisión pendiente | Depende del hosting, de costos y de condiciones de tratamiento de datos aún sin definir | Se pospone sin bloquear el desarrollo gracias a `EmailSender` y Mailpit |
| **Cola de correos con tabla *outbox* y un proceso trabajador** | Ningún correo se pierde si el servidor se reinicia | Un proceso más que desplegar y operar sin hosting definido | Para el MVP basta con tareas en segundo plano: el usuario puede pedir el correo de nuevo. Se reevalúa con el hosting |
| **Redis para sesiones y limitación de intentos** | Rápido; expiración automática de claves | Un servicio más en desarrollo y producción; para el volumen inicial, PostgreSQL alcanza | Complejidad sin beneficio en esta etapa |
| **Tokens JWT sin estado en lugar de sesión de servidor** | Sin consulta a la base por petición | No permiten revocar sesiones al instante (RF-AUTH-09, RN-AUTH-07) ni la expiración por inactividad sin estado adicional | Contradice ADR-0001 y la spec |

## Consecuencias

**Positivas**

- Un único modelo de sesión para ambos métodos de acceso: la autorización y el aislamiento por usuario no cambian según el método.
- Ningún token de Google ni de sesión llega a JavaScript.
- El desarrollo y las pruebas de punta a punta de verificación y recuperación funcionan sin proveedor real ni conexión a internet.
- Cambiar de proveedor de correo es reemplazar un adaptador.
- Sin servicios de infraestructura nuevos en producción más allá de PostgreSQL.

**Negativas y riesgos**

- Más código de seguridad propio: el *callback* de OIDC, la vinculación de cuentas y las sesiones parciales requieren pruebas de casos de abuso y una revisión de seguridad antes de producción.
- Un correo puede perderse si el servidor se reinicia mientras lo envía. Se mitiga con el reenvío (RF-AUTH-03) y se reevalúa con la cola *outbox*.
- Cada petición autenticada consulta y actualiza la sesión en PostgreSQL. Es aceptable para el volumen inicial; se mide antes de optimizar.
- Para salir a producción hacen falta un proyecto en Google Cloud con su pantalla de consentimiento (PA-AUTH-11) y el proveedor de correo (DP-ARQ-12).
- Se añade un contenedor (Mailpit) al entorno local.
