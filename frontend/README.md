# Frontend

Interfaz web responsiva con HTML5 + CSS3 + JavaScript (módulos ES), Leaflet para mapas y la Geolocation API. Se sirve en desarrollo con la extensión **Live Server** de VS Code.

Esta carpeta se construye paso a paso desde el [Hito 0](../docs/hito-0-configuracion-y-modulo-base.md). No hay código todavía a propósito: la guía docente lleva el orden exacto de tareas y checkpoints de commit.

Estructura final esperada:

```text
frontend/
├── index.html        # login
├── registro.html
├── app.html          # home alumno: botón de alerta + mapa + grupos
├── moderador.html    # panel: alertas del día, puntos, configuración
├── css/
└── js/
    ├── api.js        # helper fetch con token (sessionStorage) y errores RN-06
    └── ...           # un módulo por pantalla/funcionalidad
```

El token JWT se guarda en `sessionStorage` (RN-05) y se envía como `Authorization: Bearer <token>`.
