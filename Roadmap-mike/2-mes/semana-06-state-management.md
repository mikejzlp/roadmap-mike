# 🔄 Semana 06 — Gestión de Estado Global (Redux Toolkit & Zustand)

> **Mes 2 · Frontend & UI** | Stack: `Redux Toolkit` · `Zustand` · `Context API` · `React` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Resolver el problema del **prop-drilling** y el estado complejo en aplicaciones React mediante el uso de librerías de gestión de estado global. Comparar y practicar **Redux Toolkit** (el estándar de la industria) y **Zustand** (solución minimalista y moderna).

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| Redux Toolkit | ^2.x | Gestión de estado global (boilerplate reducido) |
| React-Redux | ^9.x | Conecta el store de Redux con los componentes React |
| Zustand | ^4.x | Alternativa minimalista al Redux |
| Redux DevTools | — | Extensión de Chrome para depurar el estado |

---

## 📐 Arquitectura Redux Toolkit

```
src/
├── store/
│   ├── index.ts                 # Configuración del Store global
│   └── slices/
│       ├── cartSlice.ts         # Slice del carrito de compras
│       └── authSlice.ts         # Slice de autenticación
├── hooks/
│   └── redux.ts                 # useAppDispatch y useAppSelector tipados
├── components/
│   ├── Cart/
│   │   ├── CartList.tsx
│   │   └── CartItem.tsx
│   └── Header.tsx
└── App.tsx
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación

```bash
# Redux Toolkit
npm install @reduxjs/toolkit react-redux

# Zustand (para la sección comparativa)
npm install zustand
```

### 2. Crear un Slice (Redux Toolkit)

```typescript
// src/store/slices/cartSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface Product {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: Product[];
  total: number;
}

const initialState: CartState = { items: [], total: 0 };

export const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem: (state, action: PayloadAction<Product>) => {
      const existing = state.items.find((i) => i.id === action.payload.id);
      if (existing) {
        existing.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
      state.total = state.items.reduce((acc, i) => acc + i.price * i.quantity, 0);
    },
    removeItem: (state, action: PayloadAction<number>) => {
      state.items = state.items.filter((i) => i.id !== action.payload);
      state.total = state.items.reduce((acc, i) => acc + i.price * i.quantity, 0);
    },
    clearCart: (state) => {
      state.items = [];
      state.total = 0;
    },
  },
});

export const { addItem, removeItem, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

### 3. Configurar el Store

```typescript
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './slices/cartSlice';
import authReducer from './slices/authSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    auth: authReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 4. Hooks tipados personalizados

```typescript
// src/hooks/redux.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from '../store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### 5. Usar Redux en un componente

```typescript
// src/components/Cart/CartList.tsx
import { useAppSelector, useAppDispatch } from '../../hooks/redux';
import { removeItem, clearCart } from '../../store/slices/cartSlice';

export const CartList = () => {
  const { items, total } = useAppSelector((state) => state.cart);
  const dispatch = useAppDispatch();

  return (
    <div>
      <h2>Carrito — Total: ${total.toFixed(2)}</h2>
      {items.map((item) => (
        <div key={item.id}>
          <span>{item.name} x{item.quantity}</span>
          <button onClick={() => dispatch(removeItem(item.id))}>Eliminar</button>
        </div>
      ))}
      <button onClick={() => dispatch(clearCart())}>Vaciar carrito</button>
    </div>
  );
};
```

### 6. Equivalente con Zustand (más simple)

```typescript
// src/store/useCartStore.ts
import { create } from 'zustand';

interface CartStore {
  items: { id: number; name: string; price: number; quantity: number }[];
  total: number;
  addItem: (item: { id: number; name: string; price: number }) => void;
  removeItem: (id: number) => void;
}

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  total: 0,
  addItem: (item) =>
    set((state) => {
      const existing = state.items.find((i) => i.id === item.id);
      const newItems = existing
        ? state.items.map((i) => (i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i))
        : [...state.items, { ...item, quantity: 1 }];
      return { items: newItems, total: newItems.reduce((acc, i) => acc + i.price * i.quantity, 0) };
    }),
  removeItem: (id) =>
    set((state) => {
      const newItems = state.items.filter((i) => i.id !== id);
      return { items: newItems, total: newItems.reduce((acc, i) => acc + i.price * i.quantity, 0) };
    }),
}));
```

---

## ⚖️ Redux Toolkit vs Zustand

| Aspecto | Redux Toolkit | Zustand |
|---|---|---|
| Boilerplate | Medio (slices, actions) | Mínimo |
| DevTools | Excelentes (Redux DevTools) | Básicas |
| Curva de aprendizaje | Media | Baja |
| Uso recomendado | Apps enterprise/grandes | Apps medianas/pequeñas |
| TypeScript | Soportado, requiere más tipos | Inferencia automática |

---

## 🧠 Conceptos Clave Aprendidos

- **Store**: Contenedor centralizado e inmutable del estado de la app.
- **Action**: Objeto `{ type, payload }` que describe un cambio de estado.
- **Reducer**: Función pura `(state, action) => newState`. En RTK usa Immer (mutaciones directas seguras).
- **Slice**: Agrupación de un reducer + sus actions relacionadas.
- **Selector**: Función que extrae una parte del store para usarla en un componente.

---

## ✅ Entregables

- [ ] Store de Redux Toolkit configurado con al menos 2 slices.
- [ ] Componente de carrito de compras usando `useAppSelector` y `useAppDispatch`.
- [ ] Misma funcionalidad del carrito replicada con Zustand.
- [ ] Redux DevTools habilitadas y funcionando en el navegador.

---

## 📖 Recursos de Referencia

- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Zustand GitHub](https://github.com/pmndrs/zustand)
- [Redux DevTools Extension](https://github.com/reduxjs/redux-devtools)

---

*← [Semana 05](../2-mes/semana-05-react-foundations.md) | [Volver al Roadmap](../../README.md) | [Semana 07 →](../2-mes/semana-07-tailwind-material.md)*
