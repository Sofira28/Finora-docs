# Especificación: Transacciones

| Campo | Valor |
|---|---|
| Identificador | `FEAT-TRX` |
| Estado | `Aprobado` |
| Responsable | Responsable de producto (usuaria) |
| Última actualización | 31/10/2026 |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> Leyenda de certeza (aplicar a cada requisito o regla):
> - ✅ **Confirmado**: validado por el responsable de producto.
> - 🟡 **Supuesto**: hipótesis de trabajo; requiere validación.
> - ❓ **Pregunta abierta**: sin resolver; bloquea la aprobación si afecta el alcance.

---

## 1. Propósito

Que cada persona registre lo que entra, sale y se mueve entre sus cuentas, y que FINORA lo clasifique sin confundir conceptos: un **gasto** o un **ingreso** cambia su patrimonio; una **transferencia** entre sus propias cuentas (incluido ahorrar o pagar la tarjeta) solo mueve dinero de un lugar a otro y **nunca** cuenta como ingreso ni gasto.

Los movimientos dan vida a `accounts`: el saldo actual de cada cuenta pasa a ser el saldo inicial más el efecto de sus movimientos (RN-ACC-02).

## 2. Alcance

- Registrar, consultar, editar y eliminar movimientos propios, solo de forma manual.
- Cuatro tipos: gasto, ingreso, transferencia entre cuentas propias y ajuste de saldo (RN-TRX-01).
- Categorías de un catálogo fijo inicial (RN-TRX-07).
- Efecto de cada movimiento en el saldo de sus cuentas, incluidas las tarjetas de crédito.
- Pantalla "Movimientos" según Claude Design: filtros, orden, resumen del resultado, paginación y estados de carga, error, vacío y sin resultados.
- Formulario de movimiento (modal del diseño, pestaña "Manual").
- Resolución de PA-ACC-05: corrección de saldo con un movimiento de ajuste.

## 3. Fuera de alcance

- Texto natural, voz, recibos con OCR, importación de CSV o extractos y sincronización bancaria: `capture-center`.
- Categorías propias y subcategorías: `budgets`.
- Hora, método de pago, etiquetas, adjuntos, estado "pendiente de revisión", recurrencia, cuotas y división de un movimiento en varias categorías.
- Movimientos con fecha futura y pagos programados: `recurring-payments`.
- Intereses y cuotas de tarjetas y créditos: `debts`.
- Exportar a CSV y aviso de posible duplicado (se evaluarán más adelante).
- Cálculo automático del 4×1000 (GMF).
- Otras monedas (solo COP).

## 4. Actores y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| Usuario autenticado | Sesión activa (`authentication`) | Solo sus propios movimientos y cuentas: crear, ver, editar y eliminar |
| Visitante sin sesión | — | Ninguno |

## 5. Historias de usuario

- **HU-TRX-01**: Como usuario, quiero registrar un gasto o un ingreso en una cuenta, para que su saldo refleje la realidad.
- **HU-TRX-02**: Como usuario, quiero registrar una transferencia entre mis cuentas (por ejemplo, retirar efectivo, ahorrar o pagar la tarjeta) sin que cuente como gasto.
- **HU-TRX-03**: Como usuario, quiero ver mis movimientos filtrados por periodo, tipo, categoría y cuenta, con el total de ingresos, gastos, ahorro y transferencias por separado.
- **HU-TRX-04**: Como usuario, quiero corregir o eliminar un movimiento mal registrado.
- **HU-TRX-05**: Como usuario, quiero ajustar el saldo de una cuenta cuando no coincide con el real, sin inventar un ingreso o un gasto.

## 6. Requisitos funcionales

