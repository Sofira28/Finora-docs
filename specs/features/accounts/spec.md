# Especificación: Cuentas y tarjetas

| Campo | Valor |
|---|---|
| Identificador | `FEAT-ACC` |
| Estado | `Aprobado` |
| Responsable | Responsable de producto (usuaria) |
| Última actualización | 30/10/2026 |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> Leyenda de certeza (aplicar a cada requisito o regla):
> - ✅ **Confirmado**: validado por el responsable de producto.
> - 🟡 **Supuesto**: hipótesis de trabajo; requiere validación.
> - ❓ **Pregunta abierta**: sin resolver; bloquea la aprobación si afecta el alcance.

---

## 1. Propósito

Que cada persona registre dónde tiene su dinero (cuentas bancarias, billeteras digitales, efectivo, ahorro e inversión) y lo que debe en sus tarjetas de crédito, y vea en un solo lugar dos cifras distintas: cuánto puede gastar ya (**saldo disponible**) y cuánto tiene en total descontando sus deudas de tarjeta (**patrimonio neto**).

Es la base de `transactions`: todo movimiento pertenece a una cuenta.

## 2. Alcance

- Crear, consultar, editar, archivar, reactivar y eliminar cuentas propias.
- Seis tipos de cuenta (RN-ACC-01) agrupados como en el diseño: "Cuentas y efectivo", "Ahorro e inversión" y "Tarjetas de crédito".
- Saldo inicial de cada cuenta y saldo actual calculado por el backend.
- Tarjetas de crédito: deuda, cupo total, cupo disponible, porcentaje de uso, día de corte y día límite de pago.
- Totales del usuario: saldo disponible, patrimonio neto y deuda total en tarjetas.
- Pantalla "Cuentas" según Claude Design, con alta de cuenta.

## 3. Fuera de alcance

- Registro de movimientos, transferencias y pagos de tarjeta: `transactions`. Hasta que exista, el saldo actual es igual al saldo inicial.
- Tasa de interés, intereses, cuotas y proyecciones de pago de tarjetas: `debts` y `predictions-alerts`.
- Préstamos y créditos distintos de tarjetas: `debts`.
- Relación de metas de ahorro con cuentas: `savings-goals`.
- Sincronización con bancos, importación de extractos (PA-PRJ-03) y otras monedas (solo COP).
- Cuentas compartidas entre usuarios.

## 4. Actores y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| Usuario autenticado | Sesión activa (`authentication`) | Solo sus propias cuentas: crear, ver, editar, archivar, reactivar y eliminar |
| Visitante sin sesión | — | Ninguno |

## 5. Historias de usuario

- **HU-ACC-01**: Como usuario, quiero registrar mis cuentas, billeteras, efectivo, ahorros, inversiones y tarjetas con su saldo actual, para empezar a usar FINORA con mi situación real.
- **HU-ACC-02**: Como usuario, quiero ver cuánto dinero puedo gastar ya, separado de mi patrimonio total, para no confundir mis ahorros o mi cupo de tarjeta con dinero disponible.
- **HU-ACC-03**: Como usuario, quiero ver la deuda, el cupo disponible y las fechas de corte y pago de cada tarjeta, para controlar su uso.
- **HU-ACC-04**: Como usuario, quiero archivar una cuenta que ya no uso sin perder su historial.
- **HU-ACC-05**: Como usuario, quiero corregir el nombre, la entidad o los datos de una cuenta cuando me equivoco.

## 6. Requisitos funcionales

