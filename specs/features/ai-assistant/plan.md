# Plan técnico: Asistente financiero con IA

| Campo | Valor |
|---|---|
| Identificador | `FEAT-AI` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/plan-template.md`](../../templates/plan-template.md) |
| Especificación | [`spec.md`](spec.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

**No existen decisiones técnicas aprobadas para esta feature.** El plan se redactará con la estructura de la plantilla una vez que [`spec.md`](spec.md) esté en estado `Aprobado` y se hayan resuelto las decisiones de arquitectura pendientes ([`specs/architecture.md`](../../architecture.md)).

## Descripción

Asistente conversacional que responde preguntas sobre las finanzas del usuario y ofrece recomendaciones, respetando la privacidad y el aislamiento de datos.

## Dependencias pendientes

- Especificación aprobada de esta feature.
- Stack de `finora-web` y `finora-back` (DP-ARQ-01, DP-ARQ-02).
- Contratos API (ver [`specs/api-contracts.md`](../../api-contracts.md)).
