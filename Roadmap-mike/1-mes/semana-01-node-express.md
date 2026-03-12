# 🚀 Semana 01 — API REST con Node.js & Express

> **Mes 1 · Backend Foundations** | Stack: `Node.js` · `Express` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Construir un servidor backend robusto desde cero usando **Node.js**, **Express** y **TypeScript**. Diseñar e implementar una **API RESTful** completa para el recurso `Usuarios`, aprendiendo el ciclo de vida de las peticiones HTTP, el sistema de enrutamiento y el patrón de middlewares.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| Node.js | 20 LTS | Runtime de JavaScript del lado del servidor |
| Express.js | ^4.18 | Framework HTTP minimalista y flexible |
| TypeScript | ^5.x | Tipado estático para un código más seguro |
| ts-node-dev | ^2.x | Hot-reload en desarrollo para TypeScript |
| Postman / Thunder Client | — | Testing de endpoints HTTP |

---

## 📐 Arquitectura del Proyecto

```
semana-01-node-express/
├── src/
│   ├── routes/
│   │   └── users.routes.ts      # Router del recurso Usuarios
│   ├── middlewares/
│   │   ├── logger.middleware.ts  # Loguea método, URL y tiempo
│   │   └── error.middleware.ts   # Manejador global de errores
│   ├── controllers/
│   │   └── users.controller.ts  # Lógica de negocio de Usuarios
│   ├── models/
│   │   └── user.model.ts        # Interfaz/Tipo User (en memoria)
│   └── app.ts                   # Configuración central de Express
├── tsconfig.json
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Inicialización y Configuración

```bash
# Inicializar proyecto Node
npm init -y

# Instalar dependencias de producción
npm install express

# Instalar dependencias de desarrollo
npm install -D typescript ts-node-dev @types/node @types/express

# Inicializar configuración de TypeScript
npx tsc --init
```

Configura en `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### 2. Servidor Express

```typescript
// src/app.ts
import express, { Application } from 'express';
import { usersRouter } from './routes/users.routes';
import { loggerMiddleware } from './middlewares/logger.middleware';
import { errorMiddleware } from './middlewares/error.middleware';

const app: Application = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());
app.use(loggerMiddleware);

app.use('/api/users', usersRouter);

app.use(errorMiddleware);

app.listen(PORT, () => {
  console.log(`✅ Server running on http://localhost:${PORT}`);
});
```

### 3. Rutas CRUD

```typescript
// src/routes/users.routes.ts
import { Router } from 'express';
import {
  getAllUsers,
  getUserById,
  createUser,
  updateUser,
  deleteUser,
} from '../controllers/users.controller';

export const usersRouter = Router();

usersRouter.get('/', getAllUsers);
usersRouter.get('/:id', getUserById);
usersRouter.post('/', createUser);
usersRouter.put('/:id', updateUser);
usersRouter.delete('/:id', deleteUser);
```

### 4. Middlewares

```typescript
// src/middlewares/logger.middleware.ts
import { Request, Response, NextFunction } from 'express';

export const loggerMiddleware = (req: Request, res: Response, next: NextFunction) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
};

// src/middlewares/error.middleware.ts
import { Request, Response, NextFunction } from 'express';

export const errorMiddleware = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  console.error(err.stack);
  res.status(500).json({ message: err.message || 'Internal Server Error' });
};
```

### 5. Scripts en `package.json`

```json
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js"
  }
}
```

---

## 🔌 Endpoints de la API

| Método | URL | Descripción | Body esperado |
|---|---|---|---|
| `GET` | `/api/users` | Obtener todos los usuarios | — |
| `GET` | `/api/users/:id` | Obtener usuario por ID | — |
| `POST` | `/api/users` | Crear un nuevo usuario | `{ name, email }` |
| `PUT` | `/api/users/:id` | Actualizar un usuario | `{ name?, email? }` |
| `DELETE` | `/api/users/:id` | Eliminar un usuario | — |

---

## 🧠 Conceptos Clave Aprendidos

- **Event Loop**: Node.js maneja las I/O de manera no bloqueante, lo que lo hace ideal para APIs de alto rendimiento.
- **Middleware Pattern**: Express usa una cadena de funciones `(req, res, next)` para procesar requests de forma modular.
- **REST Architecture**: Uso correcto de verbos HTTP (`GET`, `POST`, `PUT`, `DELETE`) y códigos de estado (`200`, `201`, `404`, `500`).
- **Tipado con TypeScript**: Interfaces para tipar los objetos `Request`, `Response` y modelos de datos.

---

## ✅ Entregables

- [ ] Repositorio en GitHub con el código fuente y un `README.md` propio.
- [ ] Colección de Postman exportada (`.json`) probando todos los endpoints.
- [ ] Configuración correcta de `scripts` en `package.json` para modo desarrollo y producción.
- [ ] Al menos 2 middlewares funcionales: `loggerMiddleware` y `errorMiddleware`.

---

## 📖 Recursos de Referencia

- [Documentación oficial de Express.js](https://expressjs.com/)
- [TypeScript Deep Dive (Libro gratuito)](https://basarat.gitbook.io/typescript/)
- [Conceptos de REST API Design](https://restfulapi.net/)

---

*← [Volver al Roadmap principal](../../README.md) | Siguiente: [Semana 02 — Auth & Seguridad](../1-mes/semana-02-auth-security.md) →*
