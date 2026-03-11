# Semana 12: Testing (Frontend y Backend)

## 🎯 Objetivo
Garantizar la estabilidad del producto y adoptar mentalidad TDD (Test Driven Development). Aprender a escribir Código Limpio asegurando que no se rompe con los cambios.

## 🛠️ Stack Tecnológico
- Backend: Jest, Supertest
- Frontend: Vitest/Jest, React Testing Library (RTL), Playwright/Cypress

## 📋 Requisitos del Proyecto
1. Tomar el backend de E-Commerce (Semana 11) y escribir Pruebas Unitarias para las funciones principales aisladas con Jest.
2. Escribir Pruebas End-to-End (E2E) con Supertest llamando a las rutas API reales pasando un token Mockeado.
3. En Front, testear con React Testing Library que botones de un componente cambian de estado cuando se hace click.
4. Escribir un flujo de Cypress/Playwright que agarre tu MERN dashboard, abra el programa, inicie sesión llenando el formulario, entre al dashboard y haga un aserto.

## ✅ Entregables
- Repositorio con cobertura de pruebas al menos en un 60%.
- Un pipeline verde (`yarn test` o `npm run test` pasante).
