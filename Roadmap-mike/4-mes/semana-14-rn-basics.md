# 📱 Semana 14 — React Native Basics (CLI / Expo)

> **Mes 4 · Mobile App Development** | Stack: `React Native` · `Expo` · `TypeScript` · `React Navigation`

---

## 🎯 Objetivo del Proyecto

Configurar un proyecto **React Native** funcional y construir las primeras pantallas de una app mobile. Comprender las diferencias entre el desarrollo web y mobile, los componentes nativos de React Native y cómo funciona el bridge entre JavaScript y el código nativo.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión | Rol |
|---|---|---|
| React Native | 0.74+ | Framework cross-platform (iOS & Android) |
| Expo (SDK 51) | — | Herramienta de desarrollo y acceso a APIs nativas |
| TypeScript | ^5.x | Tipado estático |
| React Native Debugger | — | Herramienta de depuración especializada |

---

## 📐 Arquitectura del Proyecto

```
semana-14-rn-basics/
├── src/
│   ├── screens/
│   │   ├── HomeScreen.tsx        # Pantalla principal
│   │   └── ProfileScreen.tsx     # Perfil de usuario
│   ├── components/
│   │   ├── Avatar.tsx            # Componente de imagen circular
│   │   ├── PrimaryButton.tsx     # Botón nativo con feedback táctil
│   │   └── Card.tsx
│   ├── hooks/
│   │   └── useColorScheme.ts     # Detectar tema dark/light del sistema
│   └── styles/
│       └── theme.ts              # Paleta de colores y tipografía global
├── App.tsx
└── app.json
```

---

## 📋 Requisitos del Proyecto

### 1. Crear el proyecto con Expo

```bash
# Crear app con Expo y TypeScript
npx create-expo-app@latest semana-14-rn-basics --template

# Alternativa con React Native CLI (sin Expo)
npx react-native@latest init Semana14RNBasics --template react-native-template-typescript

# Ejecutar en Android (dispositivo o emulador)
npx expo start
# Presionar 'a' para Android, 'i' para iOS
```

### 2. Componentes Nativos vs Web

```typescript
// src/screens/HomeScreen.tsx
import {
  View,          // ≈ <div> en web
  Text,          // ≈ <span>/<p> — único componente que puede mostrar texto
  StyleSheet,    // API de estilos (similar a CSS-in-JS, pero limitado)
  FlatList,      // Lista virtualizada de alto rendimiento
  TouchableOpacity, // Área táctil con feedback visual
  SafeAreaView,  // Respeta los notches y bordes del dispositivo
  StatusBar,
} from 'react-native';
import { useColorScheme } from '../hooks/useColorScheme';

interface Item {
  id: string;
  title: string;
  description: string;
}

const SAMPLE_DATA: Item[] = [
  { id: '1', title: 'Item 1', description: 'Descripción del primer item' },
  { id: '2', title: 'Item 2', description: 'Descripción del segundo item' },
  { id: '3', title: 'Item 3', description: 'Descripción del tercer item' },
];

export const HomeScreen = () => {
  const colorScheme = useColorScheme();
  const isDark = colorScheme === 'dark';

  return (
    <SafeAreaView style={[styles.container, isDark && styles.darkContainer]}>
      <StatusBar barStyle={isDark ? 'light-content' : 'dark-content'} />
      <Text style={[styles.title, isDark && styles.darkText]}>
        🚀 Mi Primera App
      </Text>
      <FlatList
        data={SAMPLE_DATA}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity style={styles.card} activeOpacity={0.7}>
            <Text style={styles.cardTitle}>{item.title}</Text>
            <Text style={styles.cardDesc}>{item.description}</Text>
          </TouchableOpacity>
        )}
        contentContainerStyle={{ padding: 16, gap: 12 }}
      />
    </SafeAreaView>
  );
};

// StyleSheet.create compila styles en tiempo de carga (mejor rendimiento)
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  darkContainer: { backgroundColor: '#1a1a2e' },
  title: { fontSize: 28, fontWeight: 'bold', padding: 20, color: '#1a1a2e' },
  darkText: { color: '#ffffff' },
  card: {
    backgroundColor: '#ffffff',
    borderRadius: 16,
    padding: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.08,
    shadowRadius: 8,
    elevation: 4, // Android shadow
  },
  cardTitle: { fontSize: 16, fontWeight: '600', marginBottom: 4 },
  cardDesc: { fontSize: 14, color: '#666' },
});
```

