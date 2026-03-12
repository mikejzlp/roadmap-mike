# 🧪 Semana 12 — Testing Full Stack (Jest + Cypress + Supertest)

> **Mes 3 · Full Stack Integration** | Stack: `Jest` · `React Testing Library` · `Cypress` · `Supertest` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Implementar una **estrategia de testing completa** para el stack Full Stack: pruebas unitarias de funciones puras, pruebas de integración para endpoints de la API, y pruebas end-to-end (E2E) del flujo de usuario en el navegador.

---

## 🛠️ Stack Tecnológico

| Herramienta | Tipo de Testing | Rol |
|---|---|---|
| Jest | Unit + Integration | Test runner y assertions |
| React Testing Library | Unit (Frontend) | Testear componentes React como lo haría un usuario |
| Supertest | Integration (Backend) | Hacer requests HTTP a la API Express en tests |
| Cypress | E2E | Automatizar el navegador para flujos completos |
| ts-jest | — | Transpilar TypeScript en tests de Jest |

---

## 📐 Pirámide de Testing

```
           /  E2E Tests  \          ← Cypress (pocos, caros, lentos)
          / Integration   \
         /   Tests (API)   \        ← Supertest + Jest (moderados)
        /   Unit Tests      \
       /  (Components/Utils) \      ← Jest + RTL (muchos, rápidos)
```

---

## 📋 Requisitos del Proyecto

### 1. Configurar Jest + ts-jest

```bash
npm install -D jest ts-jest @types/jest
npx ts-jest config:init
```

```json
// jest.config.json
{
  "preset": "ts-jest",
  "testEnvironment": "node",
  "roots": ["<rootDir>/src"],
  "testMatch": ["**/__tests__/**/*.ts", "**/*.test.ts"],
  "collectCoverageFrom": ["src/**/*.ts", "!src/**/*.d.ts"]
}
```

### 2. Prueba unitaria (función pura)

```typescript
// src/utils/__tests__/formatPrice.test.ts
import { formatPrice } from '../formatPrice';

describe('formatPrice', () => {
  it('formatea un número como precio en USD', () => {
    expect(formatPrice(1999.99)).toBe('$1,999.99');
  });

  it('maneja el caso de 0', () => {
    expect(formatPrice(0)).toBe('$0.00');
  });

  it('redondea correctamente', () => {
    expect(formatPrice(10.999)).toBe('$11.00');
  });
});
```

### 3. Prueba de integración con Supertest

```typescript
// src/__tests__/users.api.test.ts
import request from 'supertest';
import app from '../app';
import { prisma } from '../lib/prisma';

describe('Users API', () => {
  let authToken: string;

  beforeAll(async () => {
    // Crear usuario de prueba y obtener token
    const res = await request(app)
      .post('/api/auth/register')
      .send({ name: 'Test User', email: 'test@test.com', password: 'secret123' });
    
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@test.com', password: 'secret123' });

    authToken = loginRes.body.token;
  });

  afterAll(async () => {
    await prisma.user.deleteMany({ where: { email: 'test@test.com' } });
    await prisma.$disconnect();
  });

  it('GET /api/users → 401 sin token', async () => {
    const res = await request(app).get('/api/users');
    expect(res.status).toBe(401);
  });

  it('GET /api/users → 200 con token válido', async () => {
    const res = await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${authToken}`);
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
  });

  it('POST /api/users → Crea un usuario correctamente', async () => {
    const res = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ name: 'Nuevo Usuario', email: 'nuevo@test.com' });
    expect(res.status).toBe(201);
    expect(res.body).toHaveProperty('id');
  });
});
```

### 4. Prueba de componente React con RTL

```typescript
// src/components/__tests__/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '../Button';

describe('<Button />', () => {
  it('renderiza el label correctamente', () => {
    render(<Button label="Enviar" />);
    expect(screen.getByText('Enviar')).toBeInTheDocument();
  });

  it('llama onClick al hacer click', () => {
    const mockFn = jest.fn();
    render(<Button label="Click me" onClick={mockFn} />);
    fireEvent.click(screen.getByText('Click me'));
    expect(mockFn).toHaveBeenCalledTimes(1);
  });

  it('está deshabilitado cuando disabled=true', () => {
    render(<Button label="Disabled" disabled />);
    expect(screen.getByText('Disabled')).toBeDisabled();
  });
});
```

### 5. Test E2E con Cypress

```bash
npm install -D cypress
npx cypress open
```

```typescript
// cypress/e2e/auth.cy.ts
describe('Flujo de Autenticación', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('muestra error con credenciales incorrectas', () => {
    cy.get('[data-testid="email-input"]').type('wrong@email.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();
    cy.contains('Credenciales inválidas').should('be.visible');
  });

  it('redirige al dashboard tras login exitoso', () => {
    cy.get('[data-testid="email-input"]').type('user@example.com');
    cy.get('[data-testid="password-input"]').type('correctpassword');
    cy.get('[data-testid="login-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **AAA Pattern** | Arrange (preparar) → Act (ejecutar) → Assert (verificar) |
| **Mocking** | `jest.fn()` y `jest.mock()` para aislar dependencias |
| **Test Database** | Usar una BD de prueba separada, limpiar en `afterAll` |
| **`data-testid`** | Atributos en el HTML para que Cypress/RTL encuentren elementos sin depender de clases CSS |
| **Coverage** | `jest --coverage` genera un reporte de qué porcentaje del código está probado |

---

## ✅ Entregables

- [ ] Suite de tests unitarios para las funciones de utilidad del proyecto.
- [ ] Tests de integración para al menos 5 endpoints de la API (usando `supertest`).
- [ ] Tests de componentes React con RTL para `Button`, `TodoList` y `LoginForm`.
- [ ] Test E2E con Cypress para los flujos de registro, login y creación de recurso.
- [ ] Coverage > 70% configurado y reportado.

---

*← [Semana 11](../3-mes/semana-11-pern-ecommerce.md) | [Volver al Roadmap](../../README.md) | [Semana 13 →](../3-mes/semana-13-clean-architecture.md)*
