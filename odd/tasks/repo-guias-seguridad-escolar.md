# Feature: Repo guía "Seguridad Escolar" (esqueleto + hitos-guía)

**Objetivo**: Armar el repositorio ejemplo que el docente usará en clase: spec.md sin vacíos funcionales, esqueleto monorepo (backend/frontend), y 4 guías de hito con tasks marcables estilo ODD (`[ ]`/`[x]`), checkpoints de commit e investigación por equipos.

**Problema**: El spec.md actual tiene vacíos (moderador no nace, código de registro sin gestión, puntos sin CRUD, CORS ausente, comportamiento del creador al salir indefinido) y no existe estructura de repo ni guías paso a paso.

**Alcance autorizado**: Solo documentación y esqueleto (markdown, .gitignore, READMEs placeholder, git init + commits). NO se implementa código de la app; los snippets viven dentro de las guías como referencia.

**Ruta**: Direct inline (contenido docente que requiere el contexto completo de la conversación; delegar costaría más que escribir).

**TDD**: N/A — no hay código ejecutable, solo documentación. Checks: readback estructural de cada archivo.

**Estrategia de entrega**: single-pr n/a (repo nuevo, sin remoto); commits por unidad de trabajo en main.

## Tasks

- [x] T1 — Actualizar spec.md: renumerar RN (llenar 04–07 + nuevas), endpoints faltantes (PUT /api/config/registro, CRUD /api/puntos), cors, .env.example, seed, JWT/sessionStorage, formato de error, límites del navegador, timezone fecha.
- [x] T2 — Esqueleto root: README.md (guía general + índice de hitos + dinámica de clase), .gitignore, backend/README.md, frontend/README.md.
- [x] T3 — docs/hito-0-configuracion-y-modulo-base.md (tasks T0.1–T0.24, commits, verificación, investigación equipos A–D).
- [x] T4 — docs/hito-1-boton-de-alerta.md (tasks T1.1–T1.14).
- [x] T5 — docs/hito-2-caminemos-juntos.md (tasks T2.1–T2.15).
- [x] T6 — docs/hito-3-documentacion-y-cierre.md (tasks T3.1–T3.11 + checklist RN-01…RN-16).
- [x] T7 — git init (repo ya existía sin commits) + rama main + commits por unidad de trabajo (conventional commits).
- [x] T8 — Readback final de consistencia (RN ↔ endpoints ↔ hitos): verificado — spec tiene RN-01…RN-16 sin huecos, cada endpoint del spec aparece en un hito, hito-3 verifica las 16 RN.

## Evidencia / progreso

- Commits en `main`:
  - `5e76406` docs: actualizar spec con reglas, endpoints y limites del navegador
  - `1baeb0f` chore: esqueleto del monorepo con guia general y readmes
  - `e9a704d` docs: guia hito 0 configuracion y modulo base
  - `121e56c` docs: guia hito 1 boton de alerta
  - `594d834` docs: guia hito 2 caminemos juntos
  - `51459ae` docs: guia hito 3 documentacion y cierre
- Checks: readback de spec.md completo (144 líneas, RN continuas, §8 presente); guías con tasks `[ ]`, checkpoints de commit, verificación Thunder Client e investigación por equipos.
- TDD: N/A (solo documentación). Contenido pasivo → verificación = readback estructural.

## Próximo paso

- Feature completa. Pendiente del usuario: crear el remoto en GitHub y `git push` (no autorizado en esta sesión), y arrancar la clase con Hito 0.
