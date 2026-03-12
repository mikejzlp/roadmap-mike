# 🎓 Semana 26 — Portfolio & Perfil Senior Full Stack

> **Mes 6 · Cloud & Arquitectura Avanzada** | Optimización CV · GitHub Profile · Portfolio Web · LinkedIn · Entrevistas

---

## 🎯 Objetivo Final

**¡Lo lograste!** Completar los 6 meses de roadmap y presentar todo el trabajo de forma profesional. Esta semana es sobre **comunicar tus habilidades** de manera efectiva: un GitHub profile optimizado, un portfolio web impresionante y un CV adaptado a posiciones Senior Full Stack.

---

## 📋 Plan de la Semana

### Día 1-2: Auditoría y limpieza de GitHub

### Día 3-4: Portfolio Web personalizado

### Día 5: CV + LinkedIn optimizados

### Día 6-7: Mock interviews técnicas

---

## 🐙 GitHub Profile Profesional

### README.md del Profile (`github.com/tu-usuario`)

```markdown
# 👋 Hola, soy [Tu Nombre]

> Senior Full Stack Developer | React · Node.js · TypeScript · AWS · K8s

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/tu-perfil)
[![Portfolio](https://img.shields.io/badge/Portfolio-Ver-black?style=flat&logo=vercel)](https://tuportfolio.dev)

## 🚀 Stack Principal
**Frontend:** React · TypeScript · NextJS · Tailwind CSS · React Native  
**Backend:** Node.js · Express · NestJS · REST APIs · GraphQL  
**Bases de Datos:** PostgreSQL · MongoDB · Prisma ORM · Redis  
**DevOps:** Docker · Kubernetes · GitHub Actions · AWS (EC2, S3, Lambda, RDS)  

## 📊 Proyectos Destacados

| Proyecto | Descripción | Stack |
|---|---|---|
| [ERP Corporativo](link) | Sistema completo con RBAC, real-time y K8s | TS · React · Node · PostgreSQL · K8s |
| [E-Commerce PERN](link) | Tienda con Stripe, carrito y panel admin | React · Express · PostgreSQL · Stripe |
| [Chat en Tiempo Real](link) | App de chat con salas y Socket.io | Node.js · Socket.io · React · MongoDB |
| [Dashboard MERN](link) | Dashboard B2B con métricas en tiempo real | MERN · Recharts · JWT |
| [App Móvil](link) | App cross-platform con animaciones nativas | React Native · Expo · Reanimated |

## 📈 GitHub Stats
![Stats](https://github-readme-stats.vercel.app/api?username=tu-usuario&show_icons=true&theme=dark)
```

### Checklist de calidad para cada repositorio

```bash
# Cada uno de los 26 proyectos debe tener:
✅ README.md con:
   - Descripción clara del proyecto
   - Screenshots / GIF demo
   - Instrucciones de instalación (npm install, docker compose up)
   - Variables de entorno necesarias (.env.example)
   - Enlace al demo en producción (si aplica)
   - Stack tecnológico utilizado

✅ .gitignore correctamente configurado
✅ .env.example (sin valores reales)
✅ package.json con scripts utilitarios
✅ Tests corriendo con un comando (npm test)
✅ CI badge en el README (GitHub Actions)
```

---

## 🌐 Portfolio Web

### Estructura de la Landing Page Personal

```
Secciones del Portfolio:
1. 🦸 Hero — Nombre, título, CTA ("Contrátame" / "Ver Proyectos")
2. 💼 Proyectos — Los 5-7 mejores proyectos con filtro por tecnología
3. 🛠️ Skills — Stack tecnológico con niveles de proficiencia
4. 📊 Experiencia — Timeline de trabajo + roadmap completado
5. 📖 About — Historia personal y filosofía de trabajo
6. 📬 Contacto — Formulario + redes sociales
```

### Stack recomendado para el portfolio

```bash
# Option A: NextJS 14 con App Router (demuestra que sabes Next)
npx create-next-app@latest portfolio --typescript --tailwind --app

# Option B: Astro (ultra rápido, SSG perfecto para portfolios)
npm create astro@latest portfolio -- --template portfolio
```

```typescript
// Componente de proyecto con filtro por tecnología
const projects = [
  {
    id: 1,
    title: 'ERP Corporativo',
    description: 'Sistema de gestión empresarial completo con módulos de RRHH, inventario y finanzas.',
    stack: ['React', 'TypeScript', 'Node.js', 'PostgreSQL', 'Docker', 'K8s', 'AWS'],
    demo: 'https://erp.tu-dominio.com',
    github: 'https://github.com/tu-user/erp-corporativo',
    image: '/projects/erp.png',
    featured: true,
  },
  // ... más proyectos
];
```

---

## 📄 CV Optimizado para Posiciones Senior

### Estructura del CV de 1-2 páginas

