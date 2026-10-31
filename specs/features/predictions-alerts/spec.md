# Especificación: Predicciones y alertas

| Campo | Valor |
|---|---|
| Identificador | `FEAT-PRED` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Proyecciones del flujo de caja y alertas sobre eventos relevantes (presupuestos excedidos, saldos bajos, vencimientos próximos, gastos inusuales).

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-PRED-01 | ¿Qué tipos de alertas existen y cuáles son configurables? | Abierta |
| PA-PRED-02 | ¿Por qué canales se envían (en la app, correo, push, WhatsApp)? | Abierta |
| PA-PRED-03 | ¿Qué horizonte y qué método tienen las predicciones? | Abierta |
| PA-PRED-04 | ¿Cómo se comunica la incertidumbre de una predicción al usuario? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