| ID | Requisito | Prioridad | Certeza |
|---|---|---|---|
| RF-TRX-01 | El usuario registra un **gasto** o un **ingreso** con monto, fecha, cuenta, categoría, descripción y nota opcional. | Alta | ✅ |
| RF-TRX-02 | El usuario registra una **transferencia** con monto, fecha, cuenta de origen, cuenta de destino, descripción opcional y nota opcional. Sin categoría. | Alta | ✅ |
| RF-TRX-03 | El usuario registra un **ajuste de saldo** indicando el saldo real de una cuenta (en tarjetas, la deuda real). El sistema guarda la diferencia con el saldo actual como ajuste. Sin categoría. | Media | ✅ |
| RF-TRX-04 | El usuario ve sus movimientos de la más reciente a la más antigua, 50 por página, con fecha, descripción, tipo, categoría, cuenta (y destino en transferencias) y monto con su signo. | Alta | ✅ |
| RF-TRX-05 | El usuario filtra por texto (descripción, nota, categoría o cuenta), periodo (este mes, mes anterior, últimos 3 meses, todo el historial; meses calendario en hora de Colombia: "últimos 3 meses" es el mes actual y los dos anteriores, p. ej. del 01/08 al 31/10), tipo, categoría y cuenta, y ordena por fecha o por monto, ascendente o descendente. | Alta | ✅ |
| RF-TRX-06 | Junto al resultado se muestran, calculados por el backend sobre **todos** los movimientos que cumplen los filtros (no solo la página): cantidad, ingresos, gastos, ahorro neto y transferencias (RN-TRX-05). | Alta | ✅ |
| RF-TRX-07 | El usuario edita cualquier dato de un movimiento, incluido su tipo. Los saldos de las cuentas afectadas se recalculan. | Alta | ✅ |
| RF-TRX-08 | El usuario elimina un movimiento tras confirmar; la eliminación es definitiva y los saldos se recalculan. | Alta | ✅ |
| RF-TRX-09 | Sin movimientos, la pantalla invita a registrar el primero; con filtros sin resultados, muestra "Sin resultados" y permite limpiar los filtros. | Media | ✅ |
| RF-TRX-10 | "Movimientos" aparece en la navegación principal entre "Inicio" y "Cuentas". | Media | ✅ |

## 7. Requisitos no funcionales

| ID | Requisito | Certeza |
|---|---|---|
| RNF-TRX-01 | Aislamiento por usuario en todas las consultas (filtro por `user_id` + RLS, ADR-0004). Un movimiento o una cuenta de otro usuario se tratan igual que inexistentes (`404` o error de campo). | ✅ |
| RNF-TRX-02 | Montos con precisión decimal (`NUMERIC(19,4)` / `Decimal`); se transmiten como cadena decimal. Saldos y resúmenes los calcula solo el backend. | ✅ |
| RNF-TRX-03 | Interfaz es-CO: montos `$ 1.250.000`, fechas DD/MM/AAAA, día contable en hora de Colombia (`America/Bogota`). | ✅ |
| RNF-TRX-04 | WCAG 2.2 AA, responsive y fiel al diseño. El tipo de movimiento nunca se indica solo con color: lleva texto o signo. | ✅ |
| RNF-TRX-05 | Todas las escrituras exigen sesión activa y token anti-CSRF. | ✅ |
| RNF-TRX-06 | El listado responde en menos de 1 s con 10.000 movimientos por usuario (paginación e índices). | ✅ |

## 8. Reglas de negocio

| ID | Regla | Certeza |
|---|---|---|
| RN-TRX-01 | Tipos de movimiento: `expense` (gasto), `income` (ingreso), `transfer` (transferencia entre cuentas propias) y `adjustment` (ajuste de saldo). | ✅ |
| RN-TRX-02 | Efecto en el saldo (monto siempre positivo, salvo el ajuste). En cuentas que no son tarjeta, el saldo es dinero: un gasto lo resta y un ingreso lo suma. En tarjetas, el saldo es la deuda (RN-ACC-02): un gasto (compra) la aumenta y un ingreso (devolución) la reduce. | ✅ |
| RN-TRX-03 | Una transferencia saca el monto de la cuenta de origen y lo pone en la de destino con las mismas reglas: si el destino es una tarjeta, su deuda baja (**pago de tarjeta**); si el origen es una tarjeta, su deuda sube (avance). **No es ingreso ni gasto** y no cambia el patrimonio neto; puede cambiar el saldo disponible. | ✅ |
| RN-TRX-04 | Un ajuste guarda la diferencia firmada entre el saldo real indicado y el saldo actual (positiva o negativa) en una sola cuenta. No es ingreso ni gasto; sí cambia el saldo y el patrimonio. | ✅ |
| RN-TRX-05 | Resumen de un conjunto de movimientos: **ingresos** = suma de `income`; **gastos** = suma de `expense`; **ahorro neto** = transferencias hacia cuentas `savings` o `investment` desde cuentas de otro tipo, menos transferencias desde `savings` o `investment` hacia cuentas de otro tipo; **transferencias** = el resto de transferencias. Los ajustes no entran en ninguna de las cuatro cifras. | ✅ |
| RN-TRX-06 | Saldo actual de una cuenta = saldo inicial + efecto de todos sus movimientos (como origen o destino), en cualquier fecha. | ✅ |
| RN-TRX-07 | Categorías iniciales (catálogo fijo, CD-10). Gasto: vivienda, mercado, transporte, restaurantes, servicios, salud, entretenimiento, suscripciones, deudas. Ingreso: salario, freelance, otros ingresos. Ambos: sin categoría. Un gasto solo lleva categorías de gasto y un ingreso solo de ingreso. | ✅ |
| RN-TRX-08 | Una cuenta archivada no recibe movimientos nuevos (RN-ACC-09). Al editar un movimiento no se puede mover a una cuenta archivada; si ya estaba en una, puede editarse sin cambiar esa cuenta, y eliminarse. | ✅ |
| RN-TRX-09 | La fecha de un movimiento no puede ser posterior a hoy (hora de Colombia) ni anterior al 01/01/2000. | ✅ |
| RN-TRX-10 | Una cuenta con al menos un movimiento (como origen o destino) "tiene movimientos": su saldo inicial ya no se edita (RF-ACC-06) y no se puede eliminar, solo archivar (RN-ACC-10). | ✅ |
| RN-TRX-11 | Origen y destino de una transferencia son cuentas distintas del mismo usuario. | ✅ |