| ID | Requisito | Prioridad | Certeza |
|---|---|---|---|
| RF-ACC-01 | El usuario crea una cuenta con nombre, tipo, entidad (opcional) y saldo inicial. Si es tarjeta de crédito, además cupo total, día de corte y día límite de pago; su saldo inicial se ingresa como **deuda actual**. | Alta | ✅ |
| RF-ACC-02 | El usuario ve sus cuentas activas agrupadas en "Cuentas y efectivo" (bancaria, billetera digital, efectivo), "Ahorro e inversión" (ahorro programado, inversión) y "Tarjetas de crédito", cada una con su saldo actual. | Alta | ✅ |
| RF-ACC-03 | Para cada tarjeta se muestran deuda, cupo total, cupo disponible (cupo − deuda), porcentaje de uso (deuda ÷ cupo), día de corte y día límite de pago. | Alta | ✅ |
| RF-ACC-04 | Se muestran el saldo disponible, el patrimonio neto y la deuda total en tarjetas, calculados por el backend según RN-ACC-03 a 05. | Alta | ✅ |
| RF-ACC-05 | El usuario edita nombre, entidad y, en tarjetas, cupo, día de corte y día de pago. El tipo no se puede cambiar (RN-ACC-08). | Alta | ✅ |
| RF-ACC-06 | El usuario edita el saldo inicial mientras la cuenta no tenga movimientos. Con movimientos, el saldo se corrige con un movimiento de ajuste (PA-ACC-05). | Media | ✅ |
| RF-ACC-07 | El usuario archiva una cuenta: deja de aparecer en la lista principal y en la selección de cuenta de nuevos movimientos, conserva su historial y deja de contar en los totales. Si su saldo no es cero, se le avisa antes de confirmar. | Alta | ✅ |
| RF-ACC-08 | El usuario ve sus cuentas archivadas en una sección aparte y puede reactivarlas. | Media | ✅ |
| RF-ACC-09 | El usuario elimina una cuenta solo si no tiene movimientos, tras confirmar. Con movimientos, se le ofrece archivarla. | Alta | ✅ |
| RF-ACC-10 | Sin cuentas, la pantalla muestra un estado vacío que invita a crear la primera. | Media | ✅ |

## 7. Requisitos no funcionales

| ID | Requisito | Certeza |
|---|---|---|
| RNF-ACC-01 | Aislamiento por usuario en todas las consultas (filtro por `user_id` + RLS, DP-DB-06). Una cuenta de otro usuario responde igual que una inexistente (`404`). | ✅ |
| RNF-ACC-02 | Montos con precisión decimal (`NUMERIC(19,4)` / `Decimal`, DP-DB-02); se transmiten como cadena decimal, nunca como número en coma flotante. Los totales los calcula solo el backend. | ✅ |
| RNF-ACC-03 | Interfaz es-CO: montos con formato colombiano (`$ 1.250.000`), negativos con signo menos. | ✅ |
| RNF-ACC-04 | WCAG 2.2 AA, responsive (escritorio, tablet, móvil) y fiel al diseño de Claude Design. El porcentaje de uso de una tarjeta se expone como `progressbar` accesible y el color nunca es la única señal. | ✅ |
| RNF-ACC-05 | Todas las escrituras exigen sesión activa y token anti-CSRF (como en `authentication`). | ✅ |

## 8. Reglas de negocio

| ID | Regla | Certeza |
|---|---|---|
| RN-ACC-01 | Tipos de cuenta: `bank` (cuenta bancaria de ahorros o corriente), `wallet` (billetera digital: Nequi, Daviplata…), `cash` (efectivo), `savings` (ahorro programado), `investment` (inversión: CDT, fondos…), `credit_card` (tarjeta de crédito). | ✅ |
| RN-ACC-02 | Saldo actual = saldo inicial + efecto de los movimientos de la cuenta (`transactions`). En una tarjeta, el saldo es la deuda: las compras la aumentan y los pagos la reducen. | ✅ (inicial) · ✅ (detalle de los movimientos, se define en `transactions`) |
| RN-ACC-03 | **Saldo disponible** = suma de los saldos de las cuentas activas `bank`, `wallet` y `cash`. No incluye ahorro, inversión ni el cupo libre de las tarjetas. | ✅ |
| RN-ACC-04 | **Patrimonio neto** = suma de los saldos de todas las cuentas activas que no son tarjeta − deuda total en tarjetas. Cuando exista `debts`, también restará esas deudas. | ✅ |
| RN-ACC-05 | **Deuda total en tarjetas** = suma de las deudas de las tarjetas activas. Una tarjeta con saldo a favor (deuda negativa) suma ese valor al patrimonio. | ✅ |
| RN-ACC-06 | Saldo disponible y patrimonio neto se muestran siempre por separado y con su nombre; nunca se intercambian ni se fusionan. | ✅ |
| RN-ACC-07 | Las transferencias entre cuentas propias (incluido el pago de una tarjeta desde otra cuenta) no son ingreso ni gasto: mueven saldo entre cuentas. Pueden cambiar el saldo disponible (p. ej. pasar plata a ahorro) pero no el patrimonio neto. | ✅ |
| RN-ACC-08 | El tipo de una cuenta no cambia después de crearla: cambiarlo alteraría el significado de su historial y de los totales. Para otro tipo, se crea otra cuenta. | ✅ |
| RN-ACC-09 | Una cuenta archivada no recibe movimientos nuevos y no cuenta en los totales; su historial se conserva. | ✅ |
| RN-ACC-10 | Solo se elimina una cuenta sin movimientos. La eliminación es física y definitiva. | ✅ |
| RN-ACC-11 | Al eliminar la cuenta de usuario (`authentication`, RN-AUTH-07) se borran todas sus cuentas. | ✅ |

