# 🔌 Semana 16 — Integración APIs Mobile (Fetch, Zustand, Listas)

> **Mes 4 · Mobile App Development** | Stack: `React Native` · `TanStack Query` · `Zustand` · `Axios` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Conectar la app móvil con una **API REST real** (la construida en los meses anteriores). Implementar fetching de datos, manejo de estados de carga y error, listas paginadas de alto rendimiento con `FlatList`, y gestión de estado offline-first con **Zustand + MMKV**.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| Axios | Cliente HTTP con interceptores y tipado |
| TanStack Query (React Native) | Caché, sincronización y estados de fetching |
| Zustand + MMKV | Estado global con persistencia nativa (más rápido que AsyncStorage) |
| react-native-mmkv | Storage clave-valor eficiente en React Native |
| FlashList (Shopify) | FlatList de alta performance para listas largas |

---

## 📐 Arquitectura del Proyecto

```
semana-16-rn-api-sync/
├── src/
│   ├── api/
│   │   ├── client.ts              # Axios configurado para mobile
│   │   └── products.api.ts        # Funciones de llamada a la API
│   ├── store/
│   │   └── useAuthStore.ts        # Estado de autenticación con Zustand + MMKV
│   ├── hooks/
│   │   ├── useProducts.ts         # useQuery para productos
│   │   └── useInfiniteProducts.ts # Paginación infinita
│   ├── screens/
│   │   ├── ProductListScreen.tsx  # Lista infinite scroll
│   │   └── ProductDetailScreen.tsx
│   └── components/
│       ├── ProductCard.tsx
│       └── LoadingFooter.tsx      # Indicador de carga en la parte inferior
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación

```bash
npm install axios @tanstack/react-query zustand
npm install react-native-mmkv
npx pod-install  # Solo iOS
```

### 2. Axios para React Native

```typescript
// src/api/client.ts
import axios from 'axios';
import { useAuthStore } from '../store/useAuthStore';

// En desarrollo: usa la IP de tu máquina (NO localhost en emulador Android)
const BASE_URL = __DEV__
  ? 'http://192.168.1.X:3000/api'  // Tu IP local
  : 'https://tu-api-en-produccion.com/api';

export const apiClient = axios.create({ baseURL: BASE_URL });

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

### 3. Store de Auth con persistencia MMKV

```typescript
// src/store/useAuthStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

const mmkvStorage = {
  getItem: (key: string) => storage.getString(key) ?? null,
  setItem: (key: string, value: string) => storage.set(key, value),
  removeItem: (key: string) => storage.delete(key),
};

interface AuthStore {
  token: string | null;
  user: { id: string; name: string; email: string } | null;
  setAuth: (token: string, user: AuthStore['user']) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      setAuth: (token, user) => set({ token, user }),
      logout: () => set({ token: null, user: null }),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => mmkvStorage),
    }
  )
);
```

### 4. Paginación Infinita con TanStack Query

```typescript
// src/hooks/useInfiniteProducts.ts
import { useInfiniteQuery } from '@tanstack/react-query';
import { apiClient } from '../api/client';

interface Product {
  id: number;
  name: string;
  price: number;
  imageUrl: string;
}

interface ProductsPage {
  data: Product[];
  nextPage: number | null;
  totalPages: number;
}

export const useInfiniteProducts = (category?: string) =>
  useInfiniteQuery({
    queryKey: ['products', category],
    queryFn: ({ pageParam = 1 }) =>
      apiClient
        .get<ProductsPage>('/products', { params: { page: pageParam, limit: 20, category } })
        .then((r) => r.data),
    initialPageParam: 1,
    getNextPageParam: (lastPage) => lastPage.nextPage,
  });
```

### 5. Lista con Infinite Scroll

```typescript
// src/screens/ProductListScreen.tsx
import { FlatList, ActivityIndicator, View, Text, RefreshControl } from 'react-native';
import { useInfiniteProducts } from '../hooks/useInfiniteProducts';
import { ProductCard } from '../components/ProductCard';

export const ProductListScreen = () => {
  const {
    data,
    isLoading,
    isError,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    refetch,
    isRefetching,
  } = useInfiniteProducts();

  const products = data?.pages.flatMap((page) => page.data) ?? [];

  if (isLoading) return <ActivityIndicator size="large" style={{ flex: 1 }} />;
  if (isError) return <Text>Error al cargar productos</Text>;

  return (
    <FlatList
      data={products}
      keyExtractor={(item) => item.id.toString()}
      renderItem={({ item }) => <ProductCard product={item} />}
      numColumns={2}                    // Grid de 2 columnas
      columnWrapperStyle={{ gap: 12, paddingHorizontal: 16 }}
      contentContainerStyle={{ paddingVertical: 16, gap: 12 }}
      onEndReached={() => hasNextPage && fetchNextPage()}
      onEndReachedThreshold={0.3}       // Cargar más cuando queda 30% del scroll
      ListFooterComponent={
        isFetchingNextPage ? <ActivityIndicator style={{ padding: 16 }} /> : null
      }
      refreshControl={
        <RefreshControl refreshing={isRefetching} onRefresh={refetch} />
      }
    />
  );
};
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **IP Local vs localhost** | En Android, `localhost` apunta al emulador. Usar la IP de la máquina (`192.168.x.x`) |
| **MMKV vs AsyncStorage** | MMKV es síncrono y hasta 10x más rápido que AsyncStorage |
| **Infinite Query** | `getNextPageParam` controla la paginación. `data.pages.flatMap()` aplana las páginas |
| **`numColumns`** | Crea un grid en FlatList. Requiere `keyExtractor` por id único |
| **`onEndReached`** | Trigger para cargar más datos cuando el usuario llega al final de la lista |

---

## ✅ Entregables

- [ ] Axios configurado con la URL de la API local y el token JWT.
- [ ] Store de auth persistido en MMKV.
- [ ] Lista de productos con infinite scroll (paginación automática).
- [ ] Pull-to-refresh con `RefreshControl`.
- [ ] Grid de 2 columnas con `numColumns={2}`.

---

*← [Semana 15](../4-mes/semana-15-rn-navigation.md) | [Volver al Roadmap](../../README.md) | [Semana 17 →](../4-mes/semana-17-rn-advanced.md)*
