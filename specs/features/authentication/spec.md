# Especificación: Autenticación y perfiles de usuario

| Campo | Valor |
|---|---|
| Identificador | `FEAT-AUTH` |
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

Permitir que cada persona cree su cuenta en FINORA, acceda a ella de forma segura y gestione su perfil. Es la base del aislamiento de datos financieros por usuario: todo recurso financiero de las demás features pertenece a un usuario autenticado por esta feature.

## 2. Alcance

- Registro con correo y contraseña. ✅
- Inicio de sesión con correo y contraseña, y con Google. ✅
- Verificación obligatoria del correo antes de acceder. ✅
- Recuperación de contraseña por enlace al correo ("Olvidé mi contraseña"). ✅
- Sesión con expiración por inactividad (30 min) y máxima (7 días). ✅
- Cierre de sesión. ✅
- MFA opcional con aplicación de autenticación (TOTP) y códigos de recuperación. ✅
- Perfil mínimo: nombre (obligatorio) y correo; edición del nombre y cambio de contraseña. ✅
- Eliminación inmediata y definitiva de la cuenta y de todos sus datos. ✅
- Aceptación de la política de tratamiento de datos personales al registrarse. ✅ (Ley 1581 de 2012)

## 3. Fuera de alcance

- Inicio con Apple, passkeys y magic link (posibles extensiones futuras).
- MFA por SMS o correo; MFA obligatoria.
- Teléfono, documento de identidad u otros datos de perfil obligatorios.
- Cambio de correo electrónico de la cuenta. ✅
- Roles adicionales (administrador, soporte) y cuentas compartidas o familiares.
- Exportación de datos previa a la eliminación (posible extensión; ver `settings`).
- Preferencias de la aplicación (tema, notificaciones): feature `settings`.

## 4. Actores y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| Visitante | Persona sin sesión | Registrarse, iniciar sesión, verificar correo, solicitar y completar recuperación de contraseña |
| Usuario pendiente de verificación | Cuenta creada con correo sin verificar | Solo reenviar el correo de verificación; no accede a datos ni funciones de la aplicación |
| Usuario autenticado | Cuenta verificada con sesión activa | Ver y editar su propio perfil, gestionar MFA, cambiar contraseña, cerrar sesión, eliminar su cuenta. Nunca accede a datos de otro usuario |

## 5. Historias de usuario

- **HU-01**: Como visitante, quiero registrarme con mi correo, nombre y una contraseña, para empezar a usar FINORA.
- **HU-02**: Como visitante, quiero entrar con mi cuenta de Google, para no crear otra contraseña.
- **HU-03**: Como usuario nuevo, quiero confirmar mi correo, para demostrar que es mío y poder recuperar el acceso.
- **HU-04**: Como usuario, quiero iniciar y cerrar sesión, para proteger mi información en dispositivos compartidos.
- **HU-05**: Como usuario que olvidó su contraseña, quiero recibir un enlace para crear una nueva, para recuperar el acceso.
- **HU-06**: Como usuario, quiero activar la verificación en dos pasos, para proteger mejor mis datos financieros.
- **HU-07**: Como usuario, quiero editar mi nombre y cambiar mi contraseña, para mantener mi cuenta al día.
- **HU-08**: Como usuario, quiero eliminar mi cuenta y todos mis datos, para ejercer mi derecho a que FINORA no conserve mi información.

## 6. Requisitos funcionales