## 9. Requisitos de datos

| Entidad / campo | Tipo lógico | Obligatorio | Notas |
|---|---|---|---|
| Cuenta.id | Identificador | Sí | No secuencial (UUID) |
| Cuenta.usuario | Referencia a usuario | Sí | Aislamiento (RNF-ACC-01) |
| Cuenta.nombre | Texto | Sí | 1 a 60 caracteres sin espacios en los extremos ni caracteres de control (✅ longitud); único entre las cuentas del usuario sin distinguir mayúsculas (✅) |
| Cuenta.tipo | Enumeración RN-ACC-01 | Sí | Inmutable (RN-ACC-08) |
| Cuenta.entidad | Texto | No | Banco o emisor, hasta 60 caracteres (✅) |
| Cuenta.saldo_inicial | Monto COP | Sí | En tarjetas, la deuda inicial. Hasta 2 decimales (✅) |
| Cuenta.cupo | Monto COP | Solo tarjetas | Mayor que cero |
| Cuenta.dia_corte | Entero 1–31 | Solo tarjetas | En meses más cortos, el último día del mes |
| Cuenta.dia_pago | Entero 1–31 | Solo tarjetas | Ídem |
| Cuenta.archivada_en | Fecha y hora | No | Vacío = activa |
| Cuenta.creada_en | Fecha y hora | Sí | UTC (DP-DB-07) |

## 10. Validaciones y escenarios de error

| ID | Condición | Comportamiento esperado | Mensaje al usuario (es-CO) |
|---|---|---|---|
| VAL-ACC-01 | Nombre vacío, de más de 60 caracteres o con caracteres de control | `422` en el campo | "Escribe un nombre de hasta 60 caracteres." |
| VAL-ACC-02 | Nombre repetido entre las cuentas del usuario (activas o archivadas) | `409` en el campo | "Ya tienes una cuenta con ese nombre." |
| VAL-ACC-03 | Monto con formato inválido, más de 2 decimales o fuera de rango | `422` en el campo | "Escribe un monto válido, p. ej. 1.250.000." |
| VAL-ACC-04 | Tarjeta sin cupo, con cupo ≤ 0 o sin días de corte y pago (1 a 31) | `422` en el campo | "Indica el cupo de la tarjeta." / "Elige un día entre 1 y 31." |
| VAL-ACC-05 | Campos de tarjeta en una cuenta que no es tarjeta | `422` | — (solo la API; el formulario no los muestra) |
| VAL-ACC-06 | Intento de cambiar el tipo | `422` | — (el formulario no lo permite) |
| VAL-ACC-07 | Editar el saldo inicial de una cuenta con movimientos | `409` | "Esta cuenta ya tiene movimientos. Para corregir el saldo, registra un ajuste." |
| VAL-ACC-08 | Eliminar una cuenta con movimientos | `409` | "Esta cuenta tiene movimientos. Puedes archivarla para ocultarla sin perder su historial." |
| VAL-ACC-09 | Cuenta inexistente o de otro usuario | `404` | "No encontramos esa cuenta." |
| VAL-ACC-10 | Deuda de tarjeta mayor que el cupo | Se permite; el cupo disponible se muestra en cero y el uso por encima del 100 % con aviso | "Superaste el cupo de esta tarjeta." |
| VAL-ACC-11 | Crear una cuenta con 100 cuentas ya registradas (SUP-ACC-06) | `409` | "Llegaste al máximo de 100 cuentas. Elimina las que ya no uses." |

