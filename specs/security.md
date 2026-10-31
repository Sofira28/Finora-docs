# Seguridad

| Campo | Valor |
|---|---|
| Estado | `Draft` |
| Última actualización | 29/10/2026 |

## 1. Principios (confirmados)

- **Validación obligatoria en el backend**; el cliente nunca es fuente de confianza.
- **Aislamiento de datos por usuario**: ningún usuario puede leer ni modificar datos financieros de otro.
- **Mínimo privilegio** en servicios, credenciales y accesos.
- **Sin secretos en el repositorio**: las credenciales se gestionan por variables de entorno o un gestor de secretos (por definir).
- **Defensa en profundidad**: validación, autorización y auditoría en el backend.

## 2. Autenticación

- ✅ Mecanismo inicial: correo y contraseña con hash Argon2id, gestionado por `finora-back` ([ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md)). Proveedores externos, magic link y passkeys quedan como posibles extensiones.
- ✅ Inicio con Google (OpenID Connect gestionado por `finora-back`): ver [`authentication/spec.md`](features/authentication/spec.md) RF-AUTH-05 y 17 a 19. Arquitectura en ADR-0002 (Propuesta).
- ✅ MFA opcional con TOTP y 10 códigos de recuperación de un solo uso; se exige también al entrar con Google ([`authentication/spec.md`](features/authentication/spec.md) RF-AUTH-10 a 13 y 19).
- ✅ Política de contraseñas: de 12 a 128 caracteres, sin contraseñas comunes ni filtradas, sin reglas de composición (NIST SP 800-63B; [`authentication/spec.md`](features/authentication/spec.md) RN-AUTH-02).
- ✅ Gestión de sesión: sesión de servidor en cookie `HttpOnly`/`Secure`/`SameSite=Lax`; expira tras 30 minutos de inactividad y como máximo a los 7 días; se revocan todas las sesiones al restablecer la contraseña o eliminar la cuenta ([`authentication/spec.md`](features/authentication/spec.md) RF-AUTH-07 y 09).
- ✅ Recuperación de contraseña por enlace de un solo uso al correo verificado, válido 1 hora ([`authentication/spec.md`](features/authentication/spec.md) RF-AUTH-08). Sin recuperación si se pierden el dispositivo MFA y los códigos (RF-AUTH-20).

## 3. Autorización

- ✅ Todo recurso financiero pertenece a un usuario y solo él accede a él.
- ❓ ¿Existen roles adicionales (administrador, soporte, cuentas compartidas o familiares)? **Pendiente**.

## 4. Protección de datos

- 🟡 Cifrado en tránsito (TLS) y en reposo: se asume como requisito; falta detallar la implementación.
- ❓ Cumplimiento normativo colombiano en protección de datos personales: **pendiente de precisar** con asesoría.
- ✅ Eliminación de cuenta inmediata y definitiva, con borrado en cascada de todos los datos del usuario ([`authentication/spec.md`](features/authentication/spec.md) RF-AUTH-16, RN-AUTH-07). ❓ Retención legal de registros: pendiente (PA-AUTH-12).
- ❓ Tratamiento de documentos sensibles (extractos, facturas) en `documents` y `capture-center`: **pendiente**.
- ❓ Datos enviados al proveedor de IA (`ai-assistant`): qué se comparte, anonimización y consentimiento. **Pendiente**.

## 5. Seguridad del frontend

- Objetivo de accesibilidad WCAG 2.2 AA (también aplica a los flujos de autenticación).
- ✅ Sesión en cookie `HttpOnly` (sin tokens accesibles desde JavaScript) y token anti-CSRF en peticiones que modifican datos ([ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md)).
- ❓ Política CSP: **pendiente**.

## 6. Decisiones resueltas

| ID | Resolución | Fuente |
|---|---|---|
| DP-SEC-01 | Autenticación propia en `finora-back`: Argon2id, sesión de servidor en cookie `HttpOnly`/`Secure`/`SameSite=Lax`, token anti-CSRF. MFA (TOTP) y recuperación se detallan en `authentication/spec.md`. | [ADR-0001](../docs/decisions/ADR-0001-stack-tecnologico.md) |
| DP-SEC-02 | MFA opcional (TOTP) y política de contraseñas NIST SP 800-63B. | [`authentication/spec.md`](features/authentication/spec.md) |

## 7. Decisiones pendientes

| ID | Decisión |
|---|---|
| DP-SEC-03 | Gestión de secretos. |
| DP-SEC-04 | Marco normativo aplicable y responsabilidades. |
| DP-SEC-05 | Auditoría y registro de eventos de seguridad. |
| DP-SEC-06 | Rate limiting y protección contra abuso fuera de la autenticación (la de autenticación está en [`authentication/spec.md`](features/authentication/spec.md) RNF-AUTH-04). |
| DP-SEC-07 | Política de datos compartidos con terceros (IA, almacenamiento). |
