# Especificación: Documentos

| Campo | Valor |
|---|---|
| Identificador | `FEAT-DOC` |
| Estado | `Draft` |
| Última actualización | 29/10/2026 |
| Plantilla | [`specs/templates/spec-template.md`](../../templates/spec-template.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Tareas | [`tasks.md`](tasks.md) |

> ⚠️ **Los requisitos detallados y las decisiones técnicas están pendientes.** Este documento solo inicializa la feature y no contiene requisitos aprobados.

## Descripción

Almacenamiento y organización de documentos financieros (recibos, facturas, extractos, contratos) asociados a movimientos, cuentas o deudas.

## Reglas transversales aplicables

Esta feature debe respetar las reglas financieras y de seguridad definidas en [`specs/project.md`](../../project.md) y [`specs/security.md`](../../security.md).

## Preguntas abiertas

| ID | Pregunta | Estado |
|---|---|---|
| PA-DOC-01 | ¿Qué tipos y tamaños de archivo se permiten? | Abierta |
| PA-DOC-02 | ¿Dónde se almacenan los archivos y cómo se cifran? | Abierta |
| PA-DOC-03 | ¿Se asocian documentos a transacciones, cuentas o deudas? | Abierta |
| PA-DOC-04 | ¿Cuál es la política de retención y eliminación? | Abierta |

## Próximo paso

Completar la especificación con la estructura de la plantilla (alcance, historias de usuario, requisitos RF/RNF/RN, criterios de aceptación Dado/Cuando/Entonces) y someterla a aprobación.
