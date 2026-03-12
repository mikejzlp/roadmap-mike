# 🔃 Semana 08 — Data Fetching & Cache con React Query

> **Mes 2 · Frontend & UI** | Stack: `TanStack Query (React Query)` · `Axios` · `React` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Manejar el **estado del servidor** (datos remotos) de forma profesional usando **TanStack Query v5**. Implementar fetching de datos, caché automático, estados de carga/error, paginación y mutaciones con invalidación de caché.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| TanStack Query | ^5.x | Gestión de estado del servidor, caché, sincronización |
| Axios | ^1.x | Cliente HTTP con interceptores y tipado |
| @tanstack/react-query-devtools | ^5.x | DevTools para inspeccionar el caché |

---

## 📐 Arquitectura del Proyecto

```
semana-08-react-query/
├── src/
│   ├── api/
│   │   ├── client.ts            # Instancia de Axios configurada
│   │   └── users.api.ts         # Funciones de llamadas a la API
│   ├── hooks/
│   │   ├── useUsers.ts          # Hook useQuery para usuarios
│   │   └── useCreateUser.ts     # Hook useMutation para crear usuario
│   ├── components/
│   │   ├── UserList.tsx         # Lista con estados loading/error
│   │   └── CreateUserForm.tsx   # Formulario con mutación
│   ├── providers/
│   │   └── QueryProvider.tsx    # Configuración del QueryClient
│   └── App.tsx
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación

```bash
npm install @tanstack/react-query axios
npm install -D @tanstack/react-query-devtools
```

### 2. Configurar el QueryClient

```typescript
// src/providers/QueryProvider.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { ReactNode } from 'react';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // 5 minutos: los datos se consideran frescos
      gcTime: 1000 * 60 * 10,     // 10 minutos en caché antes de limpiar
      retry: 2,                   // Reintentar 2 veces en caso de error
      refetchOnWindowFocus: true, // Re-fetch al volver a la pestaña
    },
  },
});

export const QueryProvider = ({ children }: { children: ReactNode }) => (
  <QueryClientProvider client={queryClient}>
    {children}
    <ReactQueryDevtools initialIsOpen={false} />
  </QueryClientProvider>
);
```

### 3. Cliente HTTP con Axios

```typescript
// src/api/client.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
  headers: { 'Content-Type': 'application/json' },
});

// Interceptor para añadir el token JWT automáticamente
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Interceptor para manejar errores globalmente
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### 4. Hook `useQuery` para listar datos

```typescript
// src/hooks/useUsers.ts
import { useQuery } from '@tanstack/react-query';
import { apiClient } from '../api/client';

interface User {
  id: number;
  name: string;
  email: string;
}

const fetchUsers = async (): Promise<User[]> => {
  const { data } = await apiClient.get<User[]>('/users');
  return data;
};

export const useUsers = () =>
  useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
```

### 5. Componente con estados de carga

```typescript
// src/components/UserList.tsx
import { useUsers } from '../hooks/useUsers';

export const UserList = () => {
  const { data: users, isLoading, isError, error } = useUsers();

  if (isLoading) return <div className="animate-pulse">Cargando usuarios...</div>;
  if (isError) return <div className="text-red-500">Error: {(error as Error).message}</div>;

  return (
    <ul className="space-y-2">
      {users?.map((user) => (
        <li key={user.id} className="p-4 bg-white rounded-lg shadow">
          <strong>{user.name}</strong> — {user.email}
        </li>
      ))}
    </ul>
  );
};
```

### 6. Hook `useMutation` para crear datos

```typescript
// src/hooks/useCreateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from '../api/client';

interface CreateUserPayload {
  name: string;
  email: string;
}

export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (payload: CreateUserPayload) =>
      apiClient.post('/users', payload).then((r) => r.data),
    onSuccess: () => {
      // Invalidar caché para re-fetch automático de la lista
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **`staleTime`** | Tiempo que los datos se consideran frescos (sin re-fetch) |
| **`gcTime`** | Tiempo que los datos inactivos permanecen en caché |
| **`queryKey`** | Array que identifica y agrupa las queries. Base del sistema de caché |
| **`invalidateQueries`** | Fuerza un re-fetch de todas las queries que coincidan con la clave |
| **Optimistic Updates** | Actualizar el UI antes de que el servidor confirme (mejor UX) |
| **`prefetchQuery`** | Cargar datos en el caché antes de navegación |

---

## ✅ Entregables

- [ ] QueryClient configurado con `staleTime` y `gcTime` correctos.
- [ ] Cliente Axios con interceptores de request (JWT) y response (manejo de 401).
- [ ] `useQuery` funcionando con estados `isLoading` e `isError` en la UI.
- [ ] `useMutation` con `invalidateQueries` para mantener el caché sincronizado.
- [ ] React Query DevTools habilitadas para inspeccionar el caché.

---

## 📖 Recursos de Referencia

- [TanStack Query Docs](https://tanstack.com/query/latest)
- [Axios Docs — Interceptors](https://axios-http.com/docs/interceptors)
- [React Query Blog — Practical Guide](https://tkdodo.eu/blog/practical-react-query)

---

*← [Semana 07](../2-mes/semana-07-tailwind-material.md) | [Volver al Roadmap](../../README.md) | [Semana 09 →](../3-mes/semana-09-mern-dashboard.md)*