## 9. Requisitos de datos

| Entidad / campo | Tipo lógico | Obligatorio | Notas |
|---|---|---|---|
| Movimiento.usuario | Referencia a usuario | Sí | Aislamiento (RNF-TRX-01); borrado en cascada (RN-AUTH-07) |
| Movimiento.tipo | Enumeración | Sí | RN-TRX-01 |
| Movimiento.monto | Decimal | Sí | > 0 con hasta 2 decimales; en ajustes, diferencia firmada distinta de 0 |
| Movimiento.fecha | Fecha | Sí | Sin hora (RN-TRX-09) |
| Movimiento.cuenta | Referencia a cuenta | Sí | Origen en transferencias |
| Movimiento.cuenta de destino | Referencia a cuenta | Solo transferencias | RN-TRX-11 |
| Movimiento.categoría | Enumeración | Solo gasto e ingreso | RN-TRX-07 |
| Movimiento.descripción | Texto | Gasto e ingreso | 1 a 80 caracteres; opcional en transferencias y ajustes |
| Movimiento.nota | Texto | No | Hasta 500 caracteres |
| Movimiento.creado / actualizado | Fecha y hora | Sí | UTC |

## 10. Validaciones y escenarios de error

| ID | Condición | Comportamiento esperado | Mensaje al usuario (es-CO) |
|---|---|---|---|
| VAL-TRX-01 | Monto vacío, cero, negativo, con más de 2 decimales o fuera de rango | `422` en el campo | "Escribe un monto mayor que cero, p. ej. 1.250.000." |
| VAL-TRX-02 | Fecha vacía, inválida, futura o anterior a 2000 | `422` en el campo | "Elige una fecha entre el 01/01/2000 y hoy." |
| VAL-TRX-03 | Cuenta vacía, inexistente o de otro usuario | `422` en el campo | "Elige una de tus cuentas." |
| VAL-TRX-04 | Cuenta (o destino) archivada en un movimiento nuevo o al cambiarla | `422` en el campo | "Esta cuenta está archivada. Reactívala para registrar movimientos." |
| VAL-TRX-05 | Transferencia sin destino o con el mismo origen y destino | `422` en el campo | "Elige una cuenta de destino distinta a la de origen." |
| VAL-TRX-06 | Gasto o ingreso sin categoría o con una que no corresponde al tipo | `422` en el campo | "Elige una categoría." |
| VAL-TRX-07 | Descripción vacía en gasto o ingreso, de más de 80 caracteres o con caracteres de control | `422` en el campo | "Escribe una descripción de hasta 80 caracteres." |
| VAL-TRX-08 | Nota de más de 500 caracteres o con caracteres de control (salvo saltos de línea) | `422` en el campo | "La nota puede tener hasta 500 caracteres." |
| VAL-TRX-09 | Ajuste cuyo saldo real es igual al actual | `422` en el campo | "Ese ya es el saldo de la cuenta." |
| VAL-TRX-10 | Movimiento inexistente o de otro usuario | `404` | "No encontramos ese movimiento." |
| VAL-TRX-11 | Campos que no corresponden al tipo (p. ej. categoría en una transferencia) | `422` | — (solo la API; el formulario no los muestra) |

## 11. Casos borde

