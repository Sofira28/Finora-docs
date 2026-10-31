# FINORA — Instrucciones permanentes del proyecto

> **FINORA** — *"Tu dinero, más claro."*
> Plataforma premium de gestión de finanzas personales, enfocada inicialmente en usuarios de Colombia.

Este archivo contiene las reglas permanentes para trabajar en el repositorio. Los detalles funcionales y técnicos viven en `specs/` y **no deben duplicarse aquí**.

---

## 1. Propósito y alcance

FINORA ayuda a las personas a entender y controlar su dinero: cuentas, transacciones, presupuestos, metas de ahorro, deudas, pagos recurrentes, análisis, alertas y un asistente financiero con IA.

- Locale inicial: `es-CO` · Moneda: COP · Fecha: DD/MM/AAAA · Formato numérico colombiano.
- Visión, público objetivo y módulos: [`specs/project.md`](specs/project.md).

## 2. Estructura del repositorio

```text
FINORA/                    ← repositorio Finora-docs
├── CLAUDE.md              ← este archivo
├── specs/                 ← especificaciones compartidas (fuente de verdad)
│   ├── project.md, architecture.md, database.md, api-contracts.md, security.md
│   ├── templates/         ← plantillas spec / plan / tasks
│   └── features/<feature>/{spec.md, plan.md, tasks.md}
├── docs/
│   ├── decisions/         ← ADRs (decisiones de arquitectura)
│   └── guides/            ← guías de trabajo
├── finora-web/            ← aplicación frontend (repositorio Finora-web)
└── finora-back/           ← aplicación backend (repositorio Finora-back)
```

- `finora-web` y `finora-back` son **aplicaciones independientes**: dependencias, configuración y pruebas separadas.
- Tres repositorios ([ADR-0003](docs/decisions/ADR-0003-repositorios.md)): `Sofira28/Finora-docs` (raíz: `CLAUDE.md`, `specs/`, `docs/`; ignora las carpetas de las aplicaciones), `Sofira28/Finora-web` y `Sofira28/Finora-back`. Rama base `main`; cada feature en `feat/<feature>`.
- No crear más repositorios ni ejecutar `git init` sin solicitud expresa.
- Claude Code no hace `commit` ni `push`: entrega los comandos y la usuaria los ejecuta. Los mensajes de commit no mencionan a Claude.

## 3. Responsabilidades

| Aplicación | Responsabilidad |
|---|---|
| `finora-web` | Interfaz de usuario, experiencia visual, accesibilidad, validación de formularios **solo como ayuda al usuario**, consumo de la API según `specs/api-contracts.md`. |
| `finora-back` | Lógica de negocio, cálculos financieros, persistencia, autenticación/autorización, **validación obligatoria** de toda entrada, aislamiento de datos por usuario. |

Stack aprobado en [`docs/decisions/ADR-0001-stack-tecnologico.md`](docs/decisions/ADR-0001-stack-tecnologico.md): `finora-web` en TypeScript + React + Vite; `finora-back` en Python + FastAPI + PostgreSQL. Hosting, proveedor de IA y almacenamiento de archivos siguen pendientes (ver `specs/architecture.md`); no asumirlos.

## 4. Flujo SDD (Spec-Driven Development)

Todo desarrollo sigue este ciclo, sin saltar etapas:

1. **Especificar** (`spec.md`): requisitos, reglas de negocio, casos borde y criterios de aceptación.
2. **Planificar** (`plan.md`): cómo se implementará técnicamente.
3. **Tareas** (`tasks.md`): trabajo dividido en tareas concretas y verificables.
4. **Implementar**: solo el alcance aprobado.
5. **Verificar**: probar la implementación contra la especificación.

Cada feature tiene su carpeta en `specs/features/<feature>/` con `spec.md`, `plan.md` y `tasks.md`, creados a partir de `specs/templates/`.

### Reglas operativas (obligatorias)

Para cada feature o cambio significativo:

1. **Leer** el `spec.md` relevante (y las specs compartidas que apliquen) **antes de modificar código**.
2. Identificar requisitos faltantes.
3. Preguntar cuando sea necesario; no inventar requisitos.
4. Actualizar la especificación u obtener su aprobación.
5. Crear o actualizar `plan.md`.
6. Crear o actualizar `tasks.md`.
7. Obtener aprobación antes de decisiones arquitectónicas o de negocio significativas.
8. Implementar solo el alcance aprobado.
9. Ejecutar las pruebas relevantes.
10. Marcar tareas como completadas **solo después de verificarlas**.
11. Si la implementación exige un cambio, actualizar la spec y el plan, explicar la diferencia y proponer la actualización.

