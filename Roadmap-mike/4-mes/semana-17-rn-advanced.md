# ✨ Semana 17 — React Native Avanzado (Animaciones & Módulos Nativos)

> **Mes 4 · Mobile App Development** | Stack: `React Native Reanimated` · `Gesture Handler` · `Expo Camera` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Elevar la calidad de la app implementando **animaciones fluidas** con React Native Reanimated v3 y **gestos táctiles avanzados** con Gesture Handler. Integrar módulos nativos de Expo (Cámara, Notificaciones, Haptics) para una experiencia de usuario verdaderamente nativa.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| react-native-reanimated | Animaciones en el hilo nativo (sin JS thread) |
| react-native-gesture-handler | Gestos táctiles avanzados (pan, pinch, swipe) |
| expo-camera | Acceso a la cámara del dispositivo |
| expo-haptics | Feedback táctil (vibración) |
| expo-notifications | Notificaciones locales y push |

---

## 📐 Proyectos de la Semana

1. **Tarjeta animada** con `useSharedValue` y `withSpring`
2. **Lista con swipe-to-delete** usando `Gesture API`
3. **Escáner de QR** con `expo-camera`
4. **Loading Skeleton** con animación de shimmer

---

## 📋 Requisitos del Proyecto

### 1. Animación básica con Reanimated v3

```typescript
// src/components/AnimatedCard.tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
  interpolate,
  Extrapolation,
} from 'react-native-reanimated';
import { TouchableWithoutFeedback } from 'react-native';

export const AnimatedCard = ({ children }: { children: React.ReactNode }) => {
  const scale = useSharedValue(1);
  const shadowOpacity = useSharedValue(0.1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
    shadowOpacity: shadowOpacity.value,
  }));

  return (
    <TouchableWithoutFeedback
      onPressIn={() => {
        scale.value = withSpring(0.96);
        shadowOpacity.value = withTiming(0.3);
      }}
      onPressOut={() => {
        scale.value = withSpring(1);
        shadowOpacity.value = withTiming(0.1);
      }}
    >
      <Animated.View
        style={[
          { borderRadius: 16, backgroundColor: '#fff', padding: 16,
            shadowColor: '#000', shadowOffset: { width: 0, height: 4 }, shadowRadius: 12 },
          animatedStyle,
        ]}
      >
        {children}
      </Animated.View>
    </TouchableWithoutFeedback>
  );
};
```

### 2. Swipe-to-Delete con Gesture Handler

```typescript
// src/components/SwipeableItem.tsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue, useAnimatedStyle, withTiming, runOnJS
} from 'react-native-reanimated';
import { View, Text, StyleSheet, Dimensions } from 'react-native';

const { width: SCREEN_WIDTH } = Dimensions.get('window');
const SWIPE_THRESHOLD = -SCREEN_WIDTH * 0.4;

interface SwipeableItemProps {
  label: string;
  onDelete: () => void;
}

export const SwipeableItem: React.FC<SwipeableItemProps> = ({ label, onDelete }) => {
  const translateX = useSharedValue(0);

  const panGesture = Gesture.Pan()
    .onChange((event) => {
      translateX.value = Math.min(0, translateX.value + event.changeX);
    })
    .onFinalize(() => {
      if (translateX.value < SWIPE_THRESHOLD) {
        translateX.value = withTiming(-SCREEN_WIDTH, {}, () => {
          runOnJS(onDelete)(); // Llamar al callback en el JS thread
        });
      } else {
        translateX.value = withTiming(0);
      }
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));

  return (
    <View style={styles.container}>
      {/* Fondo rojo de "eliminar" */}
      <View style={styles.deleteBackground}>
        <Text style={styles.deleteText}>🗑 Eliminar</Text>
      </View>
      <GestureDetector gesture={panGesture}>
        <Animated.View style={[styles.item, animatedStyle]}>
          <Text style={styles.label}>{label}</Text>
        </Animated.View>
      </GestureDetector>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { marginHorizontal: 16, marginVertical: 4 },
  deleteBackground: {
    position: 'absolute', right: 0, top: 0, bottom: 0, width: '40%',
    backgroundColor: '#ef4444', justifyContent: 'center', alignItems: 'center',
    borderRadius: 12,
  },
  deleteText: { color: '#fff', fontWeight: '700' },
  item: {
    backgroundColor: '#fff', padding: 16, borderRadius: 12,
    shadowColor: '#000', shadowOpacity: 0.05, shadowRadius: 8, elevation: 2,
  },
  label: { fontSize: 16 },
});
```

### 3. Animación Shimmer (Loading Skeleton)

```typescript
// src/components/Skeleton.tsx
import Animated, {
  useSharedValue, useAnimatedStyle, withRepeat, withTiming, interpolateColor
} from 'react-native-reanimated';
import { useEffect } from 'react';
import { View, StyleSheet } from 'react-native';

export const Skeleton = ({ width = '100%', height = 20, borderRadius = 8 }: {
  width?: number | string;
  height?: number;
  borderRadius?: number;
}) => {
  const progress = useSharedValue(0);

  useEffect(() => {
    progress.value = withRepeat(withTiming(1, { duration: 1000 }), -1, true);
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    backgroundColor: interpolateColor(
      progress.value,
      [0, 1],
      ['#e5e7eb', '#f3f4f6']
    ),
  }));

  return (
    <Animated.View
      style={[{ width, height, borderRadius } as any, animatedStyle]}
    />
  );
};
```

### 4. Feedback Háptico

```typescript
import * as Haptics from 'expo-haptics';

// Al completar una acción importante
await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);

// Al presionar un botón
await Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);

// Al ocurrir un error
await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **`useSharedValue`** | Valor que vive en el hilo nativo de UI, no en el hilo de JavaScript |
| **`useAnimatedStyle`** | Convierte `SharedValue`s en estilos nativos. NUNCA llames funciones JS dentro |
| **`runOnJS`** | Puente para llamar a funciones de JS desde el worklet nativo |
| **`withSpring`** | Animación con física de resorte (más natural que `withTiming`) |
| **Worklets** | Funciones que se compilan para ejecutarse en el hilo nativo de UI |

---

## ✅ Entregables

- [ ] Tarjeta con animación de press (`scale + shadow`) usando `useSharedValue`.
- [ ] Lista con swipe-to-delete usando `Gesture.Pan()`.
- [ ] Componente `Skeleton` con animación de shimmer infinita.
- [ ] Integración de Haptics en botones de acción principal.

---

*← [Semana 16](../4-mes/semana-16-rn-api-sync.md) | [Volver al Roadmap](../../README.md) | [Semana 18 →](../5-mes/semana-18-linux-bash.md)*
