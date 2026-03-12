# 🏆 Semana 25 — Capstone: ERP Corporativo

> **Mes 6 · Cloud & Arquitectura Avanzada** | Stack: `TypeScript` · `React` · `Node.js` · `PostgreSQL` · `Docker` · `AWS` · `K8s`

---

## 🎯 Objetivo del Proyecto

**Proyecto cumbre del roadmap.** Construir un **sistema ERP (Enterprise Resource Planning) corporativo** completo aplicando TODAS las tecnologías aprendidas en los 6 meses. Este proyecto debe ser desplegable en producción, testeable y demostrable como pieza central del portafolio.

---

## 🛠️ Stack Tecnológico Completo

| Capa | Tecnología |
|---|---|
| **Frontend** | React 18 + TypeScript + Tailwind CSS + Zustand + React Query |
| **Backend** | Node.js + Express + TypeScript + Clean Architecture |
| **Base de Datos** | PostgreSQL + Prisma ORM |
| **Autenticación** | JWT + Refresh Tokens + Roles (ADMIN, MANAGER, EMPLOYEE) |
| **Real-time** | Socket.io (notificaciones y actualizaciones en vivo) |
| **Testing** | Jest + Supertest + Cypress (E2E) |
| **CI/CD** | GitHub Actions (build → test → deploy) |
| **Infra** | Docker + Docker Compose (dev), AWS ECS / K8s (prod) |

---

## 📐 Módulos del ERP

```
ERP Corporativo
├── 🔐 Módulo Auth & RBAC
│   ├── Login / Logout / Refresh Token
│   ├── Roles: ADMIN > MANAGER > EMPLOYEE
│   └── Middleware de autorización por rol
│
├── 👥 Módulo RRHH (Recursos Humanos)
│   ├── Gestión de empleados (CRUD)
│   ├── Departamentos y posiciones
│   └── Registro de asistencia
│
├── 📦 Módulo Inventario
│   ├── Catálogo de productos/SKUs
│   ├── Control de stock + alertas
│   └── Historial de movimientos
│
├── 💰 Módulo Financiero
│   ├── Cuentas por cobrar / pagar
│   ├── Registro de transacciones
│   └── Dashboard de P&L (Profit & Loss)
│
└── 📊 Módulo Reportes
    ├── Charts con Recharts (ventas, stock, RRHH)
    ├── Export a PDF / Excel
    └── Filtros por fecha, departamento, etc.
```

---

## 📋 Estructura Técnica del Proyecto

### Schema de Prisma (Simplificado)

```prisma
// prisma/schema.prisma
model User {
  id         Int        @id @default(autoincrement())
  email      String     @unique
  name       String
  role       Role       @default(EMPLOYEE)
  department Department @relation(fields: [deptId], references: [id])
  deptId     Int
  attendances Attendance[]
}

enum Role { ADMIN MANAGER EMPLOYEE }

model Department {
  id       Int    @id @default(autoincrement())
  name     String
  manager  User?  @relation("DepartmentManager")
  users    User[]
}

model Product {
  id            Int             @id @default(autoincrement())
  sku           String          @unique
  name          String
  stock         Int             @default(0)
  minStock      Int             @default(10)   // Alerta si stock < minStock
  price         Decimal         @db.Decimal(10,2)
  movements     StockMovement[]
}

model StockMovement {
  id        Int            @id @default(autoincrement())
  product   Product        @relation(fields: [productId], references: [id])
  productId Int
  type      MovementType
  quantity  Int
  reason    String?
  createdAt DateTime       @default(now())
}

enum MovementType { PURCHASE SALE ADJUSTMENT RETURN }

model Transaction {
  id          Int             @id @default(autoincrement())
  type        TransactionType
  amount      Decimal         @db.Decimal(10,2)
  description String
  date        DateTime        @default(now())
}

enum TransactionType { INCOME EXPENSE }
```

### Architecture (Clean Architecture aplicada)

