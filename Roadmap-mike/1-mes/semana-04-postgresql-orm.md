# 🐘 Semana 04 — PostgreSQL & Prisma ORM

> **Mes 1 · Backend Foundations** | Stack: `PostgreSQL` · `Prisma ORM` · `Node.js` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Configurar una base de datos **relacional PostgreSQL** y conectarla a la API usando **Prisma ORM**. Diseñar un esquema con relaciones entre tablas (`@relation`), ejecutar **migraciones** y realizar operaciones CRUD tipadas con el cliente de Prisma.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| PostgreSQL | 15+ | Base de datos relacional |
| Prisma ORM | ^5.x | ORM tipado, migraciones y Prisma Studio |
| @prisma/client | ^5.x | Cliente auto-generado para queries |
| Docker (opcional) | — | Levantar PostgreSQL localmente |

---

## 📐 Arquitectura del Proyecto

```
semana-04-postgresql-orm/
├── prisma/
│   ├── schema.prisma            # Definición del modelo de datos
│   └── migrations/              # Historial de migraciones (auto-generado)
├── src/
│   ├── lib/
│   │   └── prisma.ts            # Singleton del PrismaClient
│   ├── routes/
│   │   └── posts.routes.ts
│   ├── controllers/
│   │   └── posts.controller.ts
│   └── app.ts
├── .env
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Levantar PostgreSQL (con Docker)

```bash
docker run --name postgres-semana04 \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=roadmap_db \
  -p 5432:5432 \
  -d postgres:15
```

### 2. Instalación y configuración de Prisma

```bash
npm install @prisma/client
npm install -D prisma

npx prisma init  # Crea prisma/schema.prisma y .env
```

```env
# .env
DATABASE_URL="postgresql://postgres:password@localhost:5432/roadmap_db?schema=public"
```

### 3. Definir el Schema de Prisma (`schema.prisma`)

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  posts     Post[]   // Relación 1:N con Posts
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
}
```

### 4. Ejecutar la migración

```bash
# Crea y aplica la migración en la base de datos
npx prisma migrate dev --name init

# Visualizar la base de datos en el navegador
npx prisma studio
```

### 5. Singleton del PrismaClient

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ['query', 'info', 'warn', 'error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

### 6. Controlador con Prisma Client

```typescript
// src/controllers/posts.controller.ts
import { Request, Response } from 'express';
import { prisma } from '../lib/prisma';

export const getAllPosts = async (req: Request, res: Response) => {
  const posts = await prisma.post.findMany({
    include: { author: { select: { name: true, email: true } } },
  });
  res.json(posts);
};

export const createPost = async (req: Request, res: Response) => {
  const { title, content, authorId } = req.body;
  const post = await prisma.post.create({
    data: { title, content, authorId: Number(authorId) },
  });
  res.status(201).json(post);
};

export const updatePost = async (req: Request, res: Response) => {
  const post = await prisma.post.update({
    where: { id: Number(req.params.id) },
    data: req.body,
  });
  res.json(post);
};

export const deletePost = async (req: Request, res: Response) => {
  await prisma.post.delete({ where: { id: Number(req.params.id) } });
  res.status(204).send();
};
```

---

## 🗄️ Prisma vs Mongoose: Comparación Clave

| Aspecto | Mongoose (MongoDB) | Prisma (PostgreSQL) |
|---|---|---|
| Tipo de DB | NoSQL / Documentos | SQL / Relacional |
| Schema | Definido en código JS/TS | Declarativo en `schema.prisma` |
| Relaciones | Referencias manuales | `@relation` nativa y tipada |
| Migraciones | Manual / no nativo | `prisma migrate dev` automático |
| Tipado | Parcial con TypeScript | 100% auto-generado y tipado |

---

## 🧠 Conceptos Clave Aprendidos

- **Prisma Schema**: DSL declarativo para modelar tablas, relaciones y restricciones.
- **Relaciones SQL**: `@relation` implementa Foreign Keys (`authorId → User.id`).
- **Migraciones**: Historial controlado de cambios en el esquema de la BD.
- **PrismaClient Singleton**: Evita crear múltiples conexiones al pool en desarrollo.
- **`include` vs `select`**: Controlar los datos relacionados que se traen en cada query.

---

## ✅ Entregables

- [ ] Schema de Prisma con al menos 2 modelos relacionados (`User` y `Post`).
- [ ] Migración aplicada correctamente con `prisma migrate dev`.
- [ ] CRUD completo para `Post` usando el PrismaClient.
- [ ] Prisma Studio ejecutado para visualizar los datos.
- [ ] `DATABASE_URL` en `.env` (nunca en el código fuente).

---

## 📖 Recursos de Referencia

- [Prisma ORM Docs — Getting Started](https://www.prisma.io/docs/getting-started)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Prisma Studio](https://www.prisma.io/studio)

---

*← [Semana 03](../1-mes/semana-03-mongodb-nosql.md) | [Volver al Roadmap](../../README.md) | [Semana 05 →](../2-mes/semana-05-react-foundations.md)*
