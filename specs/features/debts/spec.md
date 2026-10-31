# Especificación: Deudas y préstamos

| Campo | Valor |
|---|---|
| Identificador | `FEAT-DEBT` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Registro y seguimiento de deudas y préstamos (adquiridos u otorgados), con cuotas, intereses y saldos pendientes que afectan el patrimonio neto.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-DEBT-01 | ¿Se incluyen préstamos que el usuario otorga a terceros? | Abierta |
| PA-DEBT-02 | ¿Se calculan intereses y tablas de amortización? ¿Con qué métodos? | Abierta |
| PA-DEBT-03 | ¿Cómo se relacionan los pagos de deuda con las transacciones? | Abierta |
| PA-DEBT-04 | ¿Las tarjetas de crédito se gestionan aquí o en `accounts`? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
