# Especificación: Dashboard

| Campo | Valor |
|---|---|
| Identificador | `FEAT-DASH` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Vista general del estado financiero del usuario: saldos, resumen de ingresos y gastos, y accesos rápidos a los demás módulos, según el diseño aprobado en Claude Design.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-DASH-01 | ¿Qué indicadores (KPIs) muestra el dashboard y en qué período por defecto? | Abierta |
| PA-DASH-02 | ¿Se muestran por separado el saldo disponible y el patrimonio neto? | Abierta |
| PA-DASH-03 | ¿El dashboard es personalizable por el usuario? | Abierta |
| PA-DASH-04 | ¿Qué widgets del diseño de Claude Design forman parte del MVP? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
