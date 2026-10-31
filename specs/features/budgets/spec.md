# Especificación: Presupuestos y categorías

| Campo | Valor |
|---|---|
| Identificador | `FEAT-BUD` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Definición de categorías de ingresos y gastos y de presupuestos por período, con seguimiento de lo ejecutado frente a lo planificado.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-BUD-01 | ¿Habrá categorías predeterminadas? ¿El usuario puede crear subcategorías? | Abierta |
| PA-BUD-02 | ¿Qué períodos de presupuesto se soportan (mensual, quincenal, personalizado)? | Abierta |
| PA-BUD-03 | ¿El saldo no usado se acumula al siguiente período? | Abierta |
| PA-BUD-04 | ¿Cuándo se notifica que un presupuesto está cerca de excederse? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
