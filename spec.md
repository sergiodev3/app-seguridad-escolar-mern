# Especificación de Proyecto (spec.md) - App de Seguridad Escolar (MVP Simplificado)

## 1. Descripción General
**Proyecto:** Aplicación web responsiva para enfrentar la inseguridad alrededor del plantel en horarios de entrada y salida.

Este MVP está simplificado para enfocarse exclusivamente en funcionalidades de emergencia inmediata y acompañamiento grupal, utilizando un stack basado en Node.js, Express y MongoDB, organizado en una estructura de monorepo.

## 2. Alcance del MVP
El sistema se compone de los siguientes módulos funcionales:

*   **Módulo Base (Usuarios y Roles):** Registro con código de grupo, inicio de sesión con JWT, roles de sistema (alumno y moderador), y gestión de contactos de confianza.
*   **Módulo 1: Botón de Alerta:** Botón que al mantener presionado 2 s registra la alerta con ubicación y abre la llamada a la policía municipal; opción de avisar a un contacto por WhatsApp.
*   **Módulo 2: Caminemos Juntos:** Grupos de salida por ruta, con punto de encuentro predefinido, franja y cupo; caducan al terminar el día.

## 3. Arquitectura y Estructura del Proyecto (Monorepo)
El proyecto utilizará una arquitectura cliente-servidor organizada en un **monorepo** con dos carpetas principales independientes para separar claramente las responsabilidades:

```text
seguridad-escolar/
├── backend/          # API REST con Node.js, Express y MongoDB
└── frontend/         # Interfaz de usuario (HTML, CSS, JS) servida localmente
```

## 4. Stack Tecnológico y Herramientas de Desarrollo

### Backend (`/backend`)
*   **Entorno y Framework:** Node.js + Express.
*   **Base de Datos:** **MongoDB** gestionado a través del ODM **Mongoose**.
*   **Validación de Datos:** **`express-validator`** implementado como middleware en las rutas de Express.
*   **Seguridad y Sesiones:** `bcryptjs` (cifrado con hash y sal) y `jsonwebtoken` (JWT) para la autenticación sin guardar sesiones en el servidor.
*   **Variables de Entorno:** `dotenv` para manejar credenciales, puerto y secretos de JWT. Archivo `.env.example` commiteado al repo (`PORT`, `MONGO_URI`, `JWT_SECRET`, `JWT_EXPIRA`); el `.env` real nunca se commitea.
*   **CORS:** middleware `cors` configurado con los orígenes locales del frontend en desarrollo (Live Server: `http://127.0.0.1:5500` y `http://localhost:5500`). Sin esto, todo `fetch` del frontend falla por navegador.
*   **Datos Iniciales:** script `npm run seed` que crea el primer moderador, el código de registro vigente, el número de policía y puntos de encuentro de ejemplo. No existe ningún endpoint público que cree moderadores.
*   **Documentación API:** `swagger-ui-express` + `yaml` para generar la documentación de la API estándar.
*   **Herramienta de Desarrollo:** **`nodemon`** configurado en el `package.json` para reiniciar automáticamente el servidor al detectar cambios en el código.
*   **Pruebas de API:** Extensión **Thunder Client** en VS Code para probar los endpoints localmente sin necesidad de salir del editor.

### Frontend (`/frontend`)
*   **Lenguajes:** HTML5, CSS3 (Flexbox, media queries), JavaScript con módulos ES.
*   **Mapas y Ubicación:** Leaflet + OpenStreetMap y Geolocation API del navegador.
*   **Herramienta de Desarrollo:** Extensión **Live Server** en VS Code para levantar la interfaz en un servidor local durante la fase de desarrollo y pruebas.

## 5. Modelos de Datos (Esquemas de Mongoose)

**Colección `usuarios`**
*   `nombre`: String (Requerido)
*   `matricula`: String (Requerido, Único)
*   `grupoEscolar`: String (Requerido)
*   `passwordHash`: String (Requerido)
*   `rol`: String (Enum: `['alumno', 'moderador']`, Default: `'alumno'`)
*   `activo`: Boolean (Default: `true`)
*   `contactos`: Array de Subdocumentos `[{ nombre: String, telefono: String }]` (Máximo 2)
*   `creadoEn`: Date (Default: `Date.now`)

