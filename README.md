# Seguridad Escolar — Proyecto Guía Paso a Paso

Aplicación web que ataca un problema real: **la inseguridad alrededor del plantel en horarios de entrada y salida**. MVP con botón de alerta (ubicación + llamada a la policía + aviso por WhatsApp) y "Caminemos Juntos" (grupos de salida por ruta con punto de encuentro).

Este repositorio es una **guía docente**: se recorre en clase hito por hito. No es una solución para clonar y correr.

## Dinámica de clase

1. El docente explica y construye cada hito en vivo usando este repo como mapa.
2. Cada alumno/equipo crea **su propio repositorio** y lo construye a la par.
3. Las guías de hito tienen **tasks con checkboxes** (`[ ]` → `[x]`): se marcan al terminar cada tarea.
4. En cada **checkpoint de commit** se hace commit en el repo propio usando [conventional commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, `chore:`).
5. El **trabajo en equipo** vive en las secciones de *Investigación* de cada hito: los equipos documentan y presentan.

## Estructura del monorepo

```text
seguridad-escolar/
├── README.md        # esta guía general
├── spec.md          # especificación: la fuente de verdad del proyecto
├── docs/            # guías de hito (empezar por hito-0)
├── backend/         # API REST (Node.js + Express + MongoDB) — se construye en Hito 0
└── frontend/        # Interfaz (HTML + CSS + JS vanilla, Leaflet) — se construye en Hito 0
```

## Hitos

| Hito | Guía | Qué se construye |
| :-- | :-- | :-- |
| 0 | [docs/hito-0-configuracion-y-modulo-base.md](docs/hito-0-configuracion-y-modulo-base.md) | Monorepo, Express + MongoDB, auth JWT, roles, contactos, configuraciones, seed, frontend login/registro |
| 1 | [docs/hito-1-boton-de-alerta.md](docs/hito-1-boton-de-alerta.md) | Botón de alerta: geolocalización, mapa Leaflet, long-press 2 s, llamada `tel:`, aviso `wa.me`, panel del moderador |
| 2 | [docs/hito-2-caminemos-juntos.md](docs/hito-2-caminemos-juntos.md) | Puntos de encuentro (CRUD moderador), grupos de hoy, unirse/salir con cupo, populate |
| 3 | [docs/hito-3-documentacion-y-cierre.md](docs/hito-3-documentacion-y-cierre.md) | Swagger/OpenAPI, colección Thunder Client completa, checklist final contra las RN del spec |

## Requisitos previos

- Node.js 20+ y npm 10+
- MongoDB local o MongoDB Atlas
- Git
- VS Code con extensiones: **Thunder Client** (pruebas de API) y **Live Server** (frontend)

## Cómo usar las guías

1. Leé [spec.md](spec.md) completo antes de empezar: es la fuente de verdad (modelos, reglas de negocio RN-01…RN-16 y endpoints).
2. Abrí la guía del hito y seguí las tasks en orden (backend → frontend → verificación).
3. Verificá cada endpoint con Thunder Client antes de marcar la task como hecha.
4. Commiteá en cada checkpoint; el mensaje del commit referencia el hito.

## Convenciones

- **Commits:** conventional commits, en español, sin atribución de herramientas.
- **UI y comentarios del código:** en español (proyecto escolar para México).
- **Errores de la API:** formato unificado `{ "mensaje": "...", "errores": [...] }` (RN-06).
- **Ramas (opcional para equipos):** una rama por módulo/hito y PR al terminar.