| ID | Requisito | Prioridad | Certeza |
|---|---|---|---|
| RF-AUTH-01 | El sistema permite registrarse con correo, nombre y contraseña, y aceptar la política de tratamiento de datos. | Alta | ✅ |
| RF-AUTH-02 | Tras el registro, el sistema envía un correo con un enlace de verificación de un solo uso. | Alta | ✅ |
| RF-AUTH-03 | Una cuenta con correo sin verificar no puede acceder a la aplicación; se le ofrece reenviar el correo de verificación. | Alta | ✅ |
| RF-AUTH-04 | El sistema permite iniciar sesión con correo y contraseña. | Alta | ✅ |
| RF-AUTH-05 | El sistema permite registrarse e iniciar sesión con Google. Un correo devuelto por Google como verificado se considera verificado en FINORA; el nombre se toma de Google y es editable. Al crear una cuenta con Google, la política de tratamiento de datos se acepta con un aviso junto al botón ("Al continuar con Google aceptas la Política de tratamiento de datos"); el consentimiento se registra con su versión. | Alta | ✅ |
| RF-AUTH-06 | El sistema permite cerrar la sesión actual. | Alta | ✅ |
| RF-AUTH-07 | La sesión expira tras 30 minutos sin actividad y, en cualquier caso, 7 días después de iniciada. Al expirar, el usuario debe iniciar sesión de nuevo. | Alta | ✅ |
| RF-AUTH-08 | El usuario puede solicitar un enlace de recuperación de contraseña; el enlace es de un solo uso y expira. | Alta | ✅ |
| RF-AUTH-09 | Al restablecer o cambiar la contraseña, se cierran todas las demás sesiones abiertas del usuario. | Alta | ✅ |
| RF-AUTH-10 | El usuario puede activar MFA con una aplicación de autenticación (TOTP), confirmándola con un código válido antes de quedar activa. | Media | ✅ |
| RF-AUTH-11 | Al activar MFA, el sistema entrega 10 códigos de recuperación de un solo uso, mostrados una sola vez. | Media | ✅ |
| RF-AUTH-12 | Con MFA activa, el inicio de sesión exige el código TOTP o un código de recuperación después de la contraseña. | Media | ✅ |
| RF-AUTH-13 | El usuario puede desactivar MFA tras una reautenticación reciente (RF-AUTH-21) y un código TOTP vigente. | Media | ✅ |
| RF-AUTH-14 | El usuario puede ver su perfil (nombre, correo, método de acceso, estado de MFA) y editar su nombre. | Media | ✅ |
| RF-AUTH-15 | El usuario con contraseña puede cambiarla indicando la actual. | Media | ✅ |
| RF-AUTH-16 | El usuario puede eliminar su cuenta. La eliminación es inmediata, definitiva y borra todos sus datos personales y financieros. Requiere confirmación explícita y una reautenticación reciente (RF-AUTH-21). | Alta | ✅ |
| RF-AUTH-17 | Si alguien entra con Google y ya existe una cuenta con contraseña y el mismo correo, el sistema pide la contraseña de FINORA (y el código MFA si está activa) antes de vincular Google a esa cuenta. Excepción: si esa cuenta nunca verificó su correo, Google demuestra que el correo es de quien entra; se descarta la contraseña no verificada, el correo queda verificado, se vincula Google y se inicia la sesión (protege contra cuentas registradas de antemano por un tercero). | Alta | ✅ |
| RF-AUTH-18 | Un usuario que solo entra con Google puede crear una contraseña de FINORA mediante un enlace enviado a su correo, desde "Olvidé mi contraseña" o desde su perfil. | Media | ✅ |
| RF-AUTH-19 | Si la MFA está activa, se exige el código también al entrar con Google. | Media | ✅ |
| RF-AUTH-20 | Al activar MFA, el sistema advierte que, si se pierden el dispositivo y los códigos de recuperación, no habrá forma de recuperar la cuenta, y pide confirmar que se guardaron los códigos. | Media | ✅ |
| RF-AUTH-21 | Reautenticación reciente para acciones sensibles (configurar o desactivar MFA, eliminar la cuenta): con contraseña si la cuenta la tiene, o volviendo a pasar por Google si solo usa Google; más el código MFA si está activa. Es válida 5 minutos en la sesión actual. Con Google y MFA activa, el código se pide al volver de Google, dentro de los 5 minutos siguientes. | Alta | ✅ |

## 7. Requisitos no funcionales

