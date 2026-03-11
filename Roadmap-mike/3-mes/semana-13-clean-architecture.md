# Semana 13: Clean Architecture y Principios SOLID en Node.js

## 🎯 Objetivo
Entender cómo estructurar proyectos robustos a nivel empresarial usando Clean Architecture, Patrón de Repositorios e Inyección de Dependencias.

## 🛠️ Stack Tecnológico
- Node.js + Express
- TypeScript Avanzado
- Patrones de Diseño de Software

## 📋 Requisitos del Proyecto
1. Refactorizar la API del mes 1 separando completamente la lógica en Capas (Layers).
2. **Capa de Dominio (Entities):** Modelos puros TypeScript.
3. **Capa de Casos de Uso (Application):** Lógica de negocio (ej. `CrearUsuarioUseCase.ts`).
4. **Capa de Interfaces de Red (Controllers):** Controladores de Express (`UsuarioController.ts`).
5. **Capa de Infraestructura (Repositories):** Aquí y solo aquí se toca Mongoose o Prisma.
6. Cumplir la 'D' de SOLID (Dependency Inversion) haciendo que los casos de uso dependan de una Interfaz y no de una implementación concreta de la base de datos.

## ✅ Entregables
- Proyecto desacoplado y altamente testeable y mantenible.