### 3. Botón con feedback táctil

```typescript
// src/components/PrimaryButton.tsx
import { TouchableOpacity, Text, StyleSheet, ActivityIndicator } from 'react-native';

interface PrimaryButtonProps {
  label: string;
  onPress: () => void;
  loading?: boolean;
  disabled?: boolean;
}

export const PrimaryButton: React.FC<PrimaryButtonProps> = ({
  label, onPress, loading = false, disabled = false
}) => (
  <TouchableOpacity
    onPress={onPress}
    disabled={disabled || loading}
    style={[styles.btn, (disabled || loading) && styles.disabled]}
    activeOpacity={0.85}
  >
    {loading
      ? <ActivityIndicator color="#fff" />
      : <Text style={styles.label}>{label}</Text>
    }
  </TouchableOpacity>
);

const styles = StyleSheet.create({
  btn: {
    backgroundColor: '#3b82f6',
    paddingVertical: 14,
    paddingHorizontal: 24,
    borderRadius: 12,
    alignItems: 'center',
  },
  disabled: { opacity: 0.5 },
  label: { color: '#fff', fontSize: 16, fontWeight: '600' },
});
```

### 4. Sistema de tema global

```typescript
// src/styles/theme.ts
export const theme = {
  colors: {
    primary: '#3b82f6',
    secondary: '#8b5cf6',
    background: { light: '#f9f9f9', dark: '#1a1a2e' },
    text: { primary: '#1a1a2e', secondary: '#6b7280' },
    card: { light: '#ffffff', dark: '#16213e' },
  },
  typography: {
    h1: { fontSize: 32, fontWeight: '700' as const },
    h2: { fontSize: 24, fontWeight: '600' as const },
    body: { fontSize: 16, fontWeight: '400' as const },
    caption: { fontSize: 12, fontWeight: '400' as const, color: '#6b7280' },
  },
  spacing: { xs: 4, sm: 8, md: 16, lg: 24, xl: 32 },
  borderRadius: { sm: 8, md: 12, lg: 16, xl: 24 },
};
```

---

## ⚖️ React Native vs React Web: Diferencias Clave

| Aspecto | React (Web) | React Native (Mobile) |
|---|---|---|
| Contenedor | `<div>` | `<View>` |
| Texto | `<p>`, `<span>`, `<h1>` | `<Text>` (único tipo) |
| Lista | `<ul>/<li>` | `<FlatList>` (virtualizada) |
| Estilos | CSS, clases | `StyleSheet.create()`, sin cascada |
| Routing | React Router DOM | React Navigation |
| Click | `onClick` | `onPress` |
| Shadow | `box-shadow` CSS | `elevation` (Android) / `shadow*` (iOS) |

---

## ✅ Entregables

- [ ] App corriendo en Android/iOS (dispositivo o emulador).
- [ ] Al menos 2 pantallas con `FlatList` y componentes nativos.
- [ ] Sistema de tema dark/light basado en `useColorScheme`.
- [ ] Componentes reutilizables: `PrimaryButton`, `Card`, `Avatar`.
- [ ] Estilos unificados en `theme.ts`.

---

*← [Semana 13](../3-mes/semana-13-clean-architecture.md) | [Volver al Roadmap](../../README.md) | [Semana 15 →](../4-mes/semana-15-rn-navigation.md)*