| ID | Requisito | Certeza |
|---|---|---|
| RNF-AUTH-01 | Contraseñas almacenadas solo como hash Argon2id; nunca en texto plano ni en registros. | ✅ (ADR-0001) |
| RNF-AUTH-02 | Sesión de servidor en cookie `HttpOnly`, `Secure`, `SameSite=Lax`; token anti-CSRF en peticiones que modifican datos; el identificador de sesión se regenera al iniciar sesión. | ✅ |
| RNF-AUTH-03 | Los mensajes de inicio de sesión, registro y recuperación no revelan si un correo está registrado (sin enumeración de cuentas). | ✅ |
| RNF-AUTH-04 | Limitación de intentos: tras 5 intentos fallidos de inicio de sesión o de código MFA para una misma cuenta en 15 minutos, se bloquean los intentos de esa cuenta durante 15 minutos. También hay límites por IP en registro, inicio de sesión y recuperación. | ✅ (DP-SEC-06) |
| RNF-AUTH-05 | Los tokens de verificación y de recuperación se generan con un generador criptográfico, se guardan solo como hash y son de un solo uso. Los secretos TOTP se cifran en reposo. | ✅ |
| RNF-AUTH-06 | Se registran eventos de seguridad (registro, inicio de sesión exitoso o fallido, cambio de contraseña, activación o desactivación de MFA, eliminación de cuenta) sin contraseñas, tokens ni códigos. | ✅ (DP-SEC-05) |
| RNF-AUTH-07 | Formularios y pantallas de autenticación cumplen WCAG 2.2 AA, incluido el criterio 3.3.8 (autenticación accesible: se permite pegar contraseñas y códigos y usar gestores de contraseñas). | ✅ |
| RNF-AUTH-08 | Textos de la interfaz y de los correos en español colombiano (`es-CO`); fechas en DD/MM/AAAA. | ✅ |
| RNF-AUTH-09 | La interfaz respeta la identidad visual aprobada en Claude Design (`docs/guides/claude-design.md`). | ✅ |

## 8. Reglas de negocio

| ID | Regla | Certeza |
|---|---|---|
| RN-AUTH-01 | El correo es único por cuenta y se compara sin distinguir mayúsculas y minúsculas y sin espacios al inicio o al final. | ✅ |
| RN-AUTH-02 | Contraseña: mínimo 12 caracteres y máximo 128; se rechazan contraseñas comunes o filtradas conocidas y la que coincida con el correo. No se exigen reglas de composición (mayúsculas, símbolos), según NIST SP 800-63B. | ✅ (DP-SEC-02) |
| RN-AUTH-03 | El nombre es obligatorio: entre 1 y 100 caracteres después de quitar espacios al inicio y al final, sin saltos de línea ni otros caracteres de control. | ✅ |
| RN-AUTH-04 | El enlace de verificación de correo expira a las 24 horas; reenviar uno invalida el anterior. | ✅ |
| RN-AUTH-05 | El enlace de recuperación de contraseña expira a la hora; solicitar uno nuevo invalida el anterior. | ✅ |
| RN-AUTH-06 | Cada código de recuperación de MFA solo sirve una vez. | ✅ |
| RN-AUTH-07 | La eliminación de la cuenta borra en cascada todos los datos del usuario en todas las features y cierra todas sus sesiones; no es reversible. | ✅ |
| RN-AUTH-08 | Ningún endpoint de perfil o cuenta acepta un identificador de usuario del cliente: el usuario se toma siempre de la sesión. | ✅ (aislamiento por usuario) |

## 9. Requisitos de datos

| Entidad / campo | Tipo lógico | Obligatorio | Notas |
|---|---|---|---|
| Usuario.id | Identificador | Sí | Generado por el backend |
| Usuario.correo | Texto (correo) | Sí | Único, normalizado (RN-AUTH-01) |
| Usuario.correo_verificado_en | Fecha y hora (UTC) | No | Vacío hasta verificar |
| Usuario.nombre | Texto | Sí | RN-AUTH-03 |
| Usuario.hash_contraseña | Texto | No | Vacío si la cuenta solo usa Google |
| Usuario.creado_en | Fecha y hora (UTC) | Sí | |
| IdentidadExterna (proveedor, sujeto) | Texto | No | Vinculación con Google (`sub` de OpenID Connect) |
| Sesión (id, usuario, creada_en, última_actividad, expira_en) | — | Sí | Para la expiración de RF-AUTH-07 y la revocación |
| MFA (secreto cifrado, activada_en, códigos de recuperación como hash) | — | No | RF-AUTH-10 y RF-AUTH-11 |
| Token (tipo, hash, usuario, expira_en, usado_en) | — | — | Verificación de correo y recuperación de contraseña |
| ConsentimientoDatos (versión de la política, aceptado_en) | — | Sí | ✅ Ley 1581; la versión permite pedir una nueva aceptación |

