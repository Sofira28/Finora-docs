# ADR-0003: Un repositorio por aplicación y uno para la documentación

- **Estado:** Aceptada (30/10/2026)
- **Fecha:** 30/10/2026
- **Specs relacionadas:** `CLAUDE.md` §2, `specs/architecture.md`, [ADR-0001](ADR-0001-stack-tecnologico.md)

## Contexto

`finora-web` y `finora-back` son aplicaciones independientes (ADR-0001), con dependencias, configuración y pruebas separadas. Hasta el 30/10/2026 la carpeta del proyecto no estaba bajo control de versiones y `CLAUDE.md` prohibía `git init` sin solicitud expresa.

Al implementar `authentication` se crearon los repositorios de las dos aplicaciones, a solicitud de la usuaria. Las especificaciones (`specs/`), los ADR y guías (`docs/`) y las instrucciones del proyecto (`CLAUDE.md`) quedaron fuera de todo repositorio y podían perderse.

## Decisión

Tres repositorios en GitHub, cada uno con su propio historial:

| Carpeta | Repositorio | Contenido |
|---|---|---|
| raíz del proyecto | `Sofira28/Finora-docs` | `CLAUDE.md`, `specs/`, `docs/` |
| `finora-web/` | `Sofira28/Finora-web` | aplicación frontend |
| `finora-back/` | `Sofira28/Finora-back` | aplicación backend |

- El repositorio de documentación vive en la raíz y **ignora** `finora-web/` y `finora-back/` en su `.gitignore`: las aplicaciones no se versionan dos veces ni se usan submódulos.
- Rama base `main` en los tres. El trabajo de cada feature va en una rama `feat/<feature>`.
- Un cambio que toca spec y código se versiona en ambos repositorios. El mensaje de commit de la aplicación cita la tarea (`T-…`) para poder trazarlo.
- No se crean más repositorios ni se ejecuta `git init` sin solicitud expresa.

## Alternativas consideradas

- **Un único repositorio (monorepo):** fue la primera configuración, pero se descartó al conectar los repositorios remotos que la usuaria ya había creado por aplicación.
- **Copiar `specs/` y `docs/` en cada aplicación:** duplica la fuente de verdad y las copias se desincronizan.
- **Submódulos de Git:** complejidad adicional (actualizar punteros) sin beneficio con un equipo pequeño.

## Consecuencias

- La fuente de verdad (specs y contratos API) queda versionada y respaldada.
- Un cambio coordinado de contrato API implica commits en hasta tres repositorios; no hay atomicidad entre ellos. Se mitiga con la referencia a la tarea en cada commit.
- Clonar el proyecto completo requiere clonar `Finora-docs` y, dentro de él, `Finora-web` como `finora-web/` y `Finora-back` como `finora-back/`.
