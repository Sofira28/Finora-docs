# Especificación: Pagos recurrentes

| Campo | Valor |
|---|---|
| Identificador | `FEAT-REC` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Gestión de pagos e ingresos recurrentes (suscripciones, servicios, arriendo, salario) con sus frecuencias y próximos vencimientos.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-REC-01 | ¿Qué frecuencias se soportan? | Abierta |
| PA-REC-02 | ¿Se generan transacciones automáticamente o solo recordatorios? | Abierta |
| PA-REC-03 | ¿Cómo se manejan los cambios de monto o las fechas que caen en festivos o fines de semana? | Abierta |
| PA-REC-04 | ¿Se detectan recurrencias automáticamente a partir del historial? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