## 11. Casos borde

- Saldo inicial cero: válido en cualquier tipo.
- Saldo negativo en una cuenta bancaria (sobregiro) o en efectivo por un error de registro: se permite y resta del saldo disponible.
- Tarjeta con deuda negativa (pagó de más): saldo a favor; uso 0 % y cupo disponible = cupo + saldo a favor.
- Día de corte o de pago 31 en febrero: se toma el último día del mes.
- Archivar la única cuenta: válido; la pantalla muestra el estado vacío con la sección de archivadas.
- Dos pestañas editan la misma cuenta: gana la última escritura.
- Sin ninguna cuenta: saldo disponible y patrimonio neto en `$ 0`.

## 12. Criterios de aceptación

**CA-ACC-01**: Crear cuentas y ver los totales
- **Dado** un usuario sin cuentas
- **Cuando** crea "Nómina" (`bank`, $ 2.800.000), "Efectivo" (`cash`, $ 90.000), "CDT" (`investment`, $ 12.000.000) y "Tarjeta Órbita" (`credit_card`, deuda $ 820.000, cupo $ 6.000.000)
- **Entonces** ve saldo disponible $ 2.890.000, patrimonio neto $ 14.070.000 y deuda en tarjetas $ 820.000, y cada cuenta en su grupo

**CA-ACC-02**: Datos de la tarjeta
- **Dado** la tarjeta del CA-ACC-01 con corte el día 28 y pago el día 18
- **Cuando** abre la pantalla de cuentas
- **Entonces** ve deuda $ 820.000, cupo disponible $ 5.180.000, uso 14 % y "Corte día 28 · pago día 18"

**CA-ACC-03**: El cupo no es saldo disponible
- **Dado** un usuario con solo una tarjeta de cupo $ 6.000.000 y deuda $ 0
- **Cuando** ve sus totales
- **Entonces** el saldo disponible es $ 0 y el patrimonio neto es $ 0

**CA-ACC-04**: Archivar
- **Dado** una cuenta "Billetera" con saldo $ 250.000
- **Cuando** la archiva y confirma el aviso de saldo distinto de cero
- **Entonces** desaparece de la lista principal, aparece en "Archivadas", los totales dejan de incluirla y puede reactivarla

**CA-ACC-05**: Eliminar
- **Dado** una cuenta sin movimientos
- **Cuando** la elimina y confirma
- **Entonces** desaparece por completo; y si la cuenta tuviera movimientos, recibe VAL-ACC-08 con la opción de archivar

**CA-ACC-06**: Aislamiento
- **Dado** dos usuarios A y B
- **Cuando** A consulta, edita, archiva o elimina una cuenta de B por su identificador
- **Entonces** recibe `404` y la cuenta de B no cambia; los listados y totales de A nunca incluyen cuentas de B

**CA-ACC-07**: Precisión
- **Dado** cuentas con saldos $ 0,10 y $ 0,20
- **Cuando** se calcula el saldo disponible
- **Entonces** es exactamente $ 0,30 (sin errores de coma flotante)

**CA-ACC-08**: Validaciones
- **Dado** el formulario de nueva cuenta
- **Cuando** envía un nombre repetido, una tarjeta sin cupo o un monto con 3 decimales
- **Entonces** recibe el mensaje de VAL-ACC-02, 04 o 03 en el campo correspondiente, y el backend rechaza lo mismo aunque se salte el formulario

## 13. Dependencias

