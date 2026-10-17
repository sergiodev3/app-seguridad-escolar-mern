# Hito 2 — Caminemos Juntos (Módulo 2)

> **Fuente de verdad:** [spec.md](../spec.md) — modelos `puntosEncuentro` y `grupos` (§5), RN-10 a RN-16, endpoints de Caminemos Juntos (§7).
> **Depende de:** Hito 1 completo.

## Objetivos de aprendizaje

- CRUD protegido por rol: el moderador administra los puntos de encuentro (RN-16).
- Modelar relaciones con ObjectId y usar `.populate()` con selección de campos para no filtrar datos sensibles (RN-14).
- Operadores atómicos de Mongo: `$push`/`$pull` para unirse/salir de un grupo y la condición de carrera del cupo (RN-10, RN-11).
- Reglas de ciclo de vida: grupos de hoy (RN-12) y traspaso de propiedad del creador (RN-15).

## Prerequisitos

- Hito 1 cerrado.
- Modelo `PuntoEncuentro` ya creado en el seed del Hito 0 (T0.9) con puntos de ejemplo.

---

## Tasks

### Backend — puntos de encuentro

- [ ] **T2.1** `GET /api/puntos` (Alumno): solo puntos con `activo: true`.
- [ ] **T2.2** `POST /api/puntos` y `PUT /api/puntos/:id` (Moderador): valida nombre, `lat` y `lng` con `express-validator`.
- [ ] **T2.3** `DELETE /api/puntos/:id` (Moderador): **desactiva** (`activo: false`), nunca borrado físico (RN-16). Explicar por qué: los grupos históricos lo referencian.

### Backend — grupos

- [ ] **T2.4** `src/models/Grupo.js` según spec §5: `creadorId`, `puntoId`, `ruta`, `fecha` (String `'YYYY-MM-DD'`), `franja` (enum `entrada|salida`), `cupo` (min 2, max 10), `integrantes` (array de ObjectId ref `Usuario`), `creadoEn`.
- [ ] **T2.5** Helper `fechaHoy()`: devuelve `'YYYY-MM-DD'` **local** (no `toISOString()` sin más, que usa UTC — discutir el caveat de timezone del spec §5).
- [ ] **T2.6** `GET /api/grupos` (Alumno): solo `fecha: fechaHoy()` (RN-12), con `.populate('puntoId', 'nombre lat lng')` y `.populate('integrantes', 'nombre grupoEscolar')` (RN-14: nunca teléfono ni matrícula ajena).
- [ ] **T2.7** `POST /api/grupos` (Alumno): `express-validator` valida `ruta`, `franja` (enum) y `cupo` (2–10, RN-10); el controlador valida que `puntoId` exista y esté activo (RN-13 → 400) y crea el grupo con el creador como primer integrante.
- [ ] **T2.8** `POST /api/grupos/:id/unirse` (Alumno): rechaza si ya es miembro (RN-11 → 409) y si `integrantes.length >= cupo` (RN-10 → 409). Usar actualización atómica con `$push` y filtro de cupo:

```js
const grupo = await Grupo.findOneAndUpdate(
  { _id: id, fecha: fechaHoy(), integrantes: { $ne: req.usuario.id },
    $expr: { $lt: [{ $size: '$integrantes' }, '$cupo'] } },
  { $push: { integrantes: req.usuario.id } },
  { new: true }
);
if (!grupo) return res.status(409).json({ mensaje: 'Grupo lleno, ya eres miembro o no es de hoy' });
```

> **Discutir en clase:** si hiciéramos `find` + `push` + `save` en dos pasos, dos alumnos podrían unirse al último lugar **al mismo tiempo**. El filtro dentro de `findOneAndUpdate` es atómico: Mongo lo aplica como una sola operación. (Para el MVP basta entender el problema; el mensaje 409 puede refinarse distinguiendo los casos.)

- [ ] **T2.9** `DELETE /api/grupos/:id/unirse` (Alumno): `$pull` del usuario. Luego aplicar RN-15:
  - si salió el creador y quedan integrantes → `creadorId = integrantes[0]` (el más antiguo);
  - si el grupo quedó vacío → eliminarlo (`deleteOne`).