## 10. Validaciones y escenarios de error

| ID | Condición | Comportamiento esperado | Mensaje al usuario (es-CO) |
|---|---|---|---|
| VAL-AUTH-01 | Correo con formato inválido | `422`; no se crea la cuenta | "Ingresa un correo electrónico válido." |
| VAL-AUTH-02 | Contraseña que no cumple RN-AUTH-02 | `422` indicando la regla incumplida | "La contraseña debe tener al menos 12 caracteres." / "Esta contraseña es muy común. Elige otra." |
| VAL-AUTH-03 | Nombre vacío o demasiado largo | `422` | "Ingresa tu nombre." |
| VAL-AUTH-04 | Registro con un correo ya registrado | Misma respuesta que un registro exitoso (RNF-AUTH-03); al dueño del correo se le envía un aviso. Si el correo aún no está verificado, el nuevo registro reemplaza nombre y contraseña y se envía un enlace nuevo (los anteriores dejan de servir) | "Te enviamos un correo para confirmar tu cuenta." |
| VAL-AUTH-05 | Credenciales incorrectas o cuenta inexistente | `401` con mensaje genérico | "El correo o la contraseña no son correctos." |
| VAL-AUTH-06 | Inicio de sesión con correo sin verificar | `403`; se ofrece reenviar la verificación | "Confirma tu correo para continuar. ¿Te enviamos el enlace de nuevo?" |
| VAL-AUTH-07 | Cuenta bloqueada temporalmente (RNF-AUTH-04) | `429` | "Demasiados intentos. Intenta de nuevo en unos minutos." |
| VAL-AUTH-08 | Código MFA incorrecto | `401` | "El código no es correcto." |
| VAL-AUTH-09 | Enlace de verificación o recuperación vencido, usado o inválido | `400`; se ofrece pedir uno nuevo | "Este enlace ya no es válido. Solicita uno nuevo." |
| VAL-AUTH-10 | Sesión expirada o inexistente en una petición protegida | `401`; el frontend redirige al inicio de sesión | "Tu sesión terminó. Inicia sesión de nuevo." |
| VAL-AUTH-11 | Petición que modifica datos sin token anti-CSRF válido | `403` | "No pudimos completar la acción. Recarga la página e intenta de nuevo." |
| VAL-AUTH-12 | Google devuelve un correo no verificado | Se rechaza el acceso con Google | "Tu cuenta de Google no tiene el correo verificado." |
| VAL-AUTH-13 | Error o cancelación en el flujo de Google | Se vuelve al inicio de sesión sin crear la cuenta | "No pudimos iniciar sesión con Google. Intenta de nuevo." |

Todos los errores siguen el formato RFC 9457 ya definido en `finora-back`.

## 11. Casos borde

- Alguien registra el correo de otra persona sin verificarlo y después la dueña se registra: manda el último registro (nombre y contraseña), así que quien confirma el correo entra con la contraseña que acaba de elegir (VAL-AUTH-04).