## 5. Aprobación de requisitos

- Estados de un documento: `Draft` → `En revisión` → `Aprobado` → (`Obsoleto`).
- Solo se implementa lo que está en estado **Aprobado**.
- Se distinguen siempre: **requisitos confirmados**, **supuestos** y **preguntas abiertas**.
- Nunca cambiar en silencio reglas de negocio ni contratos API.

## 6. Contratos API

- `specs/api-contracts.md` es la fuente de verdad de la comunicación entre `finora-web` y `finora-back`.
- **No inventar endpoints.** Cada endpoint se define en el plan de su feature y se registra en el contrato antes de implementarse.
- Cualquier cambio de contrato es explícito, versionado y aprobado; frontend y backend se actualizan de forma coordinada.

## 7. Pruebas y verificación

- Cada tarea en `tasks.md` declara un comando o prueba de verificación.
- Los criterios de aceptación (Dado/Cuando/Entonces) deben poder trazarse a pruebas.
- Los cálculos financieros requieren pruebas unitarias con casos borde (redondeo, transferencias, saldos negativos).
- No declarar una feature como implementada si solo está documentada.

## 8. Seguridad

Detalles en [`specs/security.md`](specs/security.md). Mínimos no negociables:

- Validación obligatoria en el backend; el frontend nunca es fuente de confianza.
- Los datos financieros están aislados por usuario en todas las consultas.
- Sin secretos en el código ni en el repositorio.
- Principio de mínimo privilegio y protección de datos personales (normativa colombiana aplicable — pendiente de precisar).

## 9. Reglas financieras de negocio

- Las **transferencias entre cuentas propias** del usuario **no cuentan** como ingreso ni gasto.
- **Saldo disponible** y **patrimonio neto** son conceptos distintos y nunca se intercambian.
- Ingresos, gastos, ahorro y transferencias se clasifican correctamente y por separado.
- Los cálculos monetarios usan **precisión segura** (nunca aritmética de punto flotante para dinero).
- Los datos financieros se aíslan por usuario.

## 10. Identidad visual

- Diseño fintech premium y elegante.
- Fondo azul marino oscuro "cósmico", con acentos violeta y lavanda.
- Tipografía **Inter**.
- Animaciones y sombras sutiles; visualización de datos financieros clara.
- Responsive: escritorio, tablet y móvil.
- Accesibilidad objetivo: **WCAG 2.2 AA**.

No rediseñar ni reemplazar la identidad visual aprobada sin autorización.

## 11. Claude Design: referencia visual

- Fuente visual de verdad: proyecto de Claude Design `5d994040-ff6b-472b-9ca0-cbce5b162fcc`, archivos `FINORA.dc.html` y `support.js`.
- **Estado: importado (29/10/2026).** Informe, tokens de diseño y conflictos abiertos (CD-01 a CD-10) en [`docs/guides/claude-design.md`](docs/guides/claude-design.md).
- El diseño se **adapta** a la arquitectura de `finora-web`; no se copia el prototipo tal cual.
- Se preservan layout, tipografía, espaciado, colores, interacciones y comportamiento responsive.
- Si el diseño entra en conflicto con estas pautas, se reporta y se pide aclaración; no se corrige en silencio.

## 12. Cambios de requisitos

Cuando un requisito cambie:

1. Actualizar primero el `spec.md` afectado (y las specs compartidas si aplica).
2. Registrar el motivo del cambio en el historial del documento.
3. Revisar y actualizar `plan.md` y `tasks.md`; reabrir las tareas afectadas.
4. Si afecta arquitectura, registrar un ADR en `docs/decisions/`.
5. Obtener aprobación antes de implementar.

## 13. Idioma

- Toda la documentación (specs, planes, tareas, ADRs, guías, este archivo) se redacta en **español**.
- Los reportes y explicaciones de Claude Code se redactan en español.
- Los nombres de tecnologías, funciones, variables, endpoints y términos de programación conservan su denominación original.
- Los nombres de archivos y carpetas pueden estar en inglés.
- Los comentarios de código se escriben en español, salvo que una convención o herramienta exija otro idioma.
- La interfaz de usuario se presenta en español colombiano (`es-CO`).
