# Semana 17: React Native Avanzado (Animaciones y UI/UX Nativas)

## 🎯 Objetivo
Hacer que tu App deje de sentirse "Web" y pase a sentirse 100% "Nativa" mediante microinteracciones, animaciones de 60 FPS y librerías robustas.

## 🛠️ Stack Tecnológico
- React Native Reanimated v3
- React Native Gesture Handler
- Expo SDK / Android Plugins

## 📋 Requisitos del Proyecto
1. Crear una interfaz que implique arrastre (Drag & Drop) elemental, como una tarjeta de Tinder o un *BottomSheet*.
2. Implementar `Reanimated` para hacer transiciones fluidas de los colores o escalas de los componentes basándose en el Scroll del usuario (Interpolación).
3. Utilizar **React Native Gesture Handler** para detectar gestos complejos (Pinch to zoom, pan, long press) sin invocar el *Main Thread* de JavaScript, para no perder fotogramas.
4. Explorar cómo pedir Permisos Nativos (Cámara, Galería, Notificaciones) usando `expo-image-picker`.

## ✅ Entregables
- Proyecto Móvil exportado a `.apk` usando *EAS Build* o ejecutado en dispositivo físico que evidencie interacciones fluidas (60fps).