- Mismo correo en mayúsculas o con espacios (`Ana@Correo.com ` frente a `ana@correo.com`): es la misma cuenta.
- Inicio con Google cuando ya existe una cuenta con contraseña y el mismo correo: se pide la contraseña antes de vincular (RF-AUTH-17).
- Alguien registró el correo con contraseña sin verificarlo y la dueña real entra con Google: Google se queda la cuenta y la contraseña no verificada se descarta (RF-AUTH-17).
- Usuario que entró solo con Google y pide "Olvidé mi contraseña": recibe un enlace para crear su primera contraseña (RF-AUTH-18).
- MFA activa en una cuenta que entra con Google: se pide el código igualmente (RF-AUTH-19).
- Usuario que pierde el teléfono con MFA: usa un código de recuperación.
- Usuario con MFA que no puede entrar con Google ni con contraseña y perdió sus códigos: la cuenta no se puede recuperar (RF-AUTH-20).
- Clic en un enlace de verificación con otra sesión abierta en el mismo navegador.
- Dos peticiones simultáneas con el mismo enlace de recuperación: solo una tiene éxito.
- Sesión que llega al límite de 7 días mientras el usuario está activo: se cierra igualmente.
- Varias pestañas abiertas: la actividad en cualquiera de ellas renueva la inactividad; el cierre de sesión aplica a todas.
- Eliminación de cuenta mientras hay sesiones abiertas en otros dispositivos: todas se invalidan.
- Registro de nuevo con un correo de una cuenta ya eliminada: se trata como un usuario nuevo, sin datos anteriores.
- Contraseña con caracteres Unicode o emojis: se acepta y se cuenta por caracteres, no por bytes.

## 12. Criterios de aceptación

**CA-AUTH-01**: Registro y verificación
- **Dado** un visitante con un correo no registrado
- **Cuando** se registra con nombre, correo y una contraseña válida, y abre el enlace recibido
- **Entonces** la cuenta queda verificada y puede iniciar sesión.

**CA-AUTH-02**: Acceso bloqueado sin verificación
- **Dado** una cuenta con correo sin verificar
- **Cuando** intenta iniciar sesión con credenciales correctas
- **Entonces** recibe `403`, no se crea la sesión y se le ofrece reenviar el enlace.

**CA-AUTH-03**: Sin enumeración de cuentas
- **Dado** un correo registrado y otro no registrado
- **Cuando** se usan en el registro, en el inicio de sesión con contraseña incorrecta y en la recuperación
- **Entonces** las respuestas para ambos correos son idénticas en código de estado y mensaje.

**CA-AUTH-04**: Inicio con Google
- **Dado** un visitante con una cuenta de Google de correo verificado
- **Cuando** inicia sesión con Google por primera vez
- **Entonces** se crea su cuenta ya verificada, con el nombre de Google, y queda con la sesión iniciada.

**CA-AUTH-05**: Expiración por inactividad
- **Dado** un usuario con sesión iniciada
- **Cuando** pasan 30 minutos sin peticiones y hace una nueva petición protegida
- **Entonces** recibe `401` y debe iniciar sesión de nuevo.

**CA-AUTH-06**: Expiración máxima
- **Dado** un usuario que usa la aplicación de forma continua
- **Cuando** se cumplen 7 días desde que inició la sesión
- **Entonces** la sesión expira aunque haya actividad reciente.

**CA-AUTH-07**: Recuperación de contraseña
- **Dado** un usuario con contraseña y sesiones abiertas en dos dispositivos
- **Cuando** pide recuperar la contraseña y la restablece con el enlace
- **Entonces** puede entrar con la nueva contraseña, la anterior deja de funcionar, el enlace no se puede reutilizar y las dos sesiones anteriores quedan cerradas.

**CA-AUTH-08**: Activación y uso de MFA
- **Dado** un usuario sin MFA
- **Cuando** la activa confirmando un código TOTP válido y luego inicia sesión
- **Entonces** se le piden la contraseña y un código TOTP, y recibe 10 códigos de recuperación de un solo uso.

**CA-AUTH-09**: Limitación de intentos
- **Dado** una cuenta existente
- **Cuando** se hacen 5 intentos fallidos de inicio de sesión en 15 minutos
- **Entonces** el sexto intento recibe `429`, incluso con la contraseña correcta.

**CA-AUTH-10**: Eliminación inmediata
- **Dado** un usuario con datos y sesiones abiertas en dos dispositivos
- **Cuando** confirma la eliminación de su cuenta y se reautentica
- **Entonces** sus datos dejan de existir en la base de datos, todas sus sesiones quedan inválidas y su correo puede registrarse como una cuenta nueva y vacía.