```
[Tu Nombre] | [Ciudad, País remoto disponible]
[Email] | [LinkedIn] | [GitHub] | [Portfolio] | [Teléfono]

RESUMEN PROFESIONAL (3 líneas)
Senior Full Stack Developer con X años de experiencia desarrollando aplicaciones
web y móviles de alto rendimiento. Especialista en React, Node.js y arquitecturas
cloud (AWS/K8s). Apasionado por código limpio, testing y CI/CD.

STACK TÉCNICO
Frontend:  React 18, TypeScript, Next.js, React Native, Tailwind CSS, Redux Toolkit
Backend:   Node.js, Express, NestJS, REST API, GraphQL, Socket.io
Databases: PostgreSQL, MongoDB, Prisma ORM, Redis
DevOps:    Docker, Kubernetes, GitHub Actions, AWS (EC2, S3, Lambda, RDS)
Testing:   Jest, React Testing Library, Cypress, Supertest

PROYECTOS DESTACADOS
[Nombre del Proyecto] | react, node.js, postgresql, aws | [link demo] | [link github]
- Construí una API REST con autenticación JWT que maneja 10,000+ requests/día
- Implementé CI/CD reduciendo el tiempo de deploy de 2h a 8 minutos
- Diseñé el schema de DB con 15+ tablas relacionadas y queries optimizadas

EXPERIENCIA LABORAL
[Nombre Empresa] | [Título] | [Fechas]
- ... logros cuantificados (no responsabilidades)

EDUCACIÓN & CERTIFICACIONES
- AWS Certified Developer – Associate
- Roadmap Full Stack Developer – 6 meses, 26 proyectos
```

---

## 🎤 Preparación para Entrevistas Técnicas

### Preguntas frecuentes (con respuesta preparada)

```
JAVASCRIPT / TYPESCRIPT:
- ¿Diferencia entre == y ===?
- ¿Qué es el Event Loop?
- ¿Cómo funciona el closure?
- ¿Diferencia entre arrow function y función regular?
- ¿Qué son los generadores e iteradores?

REACT:
- ¿Cuándo usar useMemo vs useCallback?
- ¿Cómo prevenirías renders innecesarios?
- ¿Explícame el ciclo de vida de un componente con Hooks
- ¿Qué es el reconciliation process?

NODE.JS:
- ¿Cómo manejas errores en callbacks, Promises y async/await?
- ¿Qué es el cluster module y cuándo lo usarías?
- ¿Cómo evitar el callback hell?

BASES DE DATOS:
- ¿Diferencia entre JOIN INNER vs LEFT JOIN?
- ¿Qué es un índice y cómo mejora el rendimiento?
- ¿Cuándo NoSQL vs SQL?
- ¿Qué son las transacciones ACID?

SISTEMA DESIGN:
- Diseña una URL shortener (tipo bit.ly)
- Diseña un sistema de notificaciones en tiempo real
- Diseña una caché para una API de alto tráfico
```

---

## 🧠 Métricas del Roadmap Completado

| Métrica | Resultado |
|---|---|
| **Total de proyectos** | 26 proyectos terminados |
| **Tecnologías** | 20+ herramientas dominadas |
| **Líneas de código** | ~15,000+ líneas escritas |
| **Tests escritos** | 500+ assertions |
| **Semanas dedicadas** | 26 semanas (6 meses) |
| **Stack cubierto** | Frontend · Backend · Mobile · DevOps · Cloud |

---

## ✅ Entregables Finales del Roadmap

- [ ] **GitHub Profile** README impresionante con stats, proyectos y badges.
- [ ] **Portfolio Web** desplegado en Vercel/Netlify con dominio personalizado.
- [ ] **CV actualizado** (1-2 páginas, logros cuantificados, sin relleno).
- [ ] **LinkedIn profile** optimizado con todos los proyectos y skills.
- [ ] **5 mejores proyectos** con README impecable, demo en vivo y CI verde.
- [ ] **Mock interview** realizada con un colega o grabada.

---

## 🎉 ¡Felicitaciones!

Has completado el **Roadmap Full Stack Developer de 6 Meses**. Eres capaz de:

✅ Construir APIs REST robustas con Node.js, Express y TypeScript  
✅ Implementar autenticación segura con JWT y bcrypt  
✅ Trabajar con bases de datos SQL (PostgreSQL/Prisma) y NoSQL (MongoDB)  
✅ Desarrollar UIs modernas con React 18 y manejo de estado avanzado  
✅ Crear apps móviles cross-platform con React Native  
✅ Contenerizar aplicaciones con Docker y orquestarlas con Kubernetes  
✅ Automatizar pipelines CI/CD con GitHub Actions  
✅ Desplegar en AWS con EC2, S3, Lambda y RDS  

---

*← [Semana 25](../6-mes/semana-25-capstone-erp.md) | [🏠 Volver al Roadmap Principal](../../README.md)*
