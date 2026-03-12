# 🛒 Semana 11 — E-Commerce Full Stack (PERN Stack)

> **Mes 3 · Full Stack Integration** | Stack: `PostgreSQL` · `Express` · `React` · `Node.js` · `Prisma ORM` · `Stripe`

---

## 🎯 Objetivo del Proyecto

Construir una **tienda en línea funcional** usando el stack PERN (PostgreSQL, Express, React, Node.js). El sistema debe incluir catálogo de productos, carrito de compras, proceso de pago con **Stripe**, gestión de órdenes y panel de administración básico.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| PostgreSQL + Prisma | Base de datos relacional con relaciones complejas |
| Express + Node.js | API REST backend |
| React + Zustand | Frontend + gestión del carrito |
| Stripe | Procesamiento de pagos |
| Cloudinary | Almacenamiento de imágenes de productos |

---

## 📐 Schema de Prisma (Relaciones)

```prisma
// prisma/schema.prisma

model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  role      Role      @default(CUSTOMER)
  orders    Order[]
  createdAt DateTime  @default(now())
}

enum Role {
  ADMIN
  CUSTOMER
}

model Product {
  id          Int         @id @default(autoincrement())
  name        String
  description String
  price       Decimal     @db.Decimal(10, 2)
  stock       Int         @default(0)
  imageUrl    String?
  category    Category    @relation(fields: [categoryId], references: [id])
  categoryId  Int
  orderItems  OrderItem[]
}

model Category {
  id       Int       @id @default(autoincrement())
  name     String    @unique
  products Product[]
}

model Order {
  id         Int         @id @default(autoincrement())
  user       User        @relation(fields: [userId], references: [id])
  userId     Int
  items      OrderItem[]
  total      Decimal     @db.Decimal(10, 2)
  status     OrderStatus @default(PENDING)
  stripeId   String?     // ID de la sesión de Stripe
  createdAt  DateTime    @default(now())
}

enum OrderStatus {
  PENDING
  PAID
  SHIPPED
  DELIVERED
  CANCELLED
}

model OrderItem {
  id        Int     @id @default(autoincrement())
  order     Order   @relation(fields: [orderId], references: [id])
  orderId   Int
  product   Product @relation(fields: [productId], references: [id])
  productId Int
  quantity  Int
  price     Decimal @db.Decimal(10, 2) // Precio al momento de la compra
}
```

---

## 📋 Requisitos del Proyecto

### 1. Integración con Stripe (Checkout Session)

```bash
npm install stripe
```

```typescript
// backend/src/controllers/payments.controller.ts
import Stripe from 'stripe';
import { Request, Response } from 'express';
import { prisma } from '../lib/prisma';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY as string);

export const createCheckoutSession = async (req: Request, res: Response) => {
  const { items } = req.body; // [{ productId, quantity }]

  const products = await prisma.product.findMany({
    where: { id: { in: items.map((i: any) => i.productId) } },
  });

  const lineItems = items.map((item: any) => {
    const product = products.find((p) => p.id === item.productId)!;
    return {
      price_data: {
        currency: 'usd',
        product_data: { name: product.name, images: [product.imageUrl || ''] },
        unit_amount: Math.round(Number(product.price) * 100), // Stripe trabaja en centavos
      },
      quantity: item.quantity,
    };
  });

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: lineItems,
    mode: 'payment',
    success_url: `${process.env.CLIENT_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.CLIENT_URL}/cart`,
  });

  res.json({ url: session.url });
};
```

### 2. Webhook de Stripe para confirmar pagos

```typescript
// backend/src/controllers/webhook.controller.ts
import Stripe from 'stripe';
import { Request, Response } from 'express';
import { prisma } from '../lib/prisma';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY as string);

export const stripeWebhook = async (req: Request, res: Response) => {
  const sig = req.headers['stripe-signature'] as string;
  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      req.body,  // Raw body buffer, NO parsear como JSON antes
      sig,
      process.env.STRIPE_WEBHOOK_SECRET as string
    );
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${(err as Error).message}`);
  }

  if (event.type === 'checkout.session.completed') {
    const session = event.data.object as Stripe.Checkout.Session;
    // Actualizar la orden a PAID en la base de datos
    await prisma.order.updateMany({
      where: { stripeId: session.id },
      data: { status: 'PAID' },
    });
  }

  res.json({ received: true });
};
```

### 3. Store del carrito con Zustand (Frontend)

```typescript
// frontend/src/store/useCartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CartItem {
  productId: number;
  name: string;
  price: number;
  quantity: number;
  imageUrl?: string;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (productId: number) => void;
  updateQuantity: (productId: number, quantity: number) => void;
  total: () => number;
  checkout: () => Promise<void>;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      addItem: (item) =>
        set((state) => {
          const existing = state.items.find((i) => i.productId === item.productId);
          if (existing) return { items: state.items.map((i) => i.productId === item.productId ? { ...i, quantity: i.quantity + 1 } : i) };
          return { items: [...state.items, { ...item, quantity: 1 }] };
        }),
      removeItem: (id) => set((state) => ({ items: state.items.filter((i) => i.productId !== id) })),
      updateQuantity: (id, qty) =>
        set((state) => ({ items: state.items.map((i) => (i.productId === id ? { ...i, quantity: qty } : i)) })),
      total: () => get().items.reduce((acc, i) => acc + i.price * i.quantity, 0),
      checkout: async () => {
        const { data } = await fetch('/api/payments/checkout', {
          method: 'POST',
          body: JSON.stringify({ items: get().items }),
          headers: { 'Content-Type': 'application/json' },
        }).then((r) => r.json());
        window.location.href = data.url; // Redirigir a Stripe Checkout
      },
    }),
    { name: 'cart-storage' } // Persistir en localStorage
  )
);
```

---

## 🧠 Conceptos Clave Aprendidos

- **Relaciones en Prisma**: Schema con múltiples tablas relacionadas (`User → Order → OrderItem → Product`).
- **Stripe Checkout Sessions**: Delegar el proceso de pago seguro a Stripe (nunca manejar números de tarjeta directamente).
- **Webhooks**: Confirmación asíncrona de eventos de Stripe, más confiable que el redirect de éxito.
- **`zustand/middleware/persist`**: Persistir el carrito en `localStorage` para que no se pierda al recargar.

---

## ✅ Entregables

- [ ] Schema de Prisma con todas las relaciones (`User`, `Product`, `Order`, `OrderItem`).
- [ ] Catálogo de productos con filtro por categorías.
- [ ] Carrito persistente con Zustand `persist`.
- [ ] Integración con Stripe Checkout (modo test).
- [ ] Webhook de Stripe que actualiza el estado de la orden a `PAID`.

---

*← [Semana 10](../3-mes/semana-10-websockets-chat.md) | [Volver al Roadmap](../../README.md) | [Semana 12 →](../3-mes/semana-12-testing-jest-cypress.md)*
