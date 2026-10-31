# Especificación: Centro de captura

| Campo | Valor |
|---|---|
| Identificador | `FEAT-CAP` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Punto único para registrar información financiera rápidamente (p. ej. fotos de recibos, entrada rápida o texto) y convertirla en movimientos o documentos.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-CAP-01 | ¿Qué fuentes de captura se soportan (foto, archivo, texto, voz, notificaciones bancarias, correo)? | Abierta |
| PA-CAP-02 | ¿Se usa OCR o IA para extraer datos? ¿Con qué proveedor? | Abierta |
| PA-CAP-03 | ¿El usuario confirma siempre antes de crear una transacción? | Abierta |
| PA-CAP-04 | ¿Cuál es la relación con el módulo `documents`? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
