# Plan técnico: <Nombre de la feature>

| Campo | Valor |
|---|---|
| Identificador | `FEAT-XXX` |
| Estado | `Draft` \| `En revisión` \| `Aprobado` \| `Obsoleto` |
| Especificación | [`spec.md`](spec.md) (versión / fecha de referencia) |
| Tareas | [`tasks.md`](tasks.md) |
| Última actualización | DD/MM/AAAA |

> ⚠️ No asumir frameworks, bases de datos ni infraestructura: confirmar contra el estado real del repositorio y contra `specs/architecture.md` y los ADRs en `docs/decisions/`.
> Este plan solo puede aprobarse cuando la especificación referenciada esté en estado `Aprobado`.

---

## 1. Contexto de la arquitectura actual

<Estado real de `finora-web` y `finora-back` relevante para esta feature.>

## 2. Solución propuesta

<Resumen del enfoque y su justificación frente a los requisitos (RF/RNF/RN).>

## 3. Cambios en el frontend (`finora-web`)

- Páginas / rutas:
- Componentes reutilizables vs. específicos de página:
- Estado y manejo de datos:
- Referencia en Claude Design (pantalla / componente):

## 4. Cambios en el backend (`finora-back`)

- Módulos / servicios:
- Lógica de negocio y cálculos:

## 5. Cambios en la base de datos

- Entidades / tablas / migraciones:
- Índices y restricciones:

## 6. Endpoints y contratos API

| Método | Ruta | Descripción | Request | Response | Errores |
|---|---|---|---|---|---|
| | | | | | |

> Todo endpoint nuevo o modificado debe registrarse en `specs/api-contracts.md`.

## 7. Flujo de datos

<Descripción o diagrama del recorrido de los datos entre UI, API y persistencia.>

## 8. Consideraciones de seguridad

- Autenticación / autorización:
- Aislamiento de datos por usuario:
- Datos sensibles:

## 9. Estrategia de validación

- Frontend (ayuda al usuario):
- Backend (obligatoria):

## 10. Manejo de errores

<Códigos, mensajes en es-CO, reintentos, estados vacíos y de error en la UI.>

## 11. Estrategia de pruebas

| Nivel | Alcance | Herramienta | Criterios de aceptación cubiertos |
|---|---|---|---|
| Unitarias | | | |
| Integración | | | |
| E2E | | | |
| Accesibilidad | | | |

## 12. Dependencias y riesgos

| Riesgo / dependencia | Impacto | Mitigación |
|---|---|---|
| | | |

## 13. Secuencia de implementación

1. 

## 14. Decisiones arquitectónicas

| Decisión | Alternativas consideradas | ADR | Estado |
|---|---|---|---|
| | | `docs/decisions/ADR-XXXX-...md` | Propuesta |

## 15. Historial de cambios

| Fecha | Cambio | Autor |
|---|---|---|
| DD/MM/AAAA | Creación | |
