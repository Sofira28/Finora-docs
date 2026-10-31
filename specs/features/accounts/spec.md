# Especificación: Cuentas y tarjetas

| Campo | Valor |
|---|---|
| Identificador | `FEAT-ACC` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Gestión de las cuentas del usuario (bancarias, efectivo, billeteras digitales) y tarjetas de crédito, con sus saldos y su aporte al saldo disponible y al patrimonio neto.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-ACC-01 | ¿Qué tipos de cuenta se soportan inicialmente (ahorros, corriente, efectivo, Nequi/Daviplata, tarjeta de crédito, inversión)? | Abierta |
| PA-ACC-02 | ¿Cómo se calcula el saldo de una tarjeta de crédito y su efecto en el patrimonio neto? | Abierta |
| PA-ACC-03 | ¿Qué cuentas cuentan para el saldo disponible? | Abierta |
| PA-ACC-04 | ¿Se permite archivar cuentas con historial? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
