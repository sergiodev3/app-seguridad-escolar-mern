# Hito 3 — Documentación de la API y Cierre

> **Fuente de verdad:** [spec.md](../spec.md) completo — este hito verifica el sistema contra las 16 RN.
> **Depende de:** Hitos 0, 1 y 2 completos.

## Objetivos de aprendizaje

- Documentar la API con estándar OpenAPI y servirla con Swagger UI (que la documentación viva junto al código).
- Consolidar la colección de pruebas completa en Thunder Client.
- Hacer una pasada final de calidad contra el spec: cada RN verificada, README del proyecto terminado.

## Prerequisitos

- Hitos 0–2 cerrados (todos sus checkboxes en `[x]`).

---

## Tasks

### Backend — Swagger

- [ ] **T3.1** Instalar `swagger-ui-express` y `yaml`; crear `src/docs/openapi.yaml`.
- [ ] **T3.2** Servir la documentación y cargar el YAML al arrancar:

```js
const swaggerUi = require('swagger-ui-express');
const YAML = require('yaml');
const fs = require('fs');
const openapi = YAML.parse(fs.readFileSync(`${__dirname}/docs/openapi.yaml`, 'utf8'));
app.use('/api/docs', swaggerUi.serve, swaggerUi.setup(openapi));
```

- [ ] **T3.3** En `openapi.yaml`: `info`, `servers` (`http://localhost:4000`), componente de seguridad `bearerAuth` (JWT) y esquema de error unificado (RN-06).
- [ ] **T3.4** Documentar auth: `/api/auth/registro`, `/api/auth/login`, `/api/auth/yo`.
- [ ] **T3.5** Documentar contactos y configuración (incluye `PUT /api/config/registro`).
- [ ] **T3.6** Documentar alertas (`POST`, `GET`, `PATCH`).
- [ ] **T3.7** Documentar puntos y grupos (incluye `unirse`/salir y los 409 de RN-10/RN-11).
- [ ] **T3.8** Probar en el navegador: `http://localhost:4000/api/docs` — autorizar con `Authorize` (pegar el token del login) y ejecutar al menos un endpoint protegido desde la UI.

### Consolidación

- [ ] **T3.9** Exportar/commitear la colección Thunder Client completa (carpeta `thunder-tests/`) organizada por módulo: Auth, Contactos, Config, Alertas, Puntos, Grupos.
- [ ] **T3.10** README del proyecto terminado: cómo correr backend + frontend, variables de entorno, seed, credenciales de prueba (moderador del seed) y capturas de las pantallas principales.

### Checklist final contra el spec

- [ ] **T3.11** Verificar **cada** regla de negocio con una prueba concreta y marcarla:

| RN | ¿Verificada? |
| :-- | :-- |
| RN-01 código de registro → 403 | [ ] |
| RN-02 matrícula única → 409 | [ ] |
| RN-03 password ≥ 8 → 400 | [ ] |
| RN-04 solo moderador rota el código → 403 | [ ] |
| RN-05 JWT 1 día, sessionStorage, 401 sin token | [ ] |
| RN-06 formato de error unificado en TODAS las rutas | [ ] |
| RN-07 seed idempotente | [ ] |
| RN-08 long-press 2 s confirma alerta | [ ] |
| RN-09 número policía: moderador cambia, alumno lee | [ ] |
| RN-10 cupo 2–10, lleno → 409 | [ ] |
| RN-11 duplicado en grupo → 409 | [ ] |
| RN-12 solo grupos de hoy | [ ] |
| RN-13 punto inexistente → 400 | [ ] |
| RN-14 integrantes solo nombre + grupoEscolar | [ ] |
| RN-15 creador sale → traspaso; vacío → elimina | [ ] |
| RN-16 moderador administra puntos (desactivar) | [ ] |

### Checkpoints de commit

- [ ] `docs: swagger ui con openapi para auth y contactos` (T3.1–T3.5)
- [ ] `docs: swagger para alertas, puntos y grupos` (T3.6–T3.8)
- [ ] `test: coleccion thunder client completa` (T3.9)
- [ ] `docs: readme final y checklist de reglas de negocio` (T3.10–T3.11)

---

## Investigación (equipos)

- [ ] **Equipo A — OpenAPI:** ¿qué es una especificación de API y por qué es un contrato? ¿Qué herramientas consumen un `openapi.yaml` además de Swagger UI (generadores de cliente, Postman, mocks)?
- [ ] **Equipo B — Seguridad siguiente nivel:** investigar `helmet`, `express-rate-limit` (¿qué pasa si alguien dispara 1000 alertas por minuto?) y por qué el `.env` jamás se commitea.
- [ ] **Equipo C — Deploy:** opciones gratuitas o baratas para backend (Railway, Render) y frontend (Vercel, Netlify, GitHub Pages). ¿Qué cambia con HTTPS para la Geolocation API (spec §8)?
- [ ] **Equipo D — Del MVP al producto:** ¿qué faltaría para que esto lo use realmente el plantel? (notificaciones push, moderación de contenido, auditoría de alertas, pruebas automatizadas).

## Cierre del proyecto

Con este hito termina el proyecto guía. Cada equipo:

1. Tiene su repo con el historial de commits completo por hito.
2. Presenta su investigación.
3. Arranca su **proyecto de equipo** (otra problemática) reutilizando la base: monorepo, auth JWT con roles, formato de error unificado, seed y Swagger.
