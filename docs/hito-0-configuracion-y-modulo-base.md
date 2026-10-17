# Hito 0 — Configuración y Módulo Base (Usuarios, Auth, Contactos, Config)

> **Fuente de verdad:** [spec.md](../spec.md) — secciones 4 (stack), 5 (modelos), 6 (RN-01…RN-07), 7 (endpoints de Auth, Contactos y Configuración).

## Objetivos de aprendizaje

- Inicializar un monorepo con backend (Express + Mongoose) y frontend (HTML/CSS/JS vanilla) separados.
- Implementar autenticación con JWT: registro con código del plantel, login, roles y middlewares de protección.
- CRUD de contactos de confianza y configuración clave-valor (número de policía, código de registro).
- Crear un script de seed para los datos iniciales (el primer moderador no puede nacer de un endpoint público).
- Conectar el frontend con la API usando `fetch`, guardando el token en `sessionStorage`.

## Prerequisitos

- Node.js 20+, npm 10+, Git
- MongoDB local corriendo **o** cuenta en MongoDB Atlas con su connection string
- VS Code con Thunder Client y Live Server

---

## Tasks

Marcá cada checkbox `[x]` al terminar la tarea y verificá con Thunder Client antes de pasar a la siguiente.

### Repo y estructura

- [ ] **T0.1** Crear tu propio repositorio (`git init`) con la estructura del monorepo: `backend/`, `frontend/`, `docs/`, `README.md`, `.gitignore` (ignora `node_modules/` y `.env`). Commit: `chore: estructura inicial del monorepo`.
- [ ] **T0.2** En `backend/`: `npm init -y`, instalar dependencias y dejar `nodemon` como script `dev`.

```bash
cd backend
npm install express mongoose bcryptjs jsonwebtoken dotenv cors express-validator
npm install -D nodemon
```

```json
"scripts": {
  "dev": "nodemon src/server.js",
  "seed": "node src/seed/seed.js"
}
```

- [ ] **T0.3** Crear `.env` (local, **nunca** se commitea) y `.env.example` (sí se commitea):

```text
PORT=4000
MONGO_URI=mongodb://127.0.0.1:27017/seguridad-escolar
JWT_SECRET=cambia-este-secreto
JWT_EXPIRA=1d
```

### Backend — base del servidor

- [ ] **T0.4** `src/config/db.js`: conectar Mongoose a `MONGO_URI` (fallar rápido con mensaje claro si no conecta).
- [ ] **T0.5** `src/server.js`: Express + `express.json()` + `cors` con los orígenes de Live Server + ruta de prueba `GET /api/ping`.

```js
const cors = require('cors');
app.use(cors({ origin: ['http://127.0.0.1:5500', 'http://localhost:5500'] }));
```

> **Discutir en clase:** sin `cors`, el navegador bloquea TODO fetch del frontend aunque el backend funcione. El error aparece en la consola del navegador, no en la terminal del servidor.

- [ ] **T0.6** Helper de respuestas de error con formato unificado (RN-06): `{ "mensaje": "...", "errores": [...] }`, usándolo también para volcar los errores de `express-validator` (`validationResult`).

### Backend — modelos y seed

- [ ] **T0.7** `src/models/Usuario.js` según spec §5: `nombre`, `matricula` (unique), `grupoEscolar`, `passwordHash`, `rol` (enum `alumno|moderador`, default `alumno`), `activo`, `contactos` (máx. 2 subdocumentos `{ nombre, telefono }`), `creadoEn`.
- [ ] **T0.8** `src/models/Configuracion.js`: `clave` (unique), `valor`. Claves del MVP: `numeroPolicia`, `codigoRegistro`.
- [ ] **T0.9** `src/seed/seed.js` (RN-07): idempotente (se puede correr dos veces sin duplicar). Crea/actualiza:
  - moderador (matrícula `MOD-001`, contraseña hasheada con bcryptjs),
  - `codigoRegistro` (ej. `CBTIS-2026`),
  - `numeroPolicia` (10 dígitos),
  - 2–3 `puntosEncuentro` de ejemplo (se usan en el Hito 2; crear también el modelo `PuntoEncuentro` acá).

### Backend — autenticación (RN-01 a RN-05)

- [ ] **T0.10** `POST /api/auth/registro` (público): valida con `express-validator` (nombre, matrícula, grupo, password `isLength({ min: 8 })`), valida el `codigoRegistro` del body contra `configuraciones` (RN-01 → 403), captura matrícula duplicada (RN-02 → 409), hashea con `bcryptjs` y crea el usuario con rol `alumno`.
- [ ] **T0.11** `POST /api/auth/login` (público): busca por matrícula, compara con `bcrypt.compare`, emite JWT con `jsonwebtoken.sign({ id, rol }, JWT_SECRET, { expiresIn: JWT_EXPIRA })` y devuelve `{ token, usuario: { nombre, matricula, rol } }`.
- [ ] **T0.12** `src/middlewares/auth.js`: lee `Authorization: Bearer <token>`, verifica con `jwt.verify`, carga `req.usuario` (sin `passwordHash`); 401 si falta/inválido/expirado (RN-05).
- [ ] **T0.13** `src/middlewares/soloRol.js`: factory `soloRol('moderador')` → 403 si el rol no coincide.
- [ ] **T0.14** `GET /api/auth/yo` (Alumno): devuelve los datos del usuario del token.

