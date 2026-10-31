# Especificación: Transacciones

| Campo | Valor |
|---|---|
| Identificador | `FEAT-TRX` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Registro, edición, consulta y clasificación de movimientos (ingresos, gastos, ahorro y transferencias), garantizando que las transferencias entre cuentas propias no cuenten como ingreso ni gasto.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-TRX-01 | ¿Qué campos tiene una transacción (fecha, monto, cuenta, categoría, nota, adjuntos, etiquetas)? | Abierta |
| PA-TRX-02 | ¿Cómo se modelan las transferencias entre cuentas propias y hacia terceros? | Abierta |
| PA-TRX-03 | ¿Se permite dividir una transacción en varias categorías? | Abierta |
| PA-TRX-04 | ¿Se importarán movimientos desde extractos o bancos? ¿En qué formatos? | Abierta |
| PA-TRX-05 | ¿Qué filtros y búsquedas se requieren? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