**CA-AUTH-11**: Aislamiento del perfil
- **Dado** dos usuarios A y B
- **Cuando** A consulta o edita el perfil
- **Entonces** solo ve y modifica sus propios datos; no existe forma de indicar el identificador de B.

**CA-AUTH-12**: Accesibilidad
- **Dado** las pantallas de registro, inicio de sesión, recuperación y MFA
- **Cuando** se analizan con axe-core y se recorren solo con teclado
- **Entonces** no hay violaciones de WCAG 2.2 AA y se puede pegar en los campos de contraseña y de código.

**CA-AUTH-13**: Vinculación segura con Google
- **Dado** una cuenta con contraseña y el correo `ana@correo.com`
- **Cuando** alguien entra con una cuenta de Google del mismo correo
- **Entonces** no se inicia sesión hasta que ingrese la contraseña de FINORA (y el código MFA si está activa); después, Google queda vinculado a esa cuenta y no se crea una cuenta duplicada.

**CA-AUTH-14**: MFA con Google
- **Dado** un usuario con MFA activa y Google vinculado
- **Cuando** entra con Google
- **Entonces** se le pide el código TOTP antes de crear la sesión.

## 13. Dependencias

- `project-setup` (completada): base de `finora-web` y `finora-back`, PostgreSQL, Alembic y errores RFC 9457.
- **Proveedor de envío de correos**: necesario para RF-AUTH-02 y RF-AUTH-08. Aún no está definido; ver PA-AUTH-10.
- **Credenciales de Google (OAuth 2.0 / OpenID Connect)**: un proyecto en Google Cloud con su cliente OAuth. Ver PA-AUTH-11.
- **Texto de la política de tratamiento de datos personales**: requiere asesoría legal (DP-SEC-04). Ver PA-AUTH-12.
- ADR-0001 solo contempla correo y contraseña. Agregar Google (OpenID Connect gestionado por `finora-back`) y un proveedor de correo amplía la arquitectura, así que debe registrarse en un nuevo ADR antes de implementar.
- Las features con datos financieros (`accounts`, `transactions`, etc.) deben cumplir RN-AUTH-07 (borrado en cascada).

## 14. Preguntas abiertas

> PA-AUTH-10 a 12 no bloquean la aprobación de la spec: no cambian el comportamiento descrito. Sí bloquean las tareas que dependen de ellas (envío real de correos, credenciales de Google, texto legal y salida a producción).

| ID | Pregunta | Responsable | Estado |
|---|---|---|---|
| PA-AUTH-01 | ¿Qué métodos de autenticación se ofrecerán? | Producto | Resuelta (29/10/2026): correo y contraseña, y Google. |
| PA-AUTH-02 | ¿Se requiere MFA? ¿Obligatoria u opcional? | Producto | Resuelta (29/10/2026): opcional, con TOTP. |
| PA-AUTH-03 | ¿Qué datos de perfil son obligatorios? | Producto | Resuelta (29/10/2026): solo el nombre (además del correo). |
| PA-AUTH-04 | ¿Se requiere verificación de correo o teléfono? | Producto | Resuelta (29/10/2026): verificación de correo obligatoria antes de usar la aplicación. |
| PA-AUTH-05 | ¿Cómo funciona la eliminación de cuenta y de sus datos? | Producto | Resuelta (29/10/2026): borrado inmediato y definitivo. |
| PA-AUTH-06 | Si alguien entra con Google y ya existe una cuenta con contraseña y el mismo correo, ¿se vinculan automáticamente (el correo de Google está verificado) o se pide la contraseña primero? | Producto | Resuelta (29/10/2026): pedir la contraseña de FINORA antes de vincular (RF-AUTH-17). |
| PA-AUTH-07 | Un usuario que solo entra con Google, ¿puede crear también una contraseña (desde "Olvidé mi contraseña" o desde el perfil)? | Producto | Resuelta (29/10/2026): sí, mediante enlace al correo (RF-AUTH-18). |
| PA-AUTH-08 | ¿La MFA de FINORA se pide también al entrar con Google, o se confía en la seguridad de la cuenta de Google? | Producto | Resuelta (29/10/2026): sí, la MFA se exige también con Google (RF-AUTH-19). |
| PA-AUTH-09 | Si el usuario pierde el dispositivo de MFA y los códigos de recuperación, ¿cómo recupera el acceso? (sin soporte definido, hoy perdería la cuenta) | Producto | Resuelta (29/10/2026): en el MVP la cuenta no se puede recuperar; se advierte al activar MFA (RF-AUTH-20). Se revisará cuando exista soporte. |
| PA-AUTH-10 | ¿Qué proveedor de envío de correos se usará? Para desarrollo local se puede usar un servidor de pruebas sin decidir el de producción. | Arquitectura | Abierta |
| PA-AUTH-11 | ¿Quién crea y administra el proyecto de Google Cloud y sus credenciales OAuth? | Desarrollador | Abierta |
| PA-AUTH-12 | ¿Existe ya el texto de la política de tratamiento de datos personales? ¿La eliminación inmediata es compatible con alguna obligación legal de conservar registros? | Legal | Abierta |

