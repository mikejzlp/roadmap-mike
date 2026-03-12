# 🏛️ Semana 13 — Clean Architecture & Principios SOLID

> **Mes 3 · Full Stack Integration** | Stack: `Node.js` · `TypeScript` · `Express` · `Principios SOLID`

---

## 🎯 Objetivo del Proyecto

Refactorizar una API existente aplicando los principios de **Clean Architecture** (Arquitectura Limpia) de Robert C. Martin. El objetivo es crear código desacoplado, testeable y mantenible separando las **capas de dominio, aplicación e infraestructura**.

---

## 🛠️ Conceptos Fundamentales

### Los 5 Principios SOLID

| Principio | Descripción | Ejemplo en Node.js |
|---|---|---|
| **S** — Single Responsibility | Una clase, una razón para cambiar | `UserRepository` solo gestiona DB, `AuthService` solo gestiona tokens |
| **O** — Open/Closed | Abierto para extensión, cerrado para modificación | Añadir nuevo `PaymentProvider` sin tocar el código existente |
| **L** — Liskov Substitution | Las subclases deben ser sustituibles por su clase base | `MongoUserRepo` y `PrismaUserRepo` implementan la misma interfaz |
| **I** — Interface Segregation | Interfaces pequeñas y específicas | `IReadableRepo` e `IWritableRepo` separadas |
| **D** — Dependency Inversion | Depender de abstracciones, no de implementaciones | `UserService` recibe `IUserRepository`, no `MongoUserRepository` |

---

## 📐 Arquitectura del Proyecto

```
semana-13-clean-architecture/
├── src/
│   ├── domain/                    # 🏛️ CAPA DE DOMINIO (pura, sin dependencias externas)
│   │   ├── entities/
│   │   │   └── User.ts            # Clase de dominio con lógica de negocio
│   │   └── repositories/
│   │       └── IUserRepository.ts # Interfaz (contrato) del repositorio
│   │
│   ├── application/               # ⚙️ CAPA DE APLICACIÓN (casos de uso)
│   │   └── use-cases/
│   │       ├── CreateUser.ts
│   │       ├── GetUserById.ts
│   │       └── DeleteUser.ts
│   │
│   ├── infrastructure/            # 🔌 CAPA DE INFRAESTRUCTURA (detalles externos)
│   │   ├── repositories/
│   │   │   ├── MongoUserRepository.ts  # Implementación con MongoDB
│   │   │   └── PrismaUserRepository.ts # Implementación con Prisma
│   │   └── http/
│   │       ├── controllers/
│   │       │   └── UserController.ts
│   │       └── routes/
│   │           └── users.routes.ts
│   │
│   └── app.ts
```

---

## 📋 Implementación Paso a Paso

### 1. Entidad de Dominio

```typescript
// src/domain/entities/User.ts
export class User {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly email: string,
    private readonly password: string
  ) {
    this.validate();
  }

  private validate(): void {
    if (!this.name || this.name.length < 2) {
      throw new Error('El nombre debe tener al menos 2 caracteres');
    }
    if (!this.email.includes('@')) {
      throw new Error('Email inválido');
    }
  }

  // La lógica de negocio vive en la entidad
  toPublicDTO() {
    return { id: this.id, name: this.name, email: this.email };
  }
}
```

### 2. Interfaz del Repositorio (contrato)

```typescript
// src/domain/repositories/IUserRepository.ts
import { User } from '../entities/User';

export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}
```

### 3. Casos de Uso (Application Layer)

```typescript
// src/application/use-cases/CreateUser.ts
import { IUserRepository } from '../../domain/repositories/IUserRepository';
import { User } from '../../domain/entities/User';
import { v4 as uuidv4 } from 'uuid';

interface CreateUserDTO {
  name: string;
  email: string;
  password: string;
}

export class CreateUser {
  constructor(private readonly userRepository: IUserRepository) {} // DI

  async execute(dto: CreateUserDTO): Promise<User> {
    const existing = await this.userRepository.findByEmail(dto.email);
    if (existing) throw new Error('El email ya está en uso');

    const user = new User(uuidv4(), dto.name, dto.email, dto.password);
    return this.userRepository.save(user);
  }
}
```

### 4. Implementación del Repositorio (Infrastructure Layer)

```typescript
// src/infrastructure/repositories/MongoUserRepository.ts
import { IUserRepository } from '../../domain/repositories/IUserRepository';
import { User } from '../../domain/entities/User';
import { UserModel } from '../database/models/UserModel';

export class MongoUserRepository implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    const doc = await UserModel.findById(id);
    if (!doc) return null;
    return new User(doc._id.toString(), doc.name, doc.email, doc.password);
  }

  async save(user: User): Promise<User> {
    const created = await UserModel.create({
      name: user.name,
      email: user.email,
    });
    return new User(created._id.toString(), created.name, created.email, '');
  }

  async findAll(): Promise<User[]> {
    const docs = await UserModel.find();
    return docs.map((d) => new User(d._id.toString(), d.name, d.email, ''));
  }

  async findByEmail(email: string): Promise<User | null> {
    const doc = await UserModel.findOne({ email });
    if (!doc) return null;
    return new User(doc._id.toString(), doc.name, doc.email, doc.password);
  }

  async delete(id: string): Promise<void> {
    await UserModel.findByIdAndDelete(id);
  }
}
```

### 5. Controlador HTTP (Infrastructure Layer)

```typescript
// src/infrastructure/http/controllers/UserController.ts
import { Request, Response } from 'express';
import { CreateUser } from '../../../application/use-cases/CreateUser';
import { MongoUserRepository } from '../../repositories/MongoUserRepository';

// Composición de dependencias (podría usar un IoC container)
const userRepository = new MongoUserRepository();
const createUser = new CreateUser(userRepository);

export class UserController {
  async create(req: Request, res: Response) {
    try {
      const user = await createUser.execute(req.body);
      res.status(201).json(user.toPublicDTO());
    } catch (error) {
      res.status(400).json({ message: (error as Error).message });
    }
  }
}
```

---

## 🧠 Beneficios de la Arquitectura Limpia

| Aspecto | Sin Clean Architecture | Con Clean Architecture |
|---|---|---|
| **Testing** | Difícil (acoplado a DB/HTTP) | Fácil (inyección de dependencias) |
| **Cambiar DB** | Modifica toda la app | Solo cambia el repositorio |
| **Reglas de negocio** | Distribuidas en rutas/controllers | Concentradas en Entidades y Casos de Uso |
| **Onboarding** | Confuso, spaghetti code | Clara separación de responsabilidades |

---

## ✅ Entregables

- [ ] Estructura de carpetas con separación `domain/`, `application/`, `infrastructure/`.
- [ ] Al menos 3 casos de uso implementados (Create, GetById, Delete).
- [ ] Repositorio MongoDB implementando `IUserRepository`.
- [ ] Tests unitarios de los casos de uso con el repositorio mockeado.

---

*← [Semana 12](../3-mes/semana-12-testing-jest-cypress.md) | [Volver al Roadmap](../../README.md) | [Semana 14 →](../4-mes/semana-14-rn-basics.md)*