**Colección `alertas`**
*   `usuarioId`: ObjectId (Ref: `Usuario`, Requerido)
*   `lat`: Number
*   `lng`: Number
*   `atendida`: Boolean (Default: `false`)
*   `creadaEn`: Date (Default: `Date.now`)

**Colección `puntosEncuentro`**
*   `nombre`: String (Requerido)
*   `descripcion`: String
*   `lat`: Number (Requerido)
*   `lng`: Number (Requerido)
*   `activo`: Boolean (Default: `true`)

**Colección `grupos`** 
*   `creadorId`: ObjectId (Ref: `Usuario`, Requerido)
*   `puntoId`: ObjectId (Ref: `PuntosEncuentro`, Requerido)
*   `ruta`: String (Requerido)
*   `fecha`: String (Formato 'YYYY-MM-DD', Requerido)
*   `franja`: String (Enum: `['entrada', 'salida']`, Requerido)
*   `cupo`: Number (Min: 2, Max: 10, Requerido)
*   `integrantes`: Array de ObjectId `[Ref: 'Usuario']` (El creador se incluye automáticamente como primer integrante)
*   `creadoEn`: Date (Default: `Date.now`)

**Colección `configuraciones`** (documento clave-valor; claves usadas en el MVP: `numeroPolicia`, `codigoRegistro`)
*   `clave`: String (Requerido, Único)
*   `valor`: String (Requerido)

> Nota: `grupos.fecha` es String `'YYYY-MM-DD'` comparado contra la fecha local del servidor. Es una simplificación aceptable para el MVP; el caveat de timezones se discute en clase (Hito 2).
> Nota: el máximo de 2 contactos no se puede validar con `express-validator` (es un arreglo de subdocumentos); se valida en el controlador antes de insertar.

## 6. Reglas de Negocio y Validaciones (`express-validator`)

| Regla | Descripción | Manejo |
| :--- | :--- | :--- |
| **RN-01** | Solo se registra quien tenga el código de registro vigente del plantel; el código vive en `configuraciones` (`codigoRegistro`) y se envía en el body del registro para validarlo contra la config. | Controlador. Respuesta 403. |
| **RN-02** | La matrícula es única. | Mongoose (`unique: true`) + Controlador. Respuesta 409. |
| **RN-03** | La contraseña tiene mínimo 8 caracteres y se guarda solo cifrada. | `express-validator` (`isLength({ min: 8 })`). Respuesta 400. |
| **RN-04** | El código de registro solo lo rota el moderador. | Middleware `soloRol('moderador')` en `PUT /api/config/registro`. Respuesta 403. |
| **RN-05** | El JWT expira en 1 día (`JWT_EXPIRA`); el frontend lo guarda en `sessionStorage` y lo envía como `Authorization: Bearer <token>`. | Backend + Frontend. Respuesta 401 si falta, es inválido o expiró. |
| **RN-06** | Toda respuesta de error usa el formato unificado `{ "mensaje": "...", "errores": [...] }` (`errores` presente solo cuando falla `express-validator`). | Helper de respuesta en el backend. |
| **RN-07** | Los datos iniciales (moderador, código de registro, número de policía, puntos de encuentro) se crean solo con `npm run seed`. | Script de seed. |
| **RN-08** | La alerta se confirma manteniendo el botón 2 segundos. | Frontend (`setTimeout` en evento `pointerdown`). |
| **RN-09** | El número de la policía se lee de configuración y solo el moderador lo cambia. | Middleware `soloRol('moderador')`. Respuesta 403. |
| **RN-10** | Un grupo tiene cupo de 2 a 10; nadie se une si está lleno. | `express-validator` + Controlador. Respuesta 409. |
| **RN-11** | Un alumno no puede estar dos veces en el mismo grupo. | Controlador (verificar array `integrantes`). Respuesta 409. |
| **RN-12** | Solo se listan grupos con fecha de hoy (caducan al día siguiente). | Modelo/Controlador (Query por fecha actual). |
| **RN-13** | El punto de encuentro se elige de la lista del moderador; nunca un domicilio. | Controlador (validar existencia del ObjectId). Respuesta 400. |
| **RN-14** | Los integrantes de un grupo se muestran solo con nombre y grupo escolar; nunca teléfono. | Mongoose (`.populate('integrantes', 'nombre grupoEscolar')`). |
| **RN-15** | Si el creador sale del grupo, el integrante restante más antiguo pasa a ser el creador; si el grupo queda vacío, se elimina. | Controlador de `DELETE /api/grupos/:id/unirse`. |
| **RN-16** | Los puntos de encuentro solo los crea, edita y desactiva el moderador (`activo: false`, nunca borrado físico mientras haya grupos que los referencien). | Middleware `soloRol('moderador')`. Respuesta 403. |

