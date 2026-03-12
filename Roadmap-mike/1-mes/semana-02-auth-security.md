# 🔐 Semana 02 — Autenticación & Seguridad (JWT + bcrypt)

> **Mes 1 · Backend Foundations** | Stack: `Node.js` · `Express` · `JWT` · `bcrypt` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Añadir una **capa de seguridad completa** a la API REST de la semana anterior. Implementar el flujo de autenticación con registro, login y protección de rutas mediante **JSON Web Tokens (JWT)** y hashing seguro de contraseñas con **bcrypt**.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| jsonwebtoken | ^9.x | Generación y verificación de JWTs |
| bcrypt | ^5.x | Hashing seguro de contraseñas |
| dotenv | ^16.x | Gestión de variables de entorno |
| @types/jsonwebtoken | — | Tipos TypeScript para JWT |
| @types/bcrypt | — | Tipos TypeScript para bcrypt |

---

## 📐 Arquitectura del Proyecto

```
semana-02-auth-security/
├── src/
│   ├── routes/
│   │   ├── auth.routes.ts       # /register y /login
│   │   └── users.routes.ts      # Rutas protegidas (semana 1)
│   ├── middlewares/
│   │   └── auth.middleware.ts   # Verificación del JWT
│   ├── controllers/
│   │   └── auth.controller.ts   # Lógica de registro y login
│   ├── services/
│   │   └── auth.service.ts      # Hashing, generación de tokens
│   └── app.ts
├── .env
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación de dependencias

```bash
npm install jsonwebtoken bcrypt dotenv
npm install -D @types/jsonwebtoken @types/bcrypt
```

### 2. Variables de entorno (`.env`)

```env
JWT_SECRET=tu_clave_secreta_super_segura_aqui
JWT_EXPIRES_IN=1h
PORT=3000
```

> ⚠️ **Nunca subas el archivo `.env` a GitHub.** Añádelo a tu `.gitignore`.

### 3. Servicio de Autenticación

```typescript
// src/services/auth.service.ts
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

const SALT_ROUNDS = 10;

export const hashPassword = async (password: string): Promise<string> => {
  return bcrypt.hash(password, SALT_ROUNDS);
};

export const comparePassword = async (
  plain: string,
  hashed: string
): Promise<boolean> => {
  return bcrypt.compare(plain, hashed);
};

export const generateToken = (userId: string): string => {
  return jwt.sign({ id: userId }, process.env.JWT_SECRET as string, {
    expiresIn: process.env.JWT_EXPIRES_IN || '1h',
  });
};
```

### 4. Controlador de Autenticación

```typescript
// src/controllers/auth.controller.ts
import { Request, Response } from 'express';
import { hashPassword, comparePassword, generateToken } from '../services/auth.service';

// Base en memoria para el ejemplo
const users: { id: string; email: string; password: string }[] = [];

export const register = async (req: Request, res: Response) => {
  const { email, password } = req.body;
  const hashed = await hashPassword(password);
  const newUser = { id: Date.now().toString(), email, password: hashed };
  users.push(newUser);
  res.status(201).json({ message: 'Usuario registrado', id: newUser.id });
};

export const login = async (req: Request, res: Response) => {
  const { email, password } = req.body;
  const user = users.find((u) => u.email === email);
  if (!user) return res.status(401).json({ message: 'Credenciales inválidas' });

  const valid = await comparePassword(password, user.password);
  if (!valid) return res.status(401).json({ message: 'Credenciales inválidas' });

  const token = generateToken(user.id);
  res.json({ token });
};
```

### 5. Middleware de Autenticación

```typescript
// src/middlewares/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export interface AuthRequest extends Request {
  userId?: string;
}

export const authMiddleware = (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'Token no proporcionado' });
  }

  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET as string) as { id: string };
    req.userId = decoded.id;
    next();
  } catch {
    res.status(401).json({ message: 'Token inválido o expirado' });
  }
};
```

### 6. Proteger rutas existentes

```typescript
// src/routes/users.routes.ts
import { Router } from 'express';
import { authMiddleware } from '../middlewares/auth.middleware';
import { getAllUsers, getUserById, createUser, updateUser, deleteUser } from '../controllers/users.controller';

export const usersRouter = Router();

// Todas las rutas de usuarios requieren autenticación
usersRouter.use(authMiddleware);

usersRouter.get('/', getAllUsers);
usersRouter.get('/:id', getUserById);
usersRouter.post('/', createUser);
usersRouter.put('/:id', updateUser);
usersRouter.delete('/:id', deleteUser);
```

---

## 🔌 Endpoints de la API

| Método | URL | Descripción | Auth requerida |
|---|---|---|---|
| `POST` | `/api/auth/register` | Registrar nuevo usuario | ❌ |
| `POST` | `/api/auth/login` | Iniciar sesión, devuelve JWT | ❌ |
| `GET` | `/api/users` | Listar usuarios | ✅ Bearer Token |
| `GET` | `/api/users/:id` | Ver usuario específico | ✅ Bearer Token |
| `PUT` | `/api/users/:id` | Actualizar usuario | ✅ Bearer Token |
| `DELETE` | `/api/users/:id` | Eliminar usuario | ✅ Bearer Token |

---

## 🧠 Conceptos Clave Aprendidos

- **Hashing vs Encriptación**: bcrypt usa un hash unidireccional con salt, imposible de revertir. Nunca se almacenan contraseñas en texto plano.
- **JWT (JSON Web Token)**: Token auto-contenido con 3 partes (`header.payload.signature`). El servidor lo firma con un secreto y el cliente lo envía en cada request.
- **Variables de Entorno**: Separar configuración sensible (`JWT_SECRET`) del código fuente.
- **Middleware de Guards**: Patrón común para proteger grupos de rutas con `router.use(authMiddleware)`.

---

## ✅ Entregables

- [ ] Flujo de `register` y `login` funcionando con bcrypt.
- [ ] Middleware `authMiddleware` verifica correctamente el JWT.
- [ ] Rutas CRUD de Usuarios protegidas — devuelven `401` sin token válido.
- [ ] Pruebas en Postman: registro → login → uso del token en rutas protegidas.
- [ ] Archivo `.env.example` en el repositorio (sin datos sensibles).

---

## 📖 Recursos de Referencia

- [jwt.io — Debugger y documentación](https://jwt.io/)
- [bcrypt npm package](https://www.npmjs.com/package/bcrypt)
- [OWASP: Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

---

*← [Semana 01](../1-mes/semana-01-node-express.md) | [Volver al Roadmap](../../README.md) | [Semana 03 →](../1-mes/semana-03-mongodb-nosql.md)*