### Backend — contactos y configuración

- [ ] **T0.15** `GET /api/contactos`, `POST /api/contactos` (valida formato de teléfono con `express-validator`; **máximo 2** se valida en el controlador → 409 si ya tiene 2), `DELETE /api/contactos/:idContacto` (solo contactos propios).
- [ ] **T0.16** `GET /api/config/policia` (Alumno), `PUT /api/config/policia` (Moderador, valida 10 dígitos), `PUT /api/config/registro` (Moderador, RN-04: rota el código).

### Frontend

- [ ] **T0.17** Estructura base: `index.html` (login), `registro.html`, `app.html`, `moderador.html`, `css/styles.css` (responsive con Flexbox), `js/api.js`.
- [ ] **T0.18** `js/api.js`: helper `fetch` que agrega `Authorization: Bearer` desde `sessionStorage`, hace `JSON.parse` y traduce el formato de error RN-06 a un mensaje mostrable.

```js
export async function api(path, { method = 'GET', body } = {}) {
  const token = sessionStorage.getItem('token');
  const res = await fetch(`http://localhost:4000/api${path}`, {
    method,
    headers: {
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
    },
    body: body ? JSON.stringify(body) : undefined,
  });
  const data = await res.json().catch(() => ({}));
  if (!res.ok) throw new Error(data.mensaje || `Error ${res.status}`);
  return data;
}
```

- [ ] **T0.19** Página de registro: formulario (nombre, matrícula, grupo, password, código de registro) → `POST /api/auth/registro` → muestra errores del formato unificado.
- [ ] **T0.20** Página de login: → `POST /api/auth/login` → guarda token y usuario en `sessionStorage` → redirige a `app.html` (o `moderador.html` según rol).
- [ ] **T0.21** `app.html`: al cargar, si no hay token vuelve a `index.html`; muestra datos de `GET /api/auth/yo` y botón de cerrar sesión (borra `sessionStorage`).
- [ ] **T0.22** Pantalla de contactos: listar, agregar (máx. 2), borrar.
- [ ] **T0.23** Sección moderador: cambiar número de policía y rotar código de registro.

### Verificación (Thunder Client)

- [ ] **T0.24** Correr `npm run seed` y verificar toda la colección de casos:

| Caso | Esperado |
| :-- | :-- |
| `POST /registro` sin código o con código viejo | 403 + `mensaje` |
| `POST /registro` matrícula repetida | 409 |
| `POST /registro` password de 5 chars | 400 + `errores` de validación |
| `POST /login` datos correctos | 200 + `token` |
| `GET /yo` sin token / token inventado | 401 |
| `GET /contactos` con token de alumno | 200 |
| `POST /contactos` un 3er contacto | 409 |
| `PUT /config/policia` con token de alumno | 403 |
| `PUT /config/registro` con token de moderador | 200 |

### Checkpoints de commit

- [ ] `chore: estructura inicial del monorepo` (T0.1)
- [ ] `feat: servidor express con mongo, cors y formato de error` (T0.2–T0.6)
- [ ] `feat: modelos usuario y configuracion con seed inicial` (T0.7–T0.9)
- [ ] `feat: registro y login con jwt y middlewares de rol` (T0.10–T0.14)
- [ ] `feat: crud de contactos y endpoints de configuracion` (T0.15–T0.16)
- [ ] `feat: frontend con login, registro y sesion en sessionStorage` (T0.17–T0.23)
- [ ] `test: coleccion thunder client del modulo base` (T0.24)

---

## Investigación (equipos)

Cada equipo documenta un tema y lo presenta al cerrar el hito:

- [ ] **Equipo A — Contraseñas:** ¿por qué `bcryptjs` y no SHA-256? ¿Qué es la *sal* y qué problema resuelve contra los rainbow tables?
- [ ] **Equipo B — JWT vs sesiones:** ¿dónde vive el estado en cada enfoque? ¿Qué pasa si queremos "cerrar todas las sesiones" de un usuario con JWT?
- [ ] **Equipo C — Almacenamiento en el navegador:** `sessionStorage` vs `localStorage` vs cookies. ¿Qué pasa al cerrar la pestaña? ¿Qué es XSS y por qué importa dónde guardamos el token?
- [ ] **Equipo D — CORS:** ¿quién aplica realmente la restricción, el navegador o el servidor? ¿Por qué Thunder Client no sufre CORS pero el frontend sí?

## Criterio de cierre del hito

El hito está completo cuando: la colección Thunder Client pasa en verde, un alumno puede registrarse con código, loguearse, ver sus datos y gestionar sus contactos desde el navegador, y el moderador puede rotar código y número de policía. Recién ahí: **Hito 1**.