## 15. Supuestos

> Todos los supuestos de esta tabla fueron validados por producto el 29/10/2026 y ahora son requisitos confirmados.

| ID | Supuesto | Riesgo si es falso |
|---|---|---|
| SUP-AUTH-01 | La política de contraseñas sigue NIST SP 800-63B (RN-AUTH-02). | Ajustar validaciones y mensajes |
| SUP-AUTH-02 | Los plazos de los enlaces son 24 horas (verificación) y 1 hora (recuperación). | Cambio de configuración |
| SUP-AUTH-03 | El cambio de correo de la cuenta queda fuera de esta feature. | Agregar flujo con verificación del correo nuevo |
| SUP-AUTH-04 | Los registros de seguridad se conservan sin datos que identifiquen al usuario después de eliminar su cuenta. | Puede chocar con la normativa (PA-AUTH-12) |
| SUP-AUTH-05 | Las fechas y horas se guardan en UTC; la zona horaria de presentación sigue pendiente a nivel de proyecto. | Fechas mostradas con un día de diferencia |

## 16. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 29/10/2026 | Especificación completa a partir de las respuestas a PA-AUTH-01 a 05 (más recuperación de contraseña y duración de la sesión). Nuevas preguntas PA-AUTH-06 a 12. Pasa a `En revisión`. | Claude Code |
| 29/10/2026 | Resueltas PA-AUTH-06 a 09; nuevos RF-AUTH-17 a 20 y CA-AUTH-13 y 14. | Claude Code |
| 29/10/2026 | Validados todos los supuestos (🟡 → ✅). Aprobada; PA-AUTH-10 a 12 siguen abiertas y bloquean solo las tareas que dependen de ellas. | Claude Code |
| 29/10/2026 | Cambio aprobado (DIF-AUTH-01, detectado en el plan): RF-AUTH-13 y 16 pasan de "confirmar la contraseña" a una reautenticación reciente; nuevo RF-AUTH-21. Motivo: las cuentas solo con Google no tienen contraseña. | Claude Code |
| 30/10/2026 | Cambios aprobados al implementar T-AUTH-BE-10: consentimiento de datos con aviso junto al botón de Google (RF-AUTH-05); cuenta con el mismo correo sin verificar: Google se queda la cuenta (RF-AUTH-17); código MFA al volver de la reautenticación con Google (RF-AUTH-21). Motivo: casos no cubiertos detectados al implementar. | Claude Code |
| 30/10/2026 | Cambios aprobados en T-AUTH-QA-03 (revisión de seguridad): RN-AUTH-03 prohíbe saltos de línea y caracteres de control en el nombre; VAL-AUTH-04 y caso borde: registrarse con un correo aún sin verificar reemplaza nombre y contraseña (antes se conservaban). El correo de verificación ya no incluye el nombre. Motivo: precaptura de cuentas y uso del nombre para engañar desde correos de FINORA. | Claude Code |
