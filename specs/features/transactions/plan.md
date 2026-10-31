# Plan técnico: Transacciones

| Campo | Valor |
|---|---|
| Identificador | `FEAT-TRX` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/plan-template.md`](../../templates/plan-template.md) |
| Especificación | [`spec.md`](spec.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

**No existen decisiones técnicas aprobadas para esta feature.** El plan se redactará con la estructura de la plantilla una vez que [`spec.md`](spec.md) esté en estado `Aprobado` y se hayan resuelto las decisiones de arquitectura pendientes ([`specs/architecture.md`](../../architecture.md)).

## Descripción

Registro, edición, consulta y clasificación de movimientos (ingresos, gastos, ahorro y transferencias), garantizando que las transferencias entre cuentas propias no cuenten como ingreso ni gasto.

## Dependencias pendientes

- Especificación aprobada de esta feature.
- Stack de `finora-web` y `finora-back` (DP-ARQ-01, DP-ARQ-02).
- Contratos API (ver [`specs/api-contracts.md`](../../api-contracts.md)).
