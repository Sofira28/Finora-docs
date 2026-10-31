# Especificación: Asistente financiero con IA

| Campo | Valor |
|---|---|
| Identificador | `FEAT-AI` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Asistente conversacional que responde preguntas sobre las finanzas del usuario y ofrece recomendaciones, respetando la privacidad y el aislamiento de datos.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-AI-01 | ¿Qué proveedor y modelo de IA se usará? | Abierta |
| PA-AI-02 | ¿Qué datos del usuario puede consultar el asistente y con qué consentimiento? | Abierta |
| PA-AI-03 | ¿El asistente solo informa o también puede ejecutar acciones (crear transacciones, presupuestos)? | Abierta |
| PA-AI-04 | ¿Se guarda el historial de conversaciones? | Abierta |
| PA-AI-05 | ¿Qué límites y advertencias se muestran (no es asesoría financiera profesional)? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