- Gasto mayor que el saldo: se permite; la cuenta queda en negativo (SUP-ACC-05).
- Pago de tarjeta mayor que la deuda: se permite; la tarjeta queda con saldo a favor.
- Transferencia entre dos tarjetas (compra de cartera): se permite; la deuda pasa de una a otra.
- Gasto en una cuenta de ahorro o inversión: se permite y cuenta como gasto.
- Ingreso directo en una cuenta de ahorro (p. ej. intereses): es ingreso, no ahorro.
- Editar un gasto para convertirlo en transferencia: pierde la categoría y la descripción pasa a ser opcional.
- Eliminar el último movimiento de una cuenta: vuelve a no tener movimientos y su saldo inicial vuelve a ser editable.
- Movimientos del 31 de diciembre a las 11:30 p. m. en Colombia: pertenecen a ese día y a ese mes (la fecha no tiene hora).
- Dos pestañas editan el mismo movimiento: gana la última escritura.

## 12. Criterios de aceptación

Contexto común: cuentas de CA-ACC-01 — "Nómina" (`bank`, $ 2.800.000), "Efectivo" (`cash`, $ 90.000), "CDT" (`investment`, $ 12.000.000) y "Tarjeta Órbita" (`credit_card`, deuda $ 820.000, cupo $ 6.000.000). Saldo disponible $ 2.890.000 y patrimonio neto $ 14.070.000.

**CA-TRX-01**: Gasto
- **Dado** el contexto común
- **Cuando** registra un gasto de $ 120.000 en "Nómina", categoría mercado
- **Entonces** "Nómina" queda en $ 2.680.000, el saldo disponible en $ 2.770.000, el patrimonio en $ 13.950.000 y el resumen del mes muestra gastos $ 120.000

**CA-TRX-02**: Transferencia entre cuentas líquidas
- **Dado** el contexto común
- **Cuando** transfiere $ 200.000 de "Nómina" a "Efectivo"
- **Entonces** "Nómina" queda en $ 2.600.000 y "Efectivo" en $ 290.000; saldo disponible y patrimonio no cambian; ingresos y gastos siguen en $ 0 y transferencias muestra $ 200.000

**CA-TRX-03**: Ahorro
- **Dado** el contexto común
- **Cuando** transfiere $ 500.000 de "Nómina" a "CDT"
- **Entonces** el saldo disponible baja a $ 2.390.000, el patrimonio sigue en $ 14.070.000, el resumen muestra ahorro neto $ 500.000 y gastos $ 0

**CA-TRX-04**: Compra con tarjeta
- **Dado** el contexto común
- **Cuando** registra un gasto de $ 300.000 con "Tarjeta Órbita", categoría restaurantes
- **Entonces** la deuda queda en $ 1.120.000, el cupo disponible en $ 4.880.000, el saldo disponible no cambia, el patrimonio baja a $ 13.770.000 y gastos muestra $ 300.000

**CA-TRX-05**: Pago de tarjeta
- **Dado** el contexto común
- **Cuando** transfiere $ 820.000 de "Nómina" a "Tarjeta Órbita"
- **Entonces** la deuda queda en $ 0, el saldo disponible en $ 2.070.000, el patrimonio sigue en $ 14.070.000 y gastos sigue en $ 0

**CA-TRX-06**: Ajuste de saldo
- **Dado** el contexto común
- **Cuando** ajusta "Efectivo" a un saldo real de $ 75.000
- **Entonces** se guarda un ajuste de −$ 15.000, "Efectivo" queda en $ 75.000, el patrimonio baja a $ 14.055.000 e ingresos y gastos no cambian

**CA-TRX-07**: Editar y eliminar
- **Dado** el gasto de CA-TRX-01
- **Cuando** cambia el monto a $ 100.000 y luego lo elimina
- **Entonces** "Nómina" pasa a $ 2.700.000 y después vuelve a $ 2.800.000; sin movimientos, su saldo inicial vuelve a ser editable

**CA-TRX-08**: Cuentas con movimientos
- **Dado** una cuenta con un movimiento
- **Cuando** intenta eliminarla o editar su saldo inicial
- **Entonces** recibe VAL-ACC-08 o VAL-ACC-07, y puede archivarla

**CA-TRX-09**: Validaciones
- **Dado** el formulario de movimiento
- **Cuando** envía una fecha futura, una transferencia de "Nómina" a "Nómina", un gasto sin categoría o un movimiento en una cuenta archivada
- **Entonces** recibe VAL-TRX-02, 05, 06 o 04 en el campo correspondiente y no se guarda nada

**CA-TRX-10**: Filtros y resumen
- **Dado** 60 movimientos de "Nómina" en el mes (gastos, ingresos, ahorro y transferencias) y otros en "Efectivo"
- **Cuando** filtra por "Este mes" y por la cuenta "Nómina"
- **Entonces** ve los primeros 50 y puede cargar los otros 10; el resumen suma los 60 de "Nómina", con ingresos, gastos, ahorro neto y transferencias por separado, y no incluye los de "Efectivo"

