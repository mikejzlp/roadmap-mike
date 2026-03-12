# ⚛️ Semana 05 — React Foundations

> **Mes 2 · Frontend & UI** | Stack: `React 18` · `TypeScript` · `Vite` · `Components` · `Hooks`

---

## 🎯 Objetivo del Proyecto

Construir una **SPA (Single-Page Application)** sólida desde cero con **React 18** y TypeScript usando **Vite** como bundler. Dominar el modelo mental de componentes, el flujo de datos con **props**, y los Hooks fundamentales: `useState`, `useEffect`, `useRef` y `useContext`.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| React | 18.x | Librería UI basada en componentes |
| TypeScript | ^5.x | Tipado estático para componentes y props |
| Vite | ^5.x | Build tool ultra-rápido para desarrollo |
| React DevTools | — | Extensión de Chrome para debug de React |

---

## 📐 Arquitectura del Proyecto

```
semana-05-react-foundations/
├── src/
│   ├── components/
│   │   ├── ui/
│   │   │   ├── Button.tsx          # Componente reutilizable con variantes
│   │   │   └── Card.tsx            # Componente contenedor genérico
│   │   └── features/
│   │       └── TodoList/
│   │           ├── TodoList.tsx    # Componente principal (lista)
│   │           ├── TodoItem.tsx    # Componente de ítem individual
│   │           └── TodoForm.tsx    # Formulario para añadir ítems
│   ├── context/
│   │   └── ThemeContext.tsx        # Context API para tema (dark/light)
│   ├── hooks/
│   │   └── useLocalStorage.ts     # Custom Hook reutilizable
│   ├── App.tsx
│   └── main.tsx
├── vite.config.ts
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Crear el proyecto con Vite

```bash
npm create vite@latest semana-05-react-foundations -- --template react-ts
cd semana-05-react-foundations
npm install
npm run dev
```

### 2. Componente con Props tipadas

```typescript
// src/components/ui/Button.tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'danger';
  onClick?: () => void;
  disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
  label,
  variant = 'primary',
  onClick,
  disabled = false,
}) => {
  const styles = {
    primary: 'bg-blue-500 text-white',
    secondary: 'bg-gray-200 text-gray-800',
    danger: 'bg-red-500 text-white',
  };

  return (
    <button
      className={`px-4 py-2 rounded ${styles[variant]}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

### 3. Hook `useState` y `useEffect`

```typescript
// src/components/features/TodoList/TodoList.tsx
import { useState, useEffect } from 'react';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

export const TodoList = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [input, setInput] = useState('');

  // Persistencia en localStorage
  useEffect(() => {
    const saved = localStorage.getItem('todos');
    if (saved) setTodos(JSON.parse(saved));
  }, []);

  useEffect(() => {
    localStorage.setItem('todos', JSON.stringify(todos));
  }, [todos]);

  const addTodo = () => {
    if (!input.trim()) return;
    setTodos([...todos, { id: Date.now(), text: input, completed: false }]);
    setInput('');
  };

  const toggleTodo = (id: number) => {
    setTodos(todos.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t)));
  };

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={addTodo}>Añadir</button>
      <ul>
        {todos.map((todo) => (
          <li
            key={todo.id}
            onClick={() => toggleTodo(todo.id)}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### 4. Context API para Tema Global

```typescript
// src/context/ThemeContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [theme, setTheme] = useState<Theme>('light');
  const toggleTheme = () => setTheme((t) => (t === 'light' ? 'dark' : 'light'));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
};
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **JSX** | Sintaxis que combina JS y HTML, compilada por Babel/Vite |
| **Componentes** | Funciones puras que reciben props y devuelven JSX |
| **Props** | Datos de padre a hijo (inmutables en el hijo) |
| **Estado (useState)** | Datos que al cambiar disparan un re-render del componente |
| **useEffect** | Efectos secundarios: fetch, subscripciones, localStorage |
| **Context API** | Estado global sin prop-drilling para temas, auth, etc. |
| **Virtual DOM** | React compara el árbol anterior y actual para actualizar solo lo necesario |

---

## ✅ Entregables

- [ ] App con al menos 5 componentes reutilizables.
- [ ] Todo list funcional con `useState` y persistencia con `useEffect`.
- [ ] Context API implementado para cambio de tema dark/light.
- [ ] Código completamente tipado con TypeScript (interfaces para props).
- [ ] Custom Hook `useLocalStorage` extraído y reutilizable.

---

## 📖 Recursos de Referencia

- [React Docs (react.dev)](https://react.dev/)
- [Vite.js — Why Vite?](https://vitejs.dev/guide/why.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

---

*← [Semana 04](../1-mes/semana-04-postgresql-orm.md) | [Volver al Roadmap](../../README.md) | [Semana 06 →](../2-mes/semana-06-state-management.md)*
