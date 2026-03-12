# 🧭 Semana 15 — Navegación Móvil (React Navigation)

> **Mes 4 · Mobile App Development** | Stack: `React Navigation v6` · `React Native` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Implementar un sistema de **navegación multi-pantalla** completo usando **React Navigation**, el estándar de facto en React Native. Dominar los tres tipos de navegadores principales: Stack, Tab y Drawer, y aprender a tipar correctamente los parámetros de navegación con TypeScript.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión | Rol |
|---|---|---|
| @react-navigation/native | ^6.x | Core de la navegación |
| @react-navigation/native-stack | ^6.x | Navegación tipo stack (historial) |
| @react-navigation/bottom-tabs | ^6.x | Barra de tabs inferior |
| @react-navigation/drawer | ^6.x | Menú lateral deslizante |
| react-native-screens | ^3.x | Optimización de rendimiento de pantallas |
| @expo/vector-icons | — | Iconos para las tabs |

---

## 📐 Estructura de Navegación

```
App
└── DrawerNavigator
    ├── "Main" → BottomTabNavigator
    │   ├── "Home" → StackNavigator
    │   │   ├── HomeScreen
    │   │   └── DetailScreen (recibe params: { id: number })
    │   ├── "Search" → SearchScreen
    │   └── "Profile" → ProfileScreen
    └── "Settings" → SettingsScreen
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación

```bash
npm install @react-navigation/native @react-navigation/native-stack
npm install @react-navigation/bottom-tabs @react-navigation/drawer
npm install react-native-screens react-native-safe-area-context
npm install react-native-gesture-handler react-native-reanimated
```

### 2. Tipado de Parámetros de Navegación (TypeScript)

```typescript
// src/navigation/types.ts

// Tipado del Stack Navigator
export type RootStackParamList = {
  Home: undefined;                    // Sin parámetros
  Detail: { id: number; title: string }; // Con parámetros tipados
  Auth: undefined;
};

// Tipado del Tab Navigator
export type TabParamList = {
  HomeStack: undefined;
  Search: undefined;
  Profile: undefined;
};

// Tipado del Drawer Navigator
export type DrawerParamList = {
  Main: undefined;
  Settings: undefined;
};
```

### 3. Stack Navigator

```typescript
// src/navigation/HomeStack.tsx
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { HomeScreen } from '../screens/HomeScreen';
import { DetailScreen } from '../screens/DetailScreen';
import type { RootStackParamList } from './types';

const Stack = createNativeStackNavigator<RootStackParamList>();

export const HomeStack = () => (
  <Stack.Navigator
    screenOptions={{
      headerStyle: { backgroundColor: '#3b82f6' },
      headerTintColor: '#fff',
      headerTitleStyle: { fontWeight: '700' },
      animation: 'slide_from_right',
    }}
  >
    <Stack.Screen name="Home" component={HomeScreen} options={{ title: 'Inicio' }} />
    <Stack.Screen
      name="Detail"
      component={DetailScreen}
      options={({ route }) => ({ title: route.params.title })}
    />
  </Stack.Navigator>
);
```

### 4. Bottom Tab Navigator

```typescript
// src/navigation/MainTabs.tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Ionicons } from '@expo/vector-icons';
import { HomeStack } from './HomeStack';
import { SearchScreen } from '../screens/SearchScreen';
import { ProfileScreen } from '../screens/ProfileScreen';
import type { TabParamList } from './types';

const Tab = createBottomTabNavigator<TabParamList>();

export const MainTabs = () => (
  <Tab.Navigator
    screenOptions={({ route }) => ({
      headerShown: false,
      tabBarActiveTintColor: '#3b82f6',
      tabBarInactiveTintColor: '#9ca3af',
      tabBarStyle: {
        borderTopWidth: 0,
        elevation: 10,
        shadowOpacity: 0.1,
        height: 60,
        paddingBottom: 8,
      },
      tabBarIcon: ({ focused, color, size }) => {
        const icons: Record<keyof TabParamList, { active: string; inactive: string }> = {
          HomeStack: { active: 'home', inactive: 'home-outline' },
          Search: { active: 'search', inactive: 'search-outline' },
          Profile: { active: 'person', inactive: 'person-outline' },
        };
        const iconName = focused
          ? icons[route.name as keyof TabParamList].active
          : icons[route.name as keyof TabParamList].inactive;
        return <Ionicons name={iconName as any} size={size} color={color} />;
      },
    })}
  >
    <Tab.Screen name="HomeStack" component={HomeStack} options={{ title: 'Inicio' }} />
    <Tab.Screen name="Search" component={SearchScreen} />
    <Tab.Screen name="Profile" component={ProfileScreen} />
  </Tab.Navigator>
);
```

### 5. Navegar y pasar parámetros entre pantallas

```typescript
// src/screens/HomeScreen.tsx
import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import type { RootStackParamList } from '../navigation/types';

type HomeNavProp = NativeStackNavigationProp<RootStackParamList, 'Home'>;

export const HomeScreen = () => {
  const navigation = useNavigation<HomeNavProp>();

  return (
    <TouchableOpacity
      onPress={() => navigation.navigate('Detail', { id: 42, title: 'Mi Detalle' })}
    >
      <Text>Ver Detalle</Text>
    </TouchableOpacity>
  );
};

// src/screens/DetailScreen.tsx
import { useRoute } from '@react-navigation/native';
import type { NativeStackRouteProp } from '@react-navigation/native-stack';
import type { RootStackParamList } from '../navigation/types';

type DetailRouteProp = NativeStackRouteProp<RootStackParamList, 'Detail'>;

export const DetailScreen = () => {
  const route = useRoute<DetailRouteProp>();
  const { id, title } = route.params; // Tipado automático 🎉

  return <Text>Detalle #{id}: {title}</Text>;
};
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **Stack Navigator** | Historial de pantallas con gestos de retroceso (swipe back en iOS) |
| **Tab Navigator** | Navegación entre secciones principales, siempre visible |
| **Drawer Navigator** | Menú lateral para configuraciones y secciones secundarias |
| **Tipado de rutas** | `ParamList` + `NativeStackNavigationProp` elimina errores de tipeo en `navigate()` |
| **`headerShown: false`** | Ocultar el header del Tab Navigator para que el Stack interno lo controle |

---

## ✅ Entregables

- [ ] App con 3 tipos de navigation: Stack, Tabs y Drawer.
- [ ] Parámetros tipados entre pantallas con TypeScript.
- [ ] Iconos en la barra de tabs con Ionicons.
- [ ] Header personalizado con colores y título dinámico.

---

*← [Semana 14](../4-mes/semana-14-rn-basics.md) | [Volver al Roadmap](../../README.md) | [Semana 16 →](../4-mes/semana-16-rn-api-sync.md)*