```
src/
├── domain/
│   ├── entities/          # User, Product, Transaction, etc.
│   └── repositories/      # Interfaces (contratos)
├── application/
│   └── use-cases/         # CreateUser, RegisterSale, UpdateStock, etc.
├── infrastructure/
│   ├── repositories/      # Implementaciones con Prisma
│   ├── http/
│   │   ├── controllers/
│   │   ├── middlewares/   # auth, rbac, rateLimit, requestId
│   │   └── routes/
│   └── sockets/           # Notificaciones en tiempo real
└── shared/
    ├── errors/            # DomainError, ValidationError, etc.
    └── utils/             # Logger, date helpers, etc.
```

### Sistema de Notificaciones en Tiempo Real

```typescript
// src/infrastructure/sockets/notifications.socket.ts
import { Server } from 'socket.io';

export const setupNotifications = (io: Server) => {
  io.on('connection', (socket) => {
    const { userId, role } = socket.handshake.auth;
    socket.join(`user:${userId}`);
    socket.join(`role:${role}`);
  });
};

// Emitir alerta de stock bajo a todos los managers/admins
export const notifyLowStock = (io: Server, product: { name: string; stock: number }) => {
  io.to('role:ADMIN').to('role:MANAGER').emit('alert:low_stock', {
    message: `⚠️ Stock bajo: ${product.name} (${product.stock} unidades)`,
    severity: 'warning',
    timestamp: new Date(),
  });
};
```

---

## 📊 Dashboard del ERP (Frontend)

```typescript
// src/pages/Dashboard.tsx — Composición del dashboard principal
import { StatsGrid } from '../components/StatsGrid';
import { SalesLineChart } from '../components/charts/SalesLineChart';
import { StockAlerts } from '../components/StockAlerts';
import { RecentTransactions } from '../components/RecentTransactions';
import { useDashboardStats } from '../hooks/useDashboardStats';

export const Dashboard = () => {
  const { data: stats } = useDashboardStats();

  return (
    <div className="p-6 space-y-6">
      <h1 className="text-2xl font-bold">Dashboard Ejecutivo</h1>

      {/* KPIs principales */}
      <StatsGrid stats={[
        { label: 'Ingresos del Mes', value: stats?.monthlyRevenue, icon: '💰', trend: '+12%' },
        { label: 'Productos en Stock', value: stats?.totalProducts, icon: '📦', trend: '-3' },
        { label: 'Empleados Activos', value: stats?.activeEmployees, icon: '👥', trend: 'stable' },
        { label: 'Alertas de Stock', value: stats?.lowStockCount, icon: '⚠️', trend: 'warning' },
      ]} />

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <SalesLineChart />
        </div>
        <div className="space-y-4">
          <StockAlerts />
          <RecentTransactions />
        </div>
      </div>
    </div>
  );
};
```

---

## 🧠 Habilidades Demostradas en este Proyecto

| Habilidad | Cómo se demuestra |
|---|---|
| **Full Stack** | React + Node.js integrados de punta a punta |
| **SQL Avanzado** | Queries complejas, agregaciones, relaciones |
| **RBAC** | Roles y permisos a nivel de middleware y UI |
| **Real-time** | Socket.io para notificaciones push |
| **Testing** | >70% coverage en backend, E2E con Cypress |
| **DevOps** | Dockerizado + CI/CD con GitHub Actions |
| **Arquitectura** | Clean Architecture, principles SOLID |

---

## ✅ Entregables del Capstone

- [ ] Sistema deployado y accesible públicamente (ECS, K8s o Railway).
- [ ] 5 módulos funcionales: Auth, RRHH, Inventario, Financiero, Reportes.
- [ ] RBAC: 3 roles con permisos diferenciados.
- [ ] Tests con >70% de coverage del backend.
- [ ] Pipeline CI/CD completo en GitHub Actions.
- [ ] README del proyecto con demo, arquitectura y capturas de pantalla.
- [ ] Video demo de 3-5 minutos para el portafolio.

---

*← [Semana 24](../6-mes/semana-24-k8s-fullstack.md) | [Volver al Roadmap](../../README.md) | [Semana 26 →](../6-mes/semana-26-portfolio-review.md)*
