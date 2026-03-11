# Semana 16: Fetching y Sincronización Móvil Crítica

## 🎯 Objetivo
Dominar cómo manejar problemas típicos del entorno móvil (Pérdida de red, datos en caché, persistencia) e integración de APIs reales.

## 🛠️ Stack Tecnológico
- React Native
- Zustand + AsyncStorage
- Axios / React Query Mobile

## 📋 Requisitos del Proyecto
1. Crear una App "Gestor de Gastos y Presupuesto".
2. Conectar la app móvil al backend Node/Express que diseñaste en el Mes 1.
3. Usar Zustand para guardar la sesión del usuario obtenida del login general.
4. Persistir datos vitales localmente utilizando `AsyncStorage` (para que si cierran la app, la sesión continúe abierta).
5. (Opcional) Usar NetInfo para detectar si el celular no tiene Internet y mostrar un layout de "Offline".

## ✅ Entregables
- App conectada de verdad, persistente ante cierres de aplicación e interrupciones de red.
