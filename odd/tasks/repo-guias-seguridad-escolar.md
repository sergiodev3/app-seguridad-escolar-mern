# Feature: Repo guía "Seguridad Escolar" (esqueleto + hitos-guía)

**Objetivo**: Armar el repositorio ejemplo que el docente usará en clase: spec.md sin vacíos funcionales, esqueleto monorepo (backend/frontend), y 4 guías de hito con tasks marcables estilo ODD (`[ ]`/`[x]`), checkpoints de commit e investigación por equipos.

**Problema**: El spec.md actual tiene vacíos (moderador no nace, código de registro sin gestión, puntos sin CRUD, CORS ausente, comportamiento del creador al salir indefinido) y no existe estructura de repo ni guías paso a paso.

**Alcance autorizado**: Solo documentación y esqueleto (markdown, .gitignore, READMEs placeholder, git init + commits). NO se implementa código de la app; los snippets viven dentro de las guías como referencia.

**Ruta**: Direct inline (contenido docente que requiere el contexto completo de la conversación; delegar costaría más que escribir).

**TDD**: N/A — no hay código ejecutable, solo documentación. Checks: readback estructural de cada archivo.

**Estrategia de entrega**: single-pr n/a (repo nuevo, sin remoto); commits por unidad de trabajo en main.

## Tasks

- [ ] T1 — Actualizar spec.md: renumerar RN (llenar 04–07 + nuevas), endpoints faltantes (PUT /api/config/registro, CRUD /api/puntos), cors, .env.example, seed, JWT/sessionStorage, formato de error, límites del navegador, timezone fecha.
- [ ] T2 — Esqueleto root: README.md (guía general + índice de hitos + dinámica de clase), .gitignore, backend/README.md, frontend/README.md.
- [ ] T3 — docs/hito-0-configuracion-y-modulo-base.md (tasks T0.x, commits, verificación, investigación).
- [ ] T4 — docs/hito-1-boton-de-alerta.md.
- [ ] T5 — docs/hito-2-caminemos-juntos.md.
- [ ] T6 — docs/hito-3-documentacion-y-cierre.md.
- [ ] T7 — git init + commits por unidad de trabajo (conventional commits).
- [ ] T8 — Readback final de consistencia (RN ↔ endpoints ↔ hitos) + espejo Engram actualizado.

## Evidencia / progreso

- (pendiente)

## Próximo paso

- T1: editar spec.md.