**CA-TRX-11**: Aislamiento
- **Dado** dos usuarios A y B
- **Cuando** A consulta, edita o elimina un movimiento de B, o registra uno en una cuenta de B
- **Entonces** recibe `404` o VAL-TRX-03, y los movimientos, saldos y resúmenes de A nunca incluyen datos de B

**CA-TRX-12**: Precisión
- **Dado** dos ingresos de $ 0,10 y $ 0,20 en "Nómina"
- **Cuando** consulta el resumen
- **Entonces** ingresos es exactamente $ 0,30 y el saldo de "Nómina" sube exactamente $ 0,30

## 13. Dependencias

- `authentication` (implementada): sesión, CSRF, aislamiento y borrado en cascada.
- `accounts` (implementada, QA pendiente): cuentas, saldos, totales y la función `movement_count`, que pasa a contar movimientos reales. Resuelve PA-ACC-05.
- `budgets`: categorías propias y subcategorías; reemplazará o extenderá el catálogo de RN-TRX-07 (PA-BUD-01).
- `capture-center`: otras formas de registro que crearán movimientos con estas mismas reglas.
- `dashboard` y `analytics`: usarán el resumen de RN-TRX-05.
- Claude Design: pantalla "Movimientos" y modal de movimiento; íconos según CD-08 (abierta); conforme con CD-06.

## 14. Preguntas abiertas

| ID | Pregunta | Responsable | Estado |
|---|---|---|---|
| PA-TRX-01 | Campos de una transacción | Producto | Resuelta (30/10/2026): §9; hora, método, etiquetas y adjuntos fuera de alcance |
| PA-TRX-02 | Modelo de transferencias entre cuentas propias y hacia terceros | Producto | Resuelta (30/10/2026): un movimiento `transfer` con origen y destino entre cuentas propias; un pago a un tercero es un gasto (RN-TRX-03). El modelo de tablas lo define el plan (DP-DB-03) |
| PA-TRX-03 | Dividir una transacción en varias categorías | Producto | Resuelta (30/10/2026): no en esta versión |
| PA-TRX-04 | Importación de extractos o bancos | Producto | Resuelta (30/10/2026): fuera de alcance; `capture-center` |
| PA-TRX-05 | Filtros y búsquedas | Producto | Resuelta (30/10/2026): RF-TRX-05 |

## 15. Supuestos

| ID | Supuesto | Riesgo si es falso |
|---|---|---|
| SUP-TRX-01 | Montos con hasta 2 decimales y 15 dígitos enteros, como en `accounts` | Ajustar validaciones (bajo) |
| SUP-TRX-02 | Fechas desde el 01/01/2000 | Ajustar la validación (bajo) |
| SUP-TRX-03 | El ahorro del resumen es neto: lo que entra a ahorro o inversión menos lo que sale de ahí hacia otras cuentas (puede ser negativo) | Cambiar la fórmula de RN-TRX-05 (medio) |
| SUP-TRX-04 | La búsqueda no distingue mayúsculas ni tildes ("cafe" encuentra "Café") | Resultados inesperados (bajo) |
| SUP-TRX-05 | En transferencias sin descripción, la lista muestra "Transferencia a <destino>"; en ajustes, "Ajuste de saldo" | Cambio de textos (bajo) |
| SUP-TRX-06 | Sin tope de cantidad de movimientos por usuario; RNF-TRX-06 dimensiona 10.000 | Revisar rendimiento (medio) |
| SUP-TRX-07 | Ediciones concurrentes: gana la última escritura | Pérdida de un cambio en casos raros |

## 16. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| 29/10/2026 | Creación (inicialización de la feature) | Claude Code |
| 30/10/2026 | Especificación completa con las recomendaciones aceptadas por la usuaria: cuatro tipos (con ajuste de saldo y ahorro como transferencia hacia ahorro o inversión), catálogo fijo de categorías, tarjetas, campos, fechas no futuras, pantalla y solo registro manual. Resuelve PA-TRX-01 a 05. Nuevos supuestos SUP-TRX-01 a 07. Pasa a `En revisión`. | Claude Code |
| 30/10/2026 | Aprobada por la usuaria, con los supuestos SUP-TRX-01 a 07 validados (🟡 → ✅ en RN-TRX-05, RN-TRX-09 y RNF-TRX-06). | Claude Code |
| 31/10/2026 | Aclaración en RF-TRX-05 confirmada por la usuaria (T-TRX-FE-01): los periodos son meses calendario en hora de Colombia y "últimos 3 meses" abarca el mes actual y los dos anteriores. Sin cambio de alcance. | Claude Code |
