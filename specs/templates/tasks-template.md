# Tareas: <Nombre de la feature>

| Campo | Valor |
|---|---|
| Identificador | `FEAT-XXX` |
| Estado | `Draft` \| `En revisión` \| `Aprobado` \| `En progreso` \| `Completado` |
| Especificación | [`spec.md`](spec.md) |
| Plan técnico | [`plan.md`](plan.md) |
| Última actualización | DD/MM/AAAA |

> Reglas:
> - Solo se listan tareas derivadas de un plan **aprobado**.
> - Una tarea se marca `[x]` **solo** después de ejecutar su verificación con resultado exitoso.
> - Aplicación: `finora-web` | `finora-back` | `shared` | `infraestructura`.
> - Prioridad: `Alta` | `Media` | `Baja`.

Leyenda de estado:

- [ ] Pendiente
- [x] Completada

---

## Frontend (`finora-web`)

- [ ] **T-XXX-FE-01**: <descripción>
  - **Aplicación:** `finora-web`
  - **Dependencias:** —
  - **Prioridad:** Media
  - **Criterios de aceptación:** <referencia a CA-XXX-NN o condición verificable>
  - **Verificación:** `<comando o prueba>`

## Backend (`finora-back`)

- [ ] **T-XXX-BE-01**: <descripción>
  - **Aplicación:** `finora-back`
  - **Dependencias:** —
  - **Prioridad:** Media
  - **Criterios de aceptación:**
  - **Verificación:** `<comando o prueba>`

## Base de datos

- [ ] **T-XXX-DB-01**: <descripción>
  - **Aplicación:** `finora-back`
  - **Dependencias:** —
  - **Prioridad:** Media
  - **Criterios de aceptación:**
  - **Verificación:** `<comando o prueba>`

## Integración

- [ ] **T-XXX-INT-01**: <descripción>
  - **Aplicación:** `shared`
  - **Dependencias:** T-XXX-FE-01, T-XXX-BE-01
  - **Prioridad:** Media
  - **Criterios de aceptación:**
  - **Verificación:** `<comando o prueba>`

## Pruebas

- [ ] **T-XXX-QA-01**: <descripción>
  - **Aplicación:** `finora-web` / `finora-back`
  - **Dependencias:** —
  - **Prioridad:** Media
  - **Criterios de aceptación:**
  - **Verificación:** `<comando o prueba>`

---

## Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| DD/MM/AAAA | Creación | |
