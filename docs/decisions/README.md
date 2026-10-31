# Registro de decisiones de arquitectura (ADR)

Cada decisión arquitectónica relevante se documenta en un archivo `ADR-NNNN-titulo-corto.md` dentro de esta carpeta.

## Formato

```markdown
# ADR-NNNN: <Título>

- **Estado:** Propuesta | Aceptada | Rechazada | Reemplazada por ADR-XXXX
- **Fecha:** DD/MM/AAAA
- **Specs relacionadas:** <rutas>

## Contexto
<Problema y restricciones.>

## Decisión
<Qué se decidió.>

## Alternativas consideradas
<Opciones y por qué se descartaron.>

## Consecuencias
<Impacto positivo, negativo y riesgos.>
```

## Decisiones pendientes

Las decisiones abiertas se listan en:

- [`specs/architecture.md`](../../specs/architecture.md) §3 (DP-ARQ-*)
- [`specs/database.md`](../../specs/database.md) §3 (DP-DB-*)
- [`specs/api-contracts.md`](../../specs/api-contracts.md) §2 (DP-API-*)
- [`specs/security.md`](../../specs/security.md) §6 (DP-SEC-*)

## ADRs registrados

| ADR | Título | Estado |
|---|---|---|
| [ADR-0001](ADR-0001-stack-tecnologico.md) | Stack tecnológico inicial de finora-web y finora-back | Aceptada |
| [ADR-0002](ADR-0002-autenticacion-google-y-correo.md) | Inicio de sesión con Google, envío de correos y estado de autenticación | Aceptada |
| [ADR-0003](ADR-0003-repositorios.md) | Un repositorio por aplicación y uno para la documentación | Aceptada |
| [ADR-0004](ADR-0004-row-level-security.md) | Row-Level Security con un rol de aplicación sin privilegios | Aceptada |
