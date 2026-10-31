# Guías de trabajo

## Flujo SDD

1. **Especificar**: copiar [`specs/templates/spec-template.md`](../../specs/templates/spec-template.md) en `specs/features/<feature>/spec.md` (o completar el existente). Marcar cada requisito como ✅ Confirmado, 🟡 Supuesto o ❓ Pregunta abierta. Pasar a `Aprobado` solo tras revisión.
2. **Planificar**: completar `plan.md` con [`plan-template.md`](../../specs/templates/plan-template.md). Las decisiones arquitectónicas nuevas se registran como ADR en [`docs/decisions/`](../decisions/README.md).
3. **Tareas**: desglosar el plan aprobado en `tasks.md` con [`tasks-template.md`](../../specs/templates/tasks-template.md). Cada tarea tiene criterios de aceptación y un comando de verificación.
4. **Implementar**: solo el alcance aprobado, en la aplicación correspondiente (`finora-web` o `finora-back`).
5. **Verificar**: ejecutar las pruebas y marcar `[x]` solo las tareas verificadas.

## Convenciones de identificadores

| Prefijo | Uso |
|---|---|
| `FEAT-XXX` | Identificador de la feature |
| `RF-XXX-NN` / `RNF-XXX-NN` / `RN-XXX-NN` | Requisito funcional, no funcional y regla de negocio |
| `CA-XXX-NN` | Criterio de aceptación |
| `PA-XXX-NN` / `SUP-XXX-NN` | Pregunta abierta y supuesto |
| `T-XXX-{FE,BE,DB,INT,QA}-NN` | Tarea |
| `DP-*` | Decisión pendiente en specs compartidas |

## Otras guías

- [Integración con Claude Design](claude-design.md)