- `authentication` (implementada): sesión, CSRF, aislamiento y borrado en cascada al eliminar el usuario.
- `transactions`: movimientos que cambian el saldo (RN-ACC-02), condición "tiene movimientos" (RF-ACC-06, RF-ACC-09) y ajuste de saldo (PA-ACC-05). Mientras no exista, ninguna cuenta tiene movimientos.
- `debts`: resuelve PA-DEBT-04 (las tarjetas viven en `accounts`; intereses y proyecciones en `debts`) y aportará otras deudas al patrimonio neto.
- `dashboard`: mostrará el saldo disponible y el patrimonio neto calculados aquí (PA-DASH-02).
- Claude Design: pantalla "Cuentas"; íconos según CD-08 (abierta).

## 14. Preguntas abiertas

| ID | Pregunta | Responsable | Estado |
|---|---|---|---|
| PA-ACC-01 | Tipos de cuenta iniciales | Producto | Resuelta (30/10/2026): los 6 del diseño (RN-ACC-01) |
| PA-ACC-02 | Saldo de una tarjeta y su efecto en el patrimonio | Producto | Resuelta (30/10/2026): el saldo es la deuda, que resta del patrimonio; se guardan cupo, día de corte y día de pago; la tasa queda para `debts` |
| PA-ACC-03 | Qué cuentas cuentan para el saldo disponible | Producto | Resuelta (30/10/2026): bancarias, billeteras y efectivo (RN-ACC-03) |
| PA-ACC-04 | Archivar cuentas con historial | Producto | Resuelta (30/10/2026): archivar y reactivar; eliminar solo sin movimientos |
| PA-ACC-05 | ¿Cómo se corrige el saldo de una cuenta con movimientos: un movimiento de "ajuste de saldo" que no sea ingreso ni gasto? | Producto | Resuelta (30/10/2026) en `transactions`: movimiento `adjustment` con la diferencia firmada (RF-TRX-03, RN-TRX-04) |

También resuelve PA-PRJ-05 de [`specs/project.md`](../../project.md) (definición de saldo disponible y patrimonio neto).

## 15. Supuestos

| ID | Supuesto | Riesgo si es falso |
|---|---|---|
| SUP-ACC-01 | Nombre de 1 a 60 caracteres, único por usuario sin distinguir mayúsculas; entidad opcional de hasta 60 | Ajustar validaciones (bajo) |
| SUP-ACC-02 | Los montos aceptan hasta 2 decimales, aunque en COP casi siempre son enteros | Cambiar la validación y el formato (bajo) |
| SUP-ACC-03 | Una cuenta archivada no cuenta en los totales ni recibe movimientos nuevos | Totales distintos a los esperados; cambio en RN-ACC-09 |
| SUP-ACC-04 | El tipo de cuenta no se puede cambiar | Habría que recalcular grupos y totales del historial |
| SUP-ACC-05 | Se permiten saldos negativos en cuentas que no son tarjeta y saldo a favor en tarjetas | Agregar validaciones |
| SUP-ACC-06 | Sin límite de cantidad de cuentas por usuario (más allá de un tope técnico de 100 para evitar abusos) | Bajo |
| SUP-ACC-07 | Ediciones concurrentes: gana la última escritura | Pérdida de un cambio en casos raros |

## 16. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 30/10/2026 | Especificación completa con las respuestas a PA-ACC-01 a 04 (recomendaciones aceptadas por la usuaria). Nuevos supuestos SUP-ACC-01 a 07 y PA-ACC-05. Pasa a `En revisión`. | Claude Code |
| 30/10/2026 | Aprobada por la usuaria, con los supuestos SUP-ACC-01 a 07 validados (🟡 → ✅). PA-ACC-05 sigue abierta y se resuelve en `transactions`. | Claude Code |
| 30/10/2026 | Al implementar T-ACC-BE-02: nuevo VAL-ACC-11 con el mensaje del tope de 100 cuentas (SUP-ACC-06 no lo definía). VAL-ACC-05 usa "Este dato solo aplica a tarjetas de crédito." en la API. Sin cambios de reglas. | Claude Code |
| 30/10/2026 | PA-ACC-05 resuelta por la spec aprobada de `transactions` (ajuste de saldo). Sin cambios de reglas de esta feature. | Claude Code |
