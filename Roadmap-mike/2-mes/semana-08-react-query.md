# Semana 08: Data Fetching Crítico (React Query)

## 🎯 Objetivo
Manejar el estado del servidor (Data Fetching, Cache, Re-fetching, Mutaciones) de manera óptima sin reinventar la rueda con `useEffect`.

## 🛠️ Stack Tecnológico
- React Query (TanStack Query)
- Axios / Fetch API
- APIs Públicas

## 📋 Requisitos del Proyecto
1. Construir un "Buscador de Repositorios de GitHub" consumiendo la API pública de GitHub.
2. Instalar y configurar `@tanstack/react-query`.
3. Reemplazar cualquier uso de `useEffect` para llamar APIs por el hook `useQuery`.
4. Mostrar correctamente los estados asíncronos en la UI: `isLoading`, `isError`, y los `data`.
5. Mostrar paginación administrada y cacheada por React Query.

## ✅ Entregables
- Proyecto rápido sin repetición de llamadas de red innecesarias.
- Uso de `useMutation` para simular acciones (post, delete).
