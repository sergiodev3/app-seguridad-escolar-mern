# Hito 1 — Botón de Alerta (Módulo 1)

> **Fuente de verdad:** [spec.md](../spec.md) — modelo `alertas` (§5), RN-08, endpoints de alertas (§7) y sección 8 (Límites del navegador).
> **Depende de:** Hito 0 completo (auth, middlewares, `api.js`, seed).

## Objetivos de aprendizaje

- Registrar alertas con geolocalización del navegador y mostrarlas en un mapa (Leaflet + OpenStreetMap).
- Implementar confirmación por *long-press* de 2 segundos con `pointerdown`/`pointerup`/`setTimeout` (RN-08).
- Entender los límites reales de una web: enlace `tel:` para llamar y `wa.me` para WhatsApp (spec §8).
- Construir el panel del moderador: alertas del día y marcarlas como atendidas.

## Prerequisitos

- Hito 0 cerrado (todos sus checkboxes en `[x]`).
- Leaflet se usa por CDN (no hay bundler en el frontend):

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

---

## Tasks

### Backend

- [ ] **T1.1** `src/models/Alerta.js` según spec §5: `usuarioId` (ref `Usuario`), `lat`, `lng` (opcionales), `atendida` (default `false`), `creadaEn`.
- [ ] **T1.2** `POST /api/alertas` (Alumno): crea la alerta con el `usuarioId` del token (nunca del body) y `lat`/`lng` opcionales. Si el usuario negó la geolocalización, la alerta se registra igual, sin ubicación.
- [ ] **T1.3** `GET /api/alertas` (Moderador): solo alertas **del día** (de medianoche a ahora), con `populate` del nombre y matrícula del usuario.
- [ ] **T1.4** `PATCH /api/alertas/:id` (Moderador): marca `atendida: true`. Validar que el `:id` exista (400 si no).

### Frontend — mapa y ubicación

- [ ] **T1.5** En `app.html`: inicializar mapa Leaflet centrado en el plantel (definir la constante con las coordenadas de la escuela) y obtener posición actual con `navigator.geolocation.getCurrentPosition`.

> **Recordar (spec §8):** Geolocation solo funciona en HTTPS o `localhost`/`127.0.0.1`. Con Live Server (`http://127.0.0.1:5500`) funciona. Manejar los tres casos: permiso concedido, permiso denegado y timeout.

- [ ] **T1.6** Mostrar la posición del alumno con un marcador y un círculo de precisión (`L.circle` con `coords.accuracy`).

### Frontend — botón de alerta (RN-08)

- [ ] **T1.7** Botón grande "ALERTA" con *long-press* de 2 s: arrancar `setTimeout` en `pointerdown`, cancelarlo en `pointerup`/`pointercancel`, y dar feedback visual del progreso (borde que se llena, texto "Mantén presionado…", vibración `navigator.vibrate` si existe).
- [ ] **T1.8** Al confirmarse: `POST /api/alertas` con la última posición conocida y mostrar confirmación en pantalla.
- [ ] **T1.9** Botón "Llamar a la policía": obtiene el número con `GET /api/config/policia` y renderiza un enlace `<a href="tel:...">` (la web **no** puede marcar sola — spec §8).
- [ ] **T1.10** Botón "Avisar a mi contacto": lee los contactos del usuario y abre `https://wa.me/52<telefono>?text=<mensaje>` con mensaje prearmado (incluye ubicación si hay). 100% frontend, el backend no interviene.

### Frontend — panel del moderador

- [ ] **T1.11** En `moderador.html`: lista de alertas del día (hora, alumno, ubicación, estado) con botón "Marcar atendida" (`PATCH`) y refrescar.
- [ ] **T1.12** Mostrar cada alerta en un mini-mapa Leaflet (o un enlace a OpenStreetMap con las coordenadas).

### Verificación (Thunder Client + navegador)

- [ ] **T1.13** Casos de API:

| Caso | Esperado |
| :-- | :-- |
| `POST /alertas` con token de alumno y coords | 201/200 + alerta creada |
| `POST /alertas` sin token | 401 |
| `GET /alertas` con token de alumno | 403 |
| `GET /alertas` moderador | 200 + solo alertas de hoy |
| `PATCH /alertas/:id` moderador con id inexistente | 400 |
| `PATCH /alertas/:id` moderador | 200 + `atendida: true` |

- [ ] **T1.14** Prueba de punta a punta en el navegador: alumno mantiene presionado 2 s → alerta creada → aparece en el panel del moderador → la marca atendida. Probar también el flujo con geolocalización **denegada**.

### Checkpoints de commit

- [ ] `feat: modelo y endpoints de alertas con filtro del dia` (T1.1–T1.4)
- [ ] `feat: mapa leaflet con geolocalizacion del alumno` (T1.5–T1.6)
- [ ] `feat: boton de alerta con long-press de 2 segundos` (T1.7–T1.8)
- [ ] `feat: llamada tel y aviso whatsapp al contacto` (T1.9–T1.10)
- [ ] `feat: panel del moderador para alertas del dia` (T1.11–T1.14)

---

## Investigación (equipos)

- [ ] **Equipo A — Permisos del navegador:** ¿qué es un "secure context"? ¿Por qué Geolocation se niega en HTTP plano y qué pasaría al desplegar la app?
- [ ] **Equipo B — Eventos pointer:** diferencias entre `click`, `touchstart` y `pointerdown`. ¿Por qué `pointer*` unifica mouse/touch/lápiz? ¿Qué bug aparece si no cancelamos en `pointercancel`?
- [ ] **Equipo C — `tel:` y `wa.me`:** ¿cómo funcionan los URI schemes? ¿Qué puede y qué no puede hacer una web con el teléfono del usuario?
- [ ] **Equipo D — Filtros por fecha en Mongo:** ¿cómo expresar "alertas de hoy" con `$gte`/`$lt` sobre `Date`? Comparar con el enfoque de `fecha` String del Hito 2.

## Criterio de cierre del hito

Un alumno puede disparar una alerta real (con y sin ubicación), llamar a la policía en dos toques y avisar a su contacto por WhatsApp; el moderador ve las alertas del día y las atiende. Recién ahí: **Hito 2**.
