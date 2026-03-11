# Semana 06: Gestión de Estado Avanzada

## 🎯 Objetivo
Entender cómo manejar datos a través de grandes jerarquías de componentes sin caer en el *Prop Drilling*.

## 🛠️ Stack Tecnológico
- ReactJS
- Context API
- Zustand (o Redux Toolkit)

## 📋 Requisitos del Proyecto
1. Construir un "Carrito de Compras" falso.
2. Implementar **Context API** para manejar "El Tema" de la aplicación (Modo Dark / Modo Light).
3. Utilizar **Zustand** (o Redux) para crear un *Store* global que guarde los productos añadidos al carrito.
4. Crear una vista principal con una lista de productos estáticos, y un contador global arriba en la barra de navegación que se actualice gracias al estado.

## ✅ Entregables
- Arquitectura limpia separando la lógica del estado (`/store` o `/context`) de las vistas UI.
- Navegación básica entre 'Tienda' y 'Carrito' usando React Router.
