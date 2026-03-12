# 📊 Semana 09 — Dashboard B2B Full Stack (MERN Stack)

> **Mes 3 · Full Stack Integration** | Stack: `MongoDB` · `Express` · `React` · `Node.js` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Construir un **dashboard administrativo B2B completo** integrando el stack MERN de punta a punta. El backend expone una API REST con autenticación JWT, y el frontend React consume los datos usando React Query, mostrando métricas, tablas de datos y gráficos en tiempo real.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| Node.js + Express | Backend API REST |
| MongoDB + Mongoose | Base de datos de documentos |
| React + TypeScript | Frontend SPA |
| TanStack Query | Gestión del estado del servidor |
| Recharts | Librería de gráficos SVG para React |
| Tailwind CSS | Estilos y layout del dashboard |

---

## 📐 Arquitectura del Proyecto (Monorepo)

```
semana-09-mern-dashboard/
├── backend/
│   ├── src/
│   │   ├── models/
│   │   │   ├── User.model.ts
│   │   │   └── Order.model.ts
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── orders.routes.ts
│   │   │   └── stats.routes.ts    # Endpoint de métricas
│   │   ├── controllers/
│   │   └── app.ts
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   └── Dashboard.tsx
│   │   ├── components/
│   │   │   ├── StatsGrid.tsx
│   │   │   ├── SalesChart.tsx
│   │   │   └── OrdersTable.tsx
│   │   └── App.tsx
│   └── package.json
└── docker-compose.yml            # Opcional: orquestar servicios
```

---

## 📋 Requisitos del Proyecto

### 1. Modelo de Pedido (Order)

```typescript
// backend/src/models/Order.model.ts
import { Schema, model, Document, Types } from 'mongoose';

export interface IOrder extends Document {
  customer: string;
  items: { product: string; quantity: number; price: number }[];
  total: number;
  status: 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled';
  createdAt: Date;
}

const orderSchema = new Schema<IOrder>(
  {
    customer: { type: String, required: true },
    items: [
      {
        product: String,
        quantity: { type: Number, min: 1 },
        price: { type: Number, min: 0 },
      },
    ],
    total: { type: Number, required: true },
    status: {
      type: String,
      enum: ['pending', 'processing', 'shipped', 'delivered', 'cancelled'],
      default: 'pending',
    },
  },
  { timestamps: true }
);

export const Order = model<IOrder>('Order', orderSchema);
```

### 2. Endpoint de estadísticas

```typescript
// backend/src/controllers/stats.controller.ts
import { Request, Response } from 'express';
import { Order } from '../models/Order.model';

export const getDashboardStats = async (req: Request, res: Response) => {
  const [totalOrders, totalRevenue, ordersByStatus, salesByDay] = await Promise.all([
    Order.countDocuments(),
    Order.aggregate([{ $group: { _id: null, total: { $sum: '$total' } } }]),
    Order.aggregate([{ $group: { _id: '$status', count: { $sum: 1 } } }]),
    Order.aggregate([
      {
        $group: {
          _id: { $dateToString: { format: '%Y-%m-%d', date: '$createdAt' } },
          revenue: { $sum: '$total' },
        },
      },
      { $sort: { '_id': 1 } },
      { $limit: 30 },
    ]),
  ]);

  res.json({
    totalOrders,
    totalRevenue: totalRevenue[0]?.total || 0,
    ordersByStatus,
    salesByDay,
  });
};
```

### 3. Gráfico de ventas con Recharts

```typescript
// frontend/src/components/SalesChart.tsx
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
import { useQuery } from '@tanstack/react-query';
import { apiClient } from '../api/client';

export const SalesChart = () => {
  const { data } = useQuery({
    queryKey: ['stats', 'sales'],
    queryFn: () => apiClient.get('/stats').then((r) => r.data.salesByDay),
  });

  return (
    <div className="bg-white rounded-2xl p-6 shadow-sm">
      <h3 className="text-lg font-semibold mb-4">Ventas últimos 30 días</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data}>
          <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
          <XAxis dataKey="_id" tick={{ fontSize: 12 }} />
          <YAxis tick={{ fontSize: 12 }} />
          <Tooltip formatter={(value) => [`$${value}`, 'Ventas']} />
          <Line
            type="monotone"
            dataKey="revenue"
            stroke="#3b82f6"
            strokeWidth={2}
            dot={false}
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
};
```

---

## 🧠 Conceptos Clave Aprendidos

- **Aggregation Pipeline de MongoDB**: `$group`, `$sum`, `$dateToString` para calcular métricas complejas en la base de datos.
- **Promise.all**: Ejecutar múltiples queries en paralelo para mejorar el tiempo de respuesta del endpoint de stats.
- **CORS**: Configurar `cors` en Express para permitir requests desde el frontend React en un puerto diferente.
- **Recharts**: Librería de gráficos basada en SVG, completamente controlada por props de React.

---

## ✅ Entregables

- [ ] Backend con endpoints: CRUD de órdenes + endpoint de estadísticas usando MongoDB Aggregation.
- [ ] Frontend con 4 widgets: Total Órdenes, Ingresos Totales, Gráfico de Ventas, Tabla de Órdenes.
- [ ] Rutas protegidas con JWT en el backend.
- [ ] CORS configurado correctamente para comunicación Frontend → Backend.

---

*← [Semana 08](../2-mes/semana-08-react-query.md) | [Volver al Roadmap](../../README.md) | [Semana 10 →](../3-mes/semana-10-websockets-chat.md)*
