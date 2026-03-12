# 🍃 Semana 03 — MongoDB & Mongoose (NoSQL)

> **Mes 1 · Backend Foundations** | Stack: `MongoDB Atlas` · `Mongoose` · `Node.js` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Reemplazar el almacenamiento en memoria de las semanas anteriores por una **base de datos MongoDB** real en la nube (MongoDB Atlas). Aprender a modelar datos con **Mongoose**, manejar documentos, validaciones y relaciones en un sistema NoSQL.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| MongoDB Atlas | Cloud Free Tier | Base de datos NoSQL en la nube |
| Mongoose | ^7.x | ODM (Object Document Mapper) para MongoDB |
| dotenv | ^16.x | Gestión de la cadena de conexión |

---

## 📐 Arquitectura del Proyecto

```
semana-03-mongodb-nosql/
├── src/
│   ├── config/
│   │   └── db.ts                # Conexión a MongoDB con Mongoose
│   ├── models/
│   │   └── User.model.ts        # Schema y Model de Usuario
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   └── users.routes.ts
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   └── users.controller.ts
│   └── app.ts
├── .env
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación y conexión a MongoDB Atlas

```bash
npm install mongoose
```

```typescript
// src/config/db.ts
import mongoose from 'mongoose';

export const connectDB = async (): Promise<void> => {
  try {
    await mongoose.connect(process.env.MONGO_URI as string);
    console.log('✅ MongoDB conectado');
  } catch (error) {
    console.error('❌ Error de conexión:', error);
    process.exit(1);
  }
};
```

```env
# .env
MONGO_URI=mongodb+srv://<usuario>:<password>@cluster0.xxxxx.mongodb.net/semana03?retryWrites=true&w=majority
```

### 2. Definir el Schema y Model con Mongoose

```typescript
// src/models/User.model.ts
import { Schema, model, Document } from 'mongoose';

export interface IUser extends Document {
  name: string;
  email: string;
  password: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

const userSchema = new Schema<IUser>(
  {
    name: {
      type: String,
      required: [true, 'El nombre es requerido'],
      trim: true,
      minlength: 2,
    },
    email: {
      type: String,
      required: [true, 'El email es requerido'],
      unique: true,
      lowercase: true,
      match: [/\S+@\S+\.\S+/, 'Email inválido'],
    },
    password: {
      type: String,
      required: true,
      minlength: 6,
      select: false, // No incluir en queries por defecto
    },
    role: {
      type: String,
      enum: ['admin', 'user'],
      default: 'user',
    },
  },
  { timestamps: true }
);

export const User = model<IUser>('User', userSchema);
```

### 3. Controlador con operaciones CRUD reales

```typescript
// src/controllers/users.controller.ts
import { Request, Response } from 'express';
import { User } from '../models/User.model';

export const getAllUsers = async (req: Request, res: Response) => {
  const users = await User.find().select('-password');
  res.json(users);
};

export const getUserById = async (req: Request, res: Response) => {
  const user = await User.findById(req.params.id).select('-password');
  if (!user) return res.status(404).json({ message: 'Usuario no encontrado' });
  res.json(user);
};

export const updateUser = async (req: Request, res: Response) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true,       // Devuelve el documento actualizado
    runValidators: true,
  });
  if (!user) return res.status(404).json({ message: 'Usuario no encontrado' });
  res.json(user);
};

export const deleteUser = async (req: Request, res: Response) => {
  await User.findByIdAndDelete(req.params.id);
  res.status(204).send();
};
```

### 4. Iniciar conexión al arrancar la app

```typescript
// src/app.ts
import { connectDB } from './config/db';
// ...
connectDB(); // Llamar antes de levantar el servidor
```

---

## 🗄️ Conceptos de MongoDB Cubiertos

| Concepto | Descripción |
|---|---|
| **Documento** | Unidad de datos en MongoDB (similar a una fila, pero en JSON/BSON) |
| **Colección** | Conjunto de documentos (similar a una tabla) |
| **Schema** | Definición de la estructura de un documento en Mongoose |
| **ObjectId** | Identificador único auto-generado por MongoDB (`_id`) |
| **`timestamps: true`** | Añade `createdAt` y `updatedAt` automáticamente |
| **`select: false`** | Excluye campos sensibles de las consultas por defecto |

---

## 🧠 Conceptos Clave Aprendidos

- **NoSQL vs SQL**: MongoDB almacena datos como documentos BSON flexibles, ideal para datos jerárquicos o con estructura variable.
- **ODM (Mongoose)**: Proporciona tipado, validaciones, hooks y métodos de instancia sobre los datos de MongoDB.
- **Validaciones a nivel de Schema**: `required`, `minlength`, `match`, `enum` protegen la integridad de los datos.
- **Operaciones `async/await`**: Todas las operaciones de Mongoose son asíncronas y devuelven Promises.

---

## ✅ Entregables

- [ ] Cluster de MongoDB Atlas configurado y conectado.
- [ ] Model de `User` con Schema completo y validaciones.
- [ ] CRUD funcionando con datos persistentes en MongoDB.
- [ ] Contraseñas excluidas de las respuestas usando `.select('-password')`.
- [ ] `MONGO_URI` almacenada en `.env` y nunca en el código.

---

## 📖 Recursos de Referencia

- [MongoDB Atlas — Getting Started](https://www.mongodb.com/docs/atlas/getting-started/)
- [Mongoose Docs — Schemas](https://mongoosejs.com/docs/guide.html)
- [MongoDB University (Cursos gratuitos)](https://learn.mongodb.com/)

---

*← [Semana 02](../1-mes/semana-02-auth-security.md) | [Volver al Roadmap](../../README.md) | [Semana 04 →](../1-mes/semana-04-postgresql-orm.md)*