### Frontend

- [ ] **T2.10** Sección "Caminemos Juntos" en `app.html`: lista de grupos de hoy (tarjetas con ruta, punto de encuentro, franja, `integrantes.length/cupo`), botón Unirse/Salir según corresponda, y filtro por franja (`entrada`/`salida`).
- [ ] **T2.11** Formulario "Crear grupo": selector de punto de encuentro cargado desde `GET /api/puntos`, ruta (texto), franja y cupo (2–10).
- [ ] **T2.12** Mapa con los puntos de encuentro (`GET /api/puntos`) y, al abrir un grupo, marcador en su punto.
- [ ] **T2.13** Sección moderador en `moderador.html`: alta/edición/desactivación de puntos de encuentro (puede ser formulario simple + lista con `activo`).

### Verificación (Thunder Client)

- [ ] **T2.14** Casos de API:

| Caso | Esperado |
| :-- | :-- |
| `POST /puntos` con token de alumno | 403 |
| `POST /puntos` moderador sin `lat` | 400 + `errores` |
| `DELETE /puntos/:id` moderador | 200 + punto queda `activo: false` y no aparece en `GET /puntos` de alumno |
| `POST /grupos` con `cupo: 1` o `franja: 'mediodia'` | 400 |
| `POST /grupos` con `puntoId` inexistente | 400 |
| `POST /grupos` válido | 201/200 y el creador está en `integrantes` |
| Unirse dos veces | 409 |
| Unirse con grupo lleno (cupo 2, probar con 2 usuarios) | 409 |
| `GET /grupos` | solo los de hoy, integrantes con **solo** `nombre` y `grupoEscolar` |
| Sale el creador quedando 2+ | 200 y `creadorId` cambió al más antiguo |
| Sale el último integrante | el grupo desaparece de `GET /grupos` |

- [ ] **T2.15** Prueba de punta a punta en el navegador con **dos usuarios reales** (dos perfiles de navegador o uno normal + uno incógnito): uno crea el grupo, el otro se une, el creador sale y la propiedad se transfiere.

### Checkpoints de commit

- [ ] `feat: crud de puntos de encuentro del moderador` (T2.1–T2.3)
- [ ] `feat: modelo grupo y listado de grupos de hoy con populate` (T2.4–T2.6)
- [ ] `feat: crear grupo con validacion de punto y cupo` (T2.7)
- [ ] `feat: unirse y salir con operadores atomicos y traspaso de creador` (T2.8–T2.9)
- [ ] `feat: frontend caminemos juntos con mapa de puntos` (T2.10–T2.15)

---

## Investigación (equipos)

- [ ] **Equipo A — Atomicidad:** ¿qué es una condición de carrera (race condition)? Ejemplo del cupo: dos `unirse` simultáneos. ¿Por qué `$push` con filtro resuelve el caso y qué alternativas existen (transacciones)?
- [ ] **Equipo B — `populate` y privacidad:** ¿por qué proyectar `'nombre grupoEscolar'` en vez de traer el documento completo? ¿Qué es el principio de mínimo privilegio aplicado a datos personales (RN-14)?
- [ ] **Equipo C — Fechas:** `Date` vs String `'YYYY-MM-DD'`. ¿Qué es UTC, por qué `toISOString()` puede devolver "ayer" a las 7 pm en México y cuándo convendría migrar a `Date` con índices?
- [ ] **Equipo D — Borrado lógico vs físico:** ¿por qué `activo: false` en puntos de encuentro (RN-16) y borrado físico en grupos vacíos (RN-15)? Criterios para decidir cada uno.

## Criterio de cierre del hito

Dos alumnos pueden coordinar su salida: uno crea el grupo eligiendo un punto del moderador, el otro se une, el cupo se respeta, los integrantes se ven sin datos sensibles y los grupos de ayer no aparecen. Recién ahí: **Hito 3**.
