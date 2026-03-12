# 🎨 Semana 07 — UI Modernas con Tailwind CSS & Material UI

> **Mes 2 · Frontend & UI** | Stack: `Tailwind CSS v3` · `Material UI (MUI)` · `React` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Dominar dos paradigmas de diseño de UI en React: **Tailwind CSS** (utility-first CSS framework) y **Material UI** (component library basada en Material Design de Google). Construir un dashboard visual y responsivo que demuestre ambos enfoques.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión recomendada | Rol |
|---|---|---|
| Tailwind CSS | ^3.x | Framework de clases utilitarias de CSS |
| Material UI (MUI) | ^5.x | Librería de componentes UI pre-construidos |
| @emotion/react | ^11.x | Dependencia de estilos para MUI |
| clsx | ^2.x | Utilitario para combinar clases CSS condicionalmente |

---

## 📐 Arquitectura del Proyecto

```
semana-07-tailwind-material/
├── src/
│   ├── components/
│   │   ├── tailwind/           # Componentes construidos con Tailwind
│   │   │   ├── NavBar.tsx
│   │   │   ├── StatCard.tsx
│   │   │   └── DataTable.tsx
│   │   └── mui/                # Componentes construidos con MUI
│   │       ├── SideDrawer.tsx
│   │       └── FormModal.tsx
│   ├── layouts/
│   │   └── DashboardLayout.tsx
│   ├── pages/
│   │   └── Dashboard.tsx
│   └── App.tsx
├── tailwind.config.ts
└── package.json
```

---

## 📋 Requisitos del Proyecto

### 1. Instalar Tailwind CSS con Vite

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
    },
  },
  plugins: [],
} satisfies Config;
```

```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 2. Componente StatCard con Tailwind

```typescript
// src/components/tailwind/StatCard.tsx
interface StatCardProps {
  title: string;
  value: string | number;
  change: number;     // % de cambio
  icon: React.ReactNode;
}

export const StatCard: React.FC<StatCardProps> = ({ title, value, change, icon }) => {
  const isPositive = change >= 0;

  return (
    <div className="bg-white rounded-2xl shadow-md p-6 flex items-center gap-4 hover:shadow-lg transition-shadow duration-300">
      <div className="bg-primary-50 p-3 rounded-xl text-primary-500">
        {icon}
      </div>
      <div>
        <p className="text-sm text-gray-500 font-medium">{title}</p>
        <p className="text-2xl font-bold text-gray-800">{value}</p>
        <p className={`text-sm font-medium ${isPositive ? 'text-green-500' : 'text-red-500'}`}>
          {isPositive ? '↑' : '↓'} {Math.abs(change)}% este mes
        </p>
      </div>
    </div>
  );
};
```

### 3. Instalar MUI

```bash
npm install @mui/material @emotion/react @emotion/styled @mui/icons-material
```

### 4. Modal de Formulario con MUI

```typescript
// src/components/mui/FormModal.tsx
import {
  Dialog, DialogTitle, DialogContent, DialogActions,
  TextField, Button
} from '@mui/material';
import { useState } from 'react';

interface FormModalProps {
  open: boolean;
  onClose: () => void;
  onSubmit: (data: { name: string; email: string }) => void;
}

export const FormModal: React.FC<FormModalProps> = ({ open, onClose, onSubmit }) => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  return (
    <Dialog open={open} onClose={onClose} maxWidth="sm" fullWidth>
      <DialogTitle>Crear nuevo usuario</DialogTitle>
      <DialogContent>
        <TextField
          label="Nombre"
          fullWidth
          margin="normal"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
        <TextField
          label="Email"
          type="email"
          fullWidth
          margin="normal"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </DialogContent>
      <DialogActions>
        <Button onClick={onClose}>Cancelar</Button>
        <Button variant="contained" onClick={() => onSubmit({ name, email })}>
          Crear
        </Button>
      </DialogActions>
    </Dialog>
  );
};
```

### 5. Layout responsivo del Dashboard

```typescript
// src/layouts/DashboardLayout.tsx
export const DashboardLayout = ({ children }: { children: React.ReactNode }) => (
  <div className="min-h-screen bg-gray-50 flex">
    {/* Sidebar */}
    <aside className="w-64 bg-white shadow-sm hidden lg:block p-6">
      <h2 className="text-lg font-bold text-gray-800 mb-6">Dashboard</h2>
      <nav className="space-y-2">
        {['Inicio', 'Usuarios', 'Reportes', 'Configuración'].map((item) => (
          <a key={item} href="#" className="flex items-center gap-3 px-4 py-2 rounded-lg text-gray-600 hover:bg-primary-50 hover:text-primary-500 transition-colors">
            {item}
          </a>
        ))}
      </nav>
    </aside>
    {/* Main content */}
    <main className="flex-1 p-6 lg:p-8">
      {children}
    </main>
  </div>
);
```

---

## ⚖️ Tailwind vs MUI: ¿Cuándo usar cada uno?

| Aspecto | Tailwind CSS | Material UI |
|---|---|---|
| Filosofía | Utility-first (clases en HTML) | Component-based (componentes pre-construidos) |
| Personalización | Total libertad de diseño | Limitado por el sistema de tema de MUI |
| Curva de aprendizaje | Media (memorizar clases) | Media (API de componentes) |
| Tamaño del bundle | Pequeño (purge automático) | Más grande |
| Ideal para | Diseños custom, landing pages | Apps de gestión, dashboards rápidos |

---

## 🧠 Conceptos Clave Aprendidos

- **Utility-First CSS**: Tailwind predica el uso de clases atómicas (`flex`, `p-4`, `text-blue-500`) directamente en el JSX.
- **JIT (Just-In-Time)**: Tailwind v3 genera solo los estilos que se usan, resultando en bundles CSS mínimos.
- **Design System con MUI**: Configurar el `theme` de MUI centraliza colores, tipografía y espaciados en toda la app.
- **`clsx`**: Utilitario para combinar clases de forma condicional sin concatenación manual de strings.

---

## ✅ Entregables

- [ ] Dashboard con layout sidebar + content area usando Tailwind.
- [ ] Al menos 3 componentes Tailwind: `NavBar`, `StatCard`, `DataTable`.
- [ ] Al menos 2 componentes MUI: `Dialog/Modal` y `DataGrid` o `Table`.
- [ ] App completamente responsiva (mobile-first con breakpoints `sm:`, `lg:`).

---

## 📖 Recursos de Referencia

- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [Material UI Docs](https://mui.com/material-ui/getting-started/)
- [Heroicons — Iconos para Tailwind](https://heroicons.com/)

---

*← [Semana 06](../2-mes/semana-06-state-management.md) | [Volver al Roadmap](../../README.md) | [Semana 08 →](../2-mes/semana-08-react-query.md)*
