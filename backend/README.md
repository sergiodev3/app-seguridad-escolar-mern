# Backend

API REST con Node.js + Express + MongoDB (Mongoose).

Esta carpeta se construye paso a paso en el [Hito 0](../docs/hito-0-configuracion-y-modulo-base.md). No hay código todavía a propósito: la guía docente lleva el orden exacto de tareas, comandos y checkpoints de commit.

Estructura final esperada (definida en el Hito 0):

```text
backend/
├── src/
│   ├── config/       # conexión a MongoDB
│   ├── models/       # esquemas de Mongoose
│   ├── controllers/  # lógica de cada endpoint
│   ├── middlewares/  # auth (JWT), soloRol, errores
│   ├── routes/       # rutas Express + express-validator
│   └── seed/         # datos iniciales (moderador, configuraciones, puntos)
├── .env.example
└── package.json
```