## 7. Endpoints de la API REST

### Autenticación e Inicio
*   `POST /api/auth/registro`: Crea cuenta de alumno. Valida matrícula, nombre, grupo y password mediante `express-validator`, y el `codigoRegistro` contra la configuración (RN-01). (Público).
*   `POST /api/auth/login`: Autentica y devuelve token JWT y datos del usuario. (Público).
*   `GET /api/auth/yo`: Devuelve los datos del usuario del token. (Alumno).

### Contactos de Confianza
*   `GET /api/contactos`: Devuelve sus contactos de confianza. (Alumno).
*   `POST /api/contactos`: Agrega un contacto (máximo 2). Valida formato de teléfono. (Alumno).
*   `DELETE /api/contactos/:idContacto`: Borra un contacto suyo. (Alumno).

### Botón de Alerta (Módulo 1)
*   `POST /api/alertas`: Registra alerta con `lat`/`lng` opcionales. (Alumno).
*   `GET /api/alertas`: Lista las alertas del día. (Moderador).
*   `PATCH /api/alertas/:id`: Marca como atendida. (Moderador).

### Caminemos Juntos (Módulo 2)
*   `GET /api/puntos`: Lista los puntos de encuentro activos. (Alumno).
*   `POST /api/puntos`: Crea un punto de encuentro. Valida nombre, lat y lng. (Moderador).
*   `PUT /api/puntos/:id`: Edita un punto de encuentro. (Moderador).
*   `DELETE /api/puntos/:id`: Desactiva un punto (`activo: false`), nunca borrado físico (RN-16). (Moderador).
*   `GET /api/grupos`: Obtiene los grupos de hoy con integrantes y cupo, poblando los datos de los usuarios. (Alumno).
*   `POST /api/grupos`: Crea grupo; el creador queda como integrante. `express-validator` verifica franja (`entrada`/`salida`) y cupo (2 a 10). (Alumno).
*   `POST /api/grupos/:id/unirse`: Agrega al usuario logueado al arreglo de `integrantes` del grupo si hay cupo y no es miembro. (Alumno).
*   `DELETE /api/grupos/:id/unirse`: Sale del grupo eliminando al usuario del arreglo; si era el creador, la propiedad pasa al integrante más antiguo y si queda vacío el grupo se elimina (RN-15). (Alumno).

### Configuración
*   `GET /api/config/policia`: Devuelve el número de la policía municipal. (Alumno).
*   `PUT /api/config/policia`: Actualiza el número. Valida que sean 10 dígitos mediante `express-validator`. (Moderador).
*   `PUT /api/config/registro`: Rota el código de registro del plantel (RN-04). (Moderador).

## 8. Límites del Navegador (expectativas del MVP)

*   **Llamada a la policía:** una página web no puede marcar por sí sola; se usa un enlace `tel:<numero>` que el alumno toca para iniciar la llamada en su teléfono.
*   **Geolocalización:** la Geolocation API solo funciona en contextos seguros (HTTPS o `localhost`/`127.0.0.1`). En desarrollo con Live Server funciona; si se desplegara sobre HTTP plano, no.
*   **WhatsApp:** el aviso al contacto es 100% frontend con un enlace `https://wa.me/52<telefono>?text=<mensaje>`; el backend no interviene.
*   **Botón 2 segundos:** se implementa con `pointerdown` / `pointerup` / `setTimeout` y feedback visual de progreso (RN-08).